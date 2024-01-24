事务：一组操作的组合，要么完整的执行，要么不执行；
事务系统可以分为三个部分：事务管理器、锁管理器、日志管理器；

事务管理器：事务处理系统的中枢；
锁管理器：并发控制；
日志管理器：记录事务执行的状态以及数据的变化过程；

显式的事务、隐式的事务；

pg的保存点（savepoint），是以子事务来实现的。  

事务块的状态

```C
typedef enum TBlockState
{
    /* not-in-transaction-block states */
    TBLOCK_DEFAULT,                /* idle */
    TBLOCK_STARTED,                /* running single-query transaction */

    /* transaction block states */
    TBLOCK_BEGIN,                /* starting transaction block */
    TBLOCK_INPROGRESS,            /* live transaction */
    TBLOCK_IMPLICIT_INPROGRESS, /* live transaction after implicit BEGIN */
    TBLOCK_PARALLEL_INPROGRESS, /* live transaction inside parallel worker */
    TBLOCK_END,                    /* COMMIT received */
    TBLOCK_ABORT,                /* failed xact, awaiting ROLLBACK */
    TBLOCK_ABORT_END,            /* failed xact, ROLLBACK received */
    TBLOCK_ABORT_PENDING,        /* live xact, ROLLBACK received */
    TBLOCK_PREPARE,                /* live xact, PREPARE received */

    /* subtransaction states */
    TBLOCK_SUBBEGIN,            /* starting a subtransaction */
    TBLOCK_SUBINPROGRESS,        /* live subtransaction */
    TBLOCK_SUBRELEASE,            /* RELEASE received */
    TBLOCK_SUBCOMMIT,            /* COMMIT received while TBLOCK_SUBINPROGRESS */
    TBLOCK_SUBABORT,            /* failed subxact, awaiting ROLLBACK */
    TBLOCK_SUBABORT_END,        /* failed subxact, ROLLBACK received */
    TBLOCK_SUBABORT_PENDING,    /* live subxact, ROLLBACK received */
    TBLOCK_SUBRESTART,            /* live subxact, ROLLBACK TO received */
    TBLOCK_SUBABORT_RESTART        /* failed subxact, ROLLBACK TO received */
} TBlockState;
```

事务相关函数

```C
extern void StartTransactionCommand(void);
extern void CommitTransactionCommand(void);
extern void AbortCurrentTransaction(void);
```

什么是子事务？ 事务的嵌套深度？
子事务和保存点的关系？ 子事务除了用来实现保存点，还有其他用途吗？

事务ID：真实的ID和虚拟的事务ID；
写操作的事务ID，用的实的事务ID，只读的操作，使用虚拟事务ID，节省事务ID；
virtualtransactionid的组成：会话进程号+localtransactionid, 避免vxid重复的时候，还要避免和xid重复；

事务开始，不会分配实的xid，而是分配vxid，当发生修改操作后，才分配实的xid;

脏读：一个事务，读到了另一个事务尚未提交的素具；
不可重复读：一个事务，先读取到的数据，和后读取的数据，出现不一致的问题；
幻读： 一个事务重复执行一个查询，返回的结果，与上一次返回的结果不一致；
幻读和不可重复读的区别：幻读一般指新增了数据；

postgresql的MVCC;

postgresql的日志管理；
clog存储在哪里的？data目录的那个文件夹下面？？
    在pg_xact目录下，SimpleLruInit的时候，指定了各种使用lru存储的资源的路径；

访问clog中的指定的页面

```C
#define TransactionIdToPage(xid)    ((xid) / (TransactionId) CLOG_XACTS_PER_PAGE)
#define TransactionIdToPgIndex(xid) ((xid) % (TransactionId) CLOG_XACTS_PER_PAGE)
#define TransactionIdToByte(xid)    (TransactionIdToPgIndex(xid) / CLOG_XACTS_PER_BYTE)
#define TransactionIdToBIndex(xid)    ((xid) % (TransactionId) CLOG_XACTS_PER_BYTE)
```