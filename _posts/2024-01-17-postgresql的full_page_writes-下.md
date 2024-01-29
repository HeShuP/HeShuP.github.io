## PostgreSQL怎么判断一个page是checkpoint后的首次更改？

### 相关数据结构

PostgreSQL中，一个page的结构如下：

```c
+----------------+---------------------------------+
| PageHeaderData | linp1 linp2 linp3 ...           |
+-----------+----+---------------------------------+
| ... linpN |                                      |
+-----------+--------------------------------------+
|           ^ pd_lower                             |
|                                                  |
|             v pd_upper                           |
+-------------+------------------------------------+
|             | tupleN ...                         |
+-------------+------------------+-----------------+
|       ... tuple3 tuple2 tuple1 | "special space" |
+--------------------------------+-----------------+
                                    ^ pd_special
```

其中，PageHeaderData存储page的头部信息，其结构如下：

```C
typedef struct PageHeaderData
{
    /* XXX LSN is member of *any* block, not only page-organized ones */
    PageXLogRecPtr pd_lsn;        /* LSN: next byte after last byte of xlog
                                 * record for last change to this page */
    uint16        pd_checksum;    /* checksum */
    uint16        pd_flags;        /* flag bits, see below */
    LocationIndex pd_lower;        /* offset to start of free space */
    LocationIndex pd_upper;        /* offset to end of free space */
    LocationIndex pd_special;    /* offset to start of special space */
    uint16        pd_pagesize_version;
    TransactionId pd_prune_xid; /* oldest prunable XID, or zero if none */
    ItemIdData    pd_linp[FLEXIBLE_ARRAY_MEMBER]; /* line pointer array */
} PageHeaderData;
```

能轻易的发现，在PageHeaderData中，pd_lsn记录了修改该page的xlog记录最后一个字节之后的下一个字节；pd_checksum则记录了该page的校验和，当进行坏块修复时，需要用到该值。

### xlog写日志的流程概述

1. buf添加锁
   
   ```C
   LockBuffer(buf, BUFFER_LOCK_EXCLUSIVE);
   ```

2. 将bufer标记为dirty
   
   ```C
   MarkBufferDirty(buf);
   ```

3. 注册buf到xlog
   
   ```C
   XLogBeginInsert();
   XLogRegisterBuffer(0, buf, REGBUF_WILL_INIT);
   xlrec.node = rel->rd_node;
   XLogRegisterData((char *) &xlrec, sizeof(xl_seq_rec));
   XLogRegisterData((char *) tuple->t_data, tuple->t_len);
   ```

4. 插入xlog记录   
- XLogInsert
  
  - 判断是否执行全页写?
    GetFullPageWriteInfo(&RedoRecPtr, &doPageWrites);
  
  - 组装xlog日志
    XLogRecordAssemble(rmid, info, RedoRecPtr, doPageWrites, &fpw_lsn); 
    
    - XLogRecPtr page_lsn = PageGetLSN(regbuf->page); 
      **此处PageGetLSN获取的LSN,为上一次更新该page，记录的LSN**
    
    - needs_backup = (page_lsn <= RedoRecPtr);  
      **RedoRecPtr为上次checkpoint后，记录的redo点；**

- XLogInsertRecord(rdt, fpw_lsn, curinsert_flags);    

- 设置page的LSN
  PageSetLSN(page, recptr);    //xlog日志插入之后，再更新该page的LSN
5. 释放buf上添加的锁
   
   ```C
     UnlockReleaseBuffer(buf);
   ```

## 坏块修复

当部署主备集群时，如果主PostgreSQL数据库出现坏块，一般是通过该坏块的LSN，去备库查找对应的块，然后发送到主库，覆盖掉主库的坏块。

## 性能与安全的取舍

启用full_page_writes后，将导致wal文件膨胀、数据库性能的下降，是否启用full_page_writes,应**从实际出发**，如果存储层能保证不会出现页断裂的问题，则完全可以不用启用full_page_writes。或者环境稳定，出现页断裂的概率极低，亦或者有坏块自动修复的手段，也可以考虑不用启动full_page_writes功能。
