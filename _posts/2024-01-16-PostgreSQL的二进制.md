### 目的

编译了PostgreSQL源代码之后，在bin目录下，存在一堆二进制的可执行文件，本文的主要目的，是介绍各个二进制文件的作用，并对重要的二进制的典型使用方法，进行举例。

### 二进制

#### clusterdb

- 功能
  clusterdb 是SQL CLUSTER 命令的一个封装。 PostgreSQL 是堆表存储的， clusterdb通过索引对数据库中基于堆表的物理文件重新排序 ，它在一定场景下可以节省磁盘访问，加快查询速度。

- 示例

```shell
  [coder@localhost build_postgres]$ ./bin/clusterdb  -p 5433 -U system  -d postgres -e
  SELECT pg_catalog.set_config('search_path', '', false);
  CLUSTER;
```

#### createdb

- 功能
  - 用于创建一个PostgreSQL的数据库，与SQL语句CREATE DATABASE的作用是一致的;
- 示例

```shell
[coder@localhost build_postgres]$ ./bin/createdb -p 5433 -U system bm
[coder@localhost build_postgres]$ ./bin/psql -p 5433 -U system -d postgres 
psql (12.17)
Type "help" for help.
postgres=# select * from pg_database;
  oid  |  datname  | datdba | encoding | datcollate  |  datctype   | datistemplate | datallowconn | datconnlimit | datlastsysoid | datfrozenxid | datminmxid | dattablespace |            datacl             
-------+-----------+--------+----------+-------------+-------------+---------------+--------------+--------------+---------------+--------------+------------+---------------+-------------------------------
 13593 | postgres  |     10 |        6 | zh_CN.UTF-8 | zh_CN.UTF-8 | f             | t            |           -1 |         13592 |          479 |          1 |          1663 | 
 24579 | bm        |     10 |        6 | zh_CN.UTF-8 | zh_CN.UTF-8 | f             | t            |           -1 |         13592 |          479 |          1 |          1663 | 
     1 | template1 |     10 |        6 | zh_CN.UTF-8 | zh_CN.UTF-8 | t             | t            |           -1 |         13592 |          479 |          1 |          1663 | {=c/system,system=CTc/system}
 13592 | template0 |     10 |        6 | zh_CN.UTF-8 | zh_CN.UTF-8 | t             | f            |           -1 |         13592 |          479 |          1 |          1663 | {=c/system,system=CTc/system}
(4 rows)

postgres=# 
```

#### createuser

- 功能
  - 创建用户；
- 示例

#### dropdb

- 功能
  - 删除数据库；
- 示例

#### dropuser

- 功能
  - 删除用户；
- 示例

#### ecpg

- 功能
- 示例

#### initdb

- 功能
- 示例

#### oid2name

- 功能
- 示例

#### pg_archivecleanup

- 功能
- 示例

#### pg_basebackup

- 功能
  
  - 获得一个PostgreSQL集簇的一个基础备份;

- 示例
  
  ```shell
  [coder@localhost build_postgres]$ ./bin/pg_basebackup -h localhost -p 5433 -U system -D /home/coder/workspace/date_temp/
  [coder@localhost build_postgres]$ ll /home/coder/workspace/date_temp/
  总用量 60
  -rw-------. 1 coder coder   224 1月  30 18:44 backup_label
  drwx------. 6 coder coder    54 1月  30 18:44 base
  drwx------. 2 coder coder  4096 1月  30 18:44 global
  drwx------. 2 coder coder  4096 1月  30 18:44 log
  drwx------. 2 coder coder     6 1月  30 18:44 pg_commit_ts
  drwx------. 2 coder coder     6 1月  30 18:44 pg_dynshmem
  -rw-------. 1 coder coder  4760 1月  30 18:44 pg_hba.conf
  -rw-------. 1 coder coder  1636 1月  30 18:44 pg_ident.conf
  drwx------. 4 coder coder    68 1月  30 18:44 pg_logical
  drwx------. 4 coder coder    36 1月  30 18:44 pg_multixact
  drwx------. 2 coder coder     6 1月  30 18:44 pg_notify
  drwx------. 2 coder coder     6 1月  30 18:44 pg_replslot
  drwx------. 2 coder coder     6 1月  30 18:44 pg_serial
  drwx------. 2 coder coder     6 1月  30 18:44 pg_snapshots
  drwx------. 2 coder coder     6 1月  30 18:44 pg_stat
  drwx------. 2 coder coder     6 1月  30 18:44 pg_stat_tmp
  drwx------. 2 coder coder     6 1月  30 18:44 pg_subtrans
  drwx------. 2 coder coder     6 1月  30 18:44 pg_tblspc
  drwx------. 2 coder coder     6 1月  30 18:44 pg_twophase
  -rw-------. 1 coder coder     3 1月  30 18:44 PG_VERSION
  drwx------. 3 coder coder    60 1月  30 18:44 pg_wal
  drwx------. 2 coder coder    18 1月  30 18:44 pg_xact
  -rw-------. 1 coder coder   113 1月  30 18:44 postgresql.auto.conf
  -rw-------. 1 coder coder 26746 1月  30 18:44 postgresql.conf
  ```

#### pgbench

- 功能
  - PostgreSQL的基准测试工具;
- 示例

```shell
/* 初始化bm库 */
[coder@localhost build_postgres]$ ./bin/pgbench -i bm -h localhost -p 5433 -U system
dropping old tables...
NOTICE:  table "pgbench_accounts" does not exist, skipping
NOTICE:  table "pgbench_branches" does not exist, skipping
NOTICE:  table "pgbench_history" does not exist, skipping
NOTICE:  table "pgbench_tellers" does not exist, skipping
creating tables...
generating data...
100000 of 100000 tuples (100%) done (elapsed 0.11 s, remaining 0.00 s)
vacuuming...
creating primary keys...
done.
[coder@localhost build_postgres]$

/* 初始化测试数据 */
[coder@localhost build_postgres]$ ./bin/pgbench -i bm -h localhost -p 5433 -U system -s 10
dropping old tables...
creating tables...
generating data...
100000 of 1000000 tuples (10%) done (elapsed 0.11 s, remaining 0.99 s)
200000 of 1000000 tuples (20%) done (elapsed 0.23 s, remaining 0.91 s)
300000 of 1000000 tuples (30%) done (elapsed 0.33 s, remaining 0.77 s)
400000 of 1000000 tuples (40%) done (elapsed 0.45 s, remaining 0.67 s)
500000 of 1000000 tuples (50%) done (elapsed 0.55 s, remaining 0.55 s)
600000 of 1000000 tuples (60%) done (elapsed 0.66 s, remaining 0.44 s)
700000 of 1000000 tuples (70%) done (elapsed 0.98 s, remaining 0.42 s)
800000 of 1000000 tuples (80%) done (elapsed 1.26 s, remaining 0.31 s)
900000 of 1000000 tuples (90%) done (elapsed 1.82 s, remaining 0.20 s)
1000000 of 1000000 tuples (100%) done (elapsed 2.25 s, remaining 0.00 s)
vacuuming...
creating primary keys...
done.
[coder@localhost build_postgres]$ du -sh data/

/* 开始测试，模拟5个客户端连接，每个客户端连接，运行10000个事务数 */
[coder@localhost build_postgres]$ ./bin/pgbench bm -p 5433 -U system -t 10000 -c 5
starting vacuum...end.
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 10
query mode: simple
number of clients: 5
number of threads: 1
number of transactions per client: 10000
number of transactions actually processed: 50000/50000
latency average = 7.021 ms
tps = 712.165617 (including connections establishing)
tps = 712.221039 (excluding connections establishing)
```

#### pg_checksums

- 功能
  - 在PostgreSQL数据库集群中启用、禁用或验证数据校验和。
- 示例

```shell
[coder@localhost build_postgres]$ ./bin/pg_checksums -D data/ -d
pg_checksums: syncing data directory
pg_checksums: updating control file
Checksums disabled in cluster
```

#### pg_config

- 功能
  
  - 获取已安装的PostgreSQL的信息;

- 示例

```shell
[coder@localhost build_postgres]$ ./bin/pg_config 
BINDIR = /home/coder/workspace/build_postgres/bin
DOCDIR = /home/coder/workspace/build_postgres/share/doc
HTMLDIR = /home/coder/workspace/build_postgres/share/doc
INCLUDEDIR = /home/coder/workspace/build_postgres/include
PKGINCLUDEDIR = /home/coder/workspace/build_postgres/include
INCLUDEDIR-SERVER = /home/coder/workspace/build_postgres/include/server
LIBDIR = /home/coder/workspace/build_postgres/lib
PKGLIBDIR = /home/coder/workspace/build_postgres/lib
LOCALEDIR = /home/coder/workspace/build_postgres/share/locale
MANDIR = /home/coder/workspace/build_postgres/share/man
SHAREDIR = /home/coder/workspace/build_postgres/share
SYSCONFDIR = /home/coder/workspace/build_postgres/etc
PGXS = /home/coder/workspace/build_postgres/lib/pgxs/src/makefiles/pgxs.mk
CONFIGURE = '--with-pgport=5432' '--prefix=/home/coder/workspace/build_postgres' '--without-readline' '--enable-debug' 'CFLAGS=-O0'
CC = gcc -std=gnu99
CPPFLAGS = -D_GNU_SOURCE
CFLAGS = -Wall -Wmissing-prototypes -Wpointer-arith -Wdeclaration-after-statement -Werror=vla -Wendif-labels -Wmissing-format-attribute -Wformat-security -fno-strict-aliasing -fwrapv -fexcess-precision=standard -g -O0
CFLAGS_SL = -fPIC
LDFLAGS = -Wl,--as-needed -Wl,-rpath,'/home/coder/workspace/build_postgres/lib',--enable-new-dtags
LDFLAGS_EX = 
LDFLAGS_SL = 
LIBS = -lpgcommon -lpgport -lpthread -lz -lrt -lcrypt -ldl -lm 
VERSION = PostgreSQL 12.17
```

#### pg_controldata

- 功能
  - 显示PostgreSQL数据库集群的控制信息。
- 示例

```shell
  [coder@localhost build_postgres]$ ./bin/pg_controldata -D data/
  pg_control version number:            1201
  Catalog version number:               201909212
  Database system identifier:           7324956055052174572
  Database cluster state:               shut down
  pg_control last modified:             2024年01月30日 星期二 19时54分50秒
  Latest checkpoint location:           0/43DF8388
  Latest checkpoint's REDO location:    0/43DF8388
  Latest checkpoint's REDO WAL file:    000000010000000000000043
  Latest checkpoint's TimeLineID:       1
  Latest checkpoint's PrevTimeLineID:   1
  Latest checkpoint's full_page_writes: off
  Latest checkpoint's NextXID:          0:252495
  Latest checkpoint's NextOID:          24645
  Latest checkpoint's NextMultiXactId:  1
  Latest checkpoint's NextMultiOffset:  0
  Latest checkpoint's oldestXID:        479
  Latest checkpoint's oldestXID's DB:   1
  Latest checkpoint's oldestActiveXID:  0
  Latest checkpoint's oldestMultiXid:   1
  Latest checkpoint's oldestMulti's DB: 1
  Latest checkpoint's oldestCommitTsXid:0
  Latest checkpoint's newestCommitTsXid:0
  Time of latest checkpoint:            2024年01月30日 星期二 19时54分50秒
  Fake LSN counter for unlogged rels:   0/3E8
  Minimum recovery ending location:     0/0
  Min recovery ending loc's timeline:   0
  Backup start location:                0/0
  Backup end location:                  0/0
  End-of-backup record required:        no
  wal_level setting:                    replica
  wal_log_hints setting:                off
  max_connections setting:              100
  max_worker_processes setting:         8
  max_wal_senders setting:              10
  max_prepared_xacts setting:           0
  max_locks_per_xact setting:           64
  track_commit_timestamp setting:       off
  Maximum data alignment:               8
  Database block size:                  8192
  Blocks per segment of large relation: 131072
  WAL block size:                       8192
  Bytes per WAL segment:                16777216
  Maximum length of identifiers:        64
  Maximum columns in an index:          32
  Maximum size of a TOAST chunk:        1996
  Size of a large-object chunk:         2048
  Date/time type storage:               64-bit integers
  Float4 argument passing:              by value
  Float8 argument passing:              by value
  Data page checksum version:           0
  Mock authentication nonce:            bacbe3f518797090c62e0e5420a528b4706f0aeb4ecfe4aa8c46d5e116e08643
```

#### pg_ctl

- 功能
  - 用于初始化、启动、停止或控制PostgreSQL服务器的工具;
- 示例

```shell
[coder@localhost build_postgres]$ ./bin/pg_ctl -D data/ start 
waiting for server to start....2024-01-30 19:58:54.730 CST [9864] LOG:  starting PostgreSQL 12.17 on x86_64-pc-linux-gnu, compiled by gcc (GCC) 4.8.5 20150623 (Red Hat 4.8.5-44), 64-bit
2024-01-30 19:58:54.734 CST [9864] LOG:  listening on IPv6 address "::1", port 5433
2024-01-30 19:58:54.734 CST [9864] LOG:  listening on IPv4 address "127.0.0.1", port 5433
2024-01-30 19:58:54.738 CST [9864] LOG:  listening on Unix socket "/tmp/.s.PGSQL.5433"
2024-01-30 19:58:54.764 CST [9865] LOG:  database system was shut down at 2024-01-30 19:54:50 CST
2024-01-30 19:58:54.769 CST [9864] LOG:  database system is ready to accept connections
done
server started
[coder@localhost build_postgres]$
```

#### pg_dump

- 功能
  - pg_dump将数据库转储为文本文件或其他格式;
  - 可用于逻辑备份；
- 示例

```shell
[coder@localhost build_postgres]$ ./bin/pg_dump -h localhost -p 5433 -U system  postgres > postgres.sql
[coder@localhost build_postgres]$ ll
总用量 648
drwxrwxr-x.  2 coder coder   4096 1月  17 20:34 bin
drwx------. 20 coder coder   4096 1月  30 19:58 data
drwxrwxr-x.  6 coder coder   4096 1月  17 20:34 include
drwxrwxr-x.  4 coder coder   4096 1月  17 20:34 lib
-rw-rw-r--.  1 coder coder 639563 1月  30 20:11 postgres.sql
drwxrwxr-x.  7 coder coder   4096 1月  17 20:34 share
[coder@localhost build_postgres]$ 
```

#### pg_dumpall

- 功能
  - pg_dumpall将PostgreSQL数据库集群提取到SQL脚本文件中；
  - 相比于pg_dump，提取的范围更大；
- 示例

```shell
[coder@localhost build_postgres]$ ./bin/pg_dumpall -h localhost -p 5433 -U system > backup.sql
[coder@localhost build_postgres]$ ll
总用量 98032
-rw-rw-r--.  1 coder coder 99718816 1月  30 20:17 backup.sql
drwxrwxr-x.  2 coder coder     4096 1月  17 20:34 bin
drwx------. 20 coder coder     4096 1月  30 19:58 data
drwxrwxr-x.  6 coder coder     4096 1月  17 20:34 include
drwxrwxr-x.  4 coder coder     4096 1月  17 20:34 lib
-rw-rw-r--.  1 coder coder   639563 1月  30 20:11 postgres.sql
drwxrwxr-x.  7 coder coder     4096 1月  17 20:34 share
[coder@localhost build_postgres]$
```

#### pg_isready

- 功能
  - pg_isready向PostgreSQL数据库发出连接检查。
  - 要获得服务器状态，不一定需要提供正确的用户名、口令或数据库名。 不过，如果提供了不正确的值，服务器将会记录一次失败的连接尝试。
- 示例

```shell
[coder@localhost build_postgres]$ ./bin/pg_isready -d postgres -h localhost -p 5433 -U system
localhost:5433 - accepting connections
```

#### pg_receivewal

- 功能
  - 以流的方式从一个PostgreSQL服务器得到预写式日志
- 示例

```shell
[coder@localhost build_postgres]$ mkdir temp
[coder@localhost build_postgres]$ ./bin/pg_receivewal -h localhost -p 5433 -U system -D temp/
^Cpg_receivewal: not renaming "000000010000000000000043.partial", segment is not complete
[coder@localhost build_postgres]$ ll temp/
总用量 16384
-rw-------. 1 coder coder 16777216 1月  30 20:36 000000010000000000000043.partial
[coder@localhost build_postgres]$ 
```

#### pg_recvlogical

- 功能
  - pg_recvlogical — 控制 PostgreSQL 逻辑解码流；
- 示例

```shell
/* 创建复制槽 */
[coder@localhost build_postgres]$ ./bin/pg_recvlogical -h localhost -p 5433 -U system -d postgres --create-slot -S node_slot
2024-01-30 20:46:24.244 CST [10759] LOG:  logical decoding found consistent point at 0/43EFB738
2024-01-30 20:46:24.244 CST [10759] DETAIL:  There are no running transactions.
2024-01-30 20:46:24.244 CST [10759] STATEMENT:  CREATE_REPLICATION_SLOT "node_slot" LOGICAL "test_decoding" NOEXPORT_SNAPSHOT

/* 开始接收逻辑解码数据 */
[coder@localhost build_postgres]$ ./bin/pg_recvlogical -h localhost -p 5433 -U system -d postgres -S node_slot --start -f logical.txt
2024-01-30 20:46:33.402 CST [10761] LOG:  starting logical decoding for slot "node_slot"
2024-01-30 20:46:33.402 CST [10761] DETAIL:  Streaming transactions committing after 0/43EFB770, reading WAL from 0/43EFB738.
2024-01-30 20:46:33.402 CST [10761] STATEMENT:  START_REPLICATION SLOT "node_slot" LOGICAL 0/0
2024-01-30 20:46:33.402 CST [10761] LOG:  logical decoding found consistent point at 0/43EFB738
2024-01-30 20:46:33.402 CST [10761] DETAIL:  There are no running transactions.
2024-01-30 20:46:33.402 CST [10761] STATEMENT:  START_REPLICATION SLOT "node_slot" LOGICAL 0/0
2024-01-30 20:46:53.451 CST [10764] ERROR:  syntax error at or near ")" at character 29
2024-01-30 20:46:53.451 CST [10764] STATEMENT:  insert into tb_1 values(2,2,);
^Cpg_recvlogical: error: unexpected termination of replication stream: 
[coder@localhost build_postgres]$
```

#### pg_resetwal

- 功能
  - 重置PostgreSQL预写日志；
- 示例

#### pg_restore

- 功能
  - 从pg_dump创建的存档中恢复PostgreSQL数据库；
- 示例

#### pg_rewind

- 功能
  - pg_rewind将一个PostgreSQL集群与该集群的另一个副本重新同步；
- 示例

```shell
/* 先用pg_basebackup 创建一个基础备份，然后将备份data提升为主，使源和目标的数据出现分叉 */

/* 然后使用pg_rewind，同步两个出现分叉的data目录 */
[coder@localhost build_postgres]$ ./bin/pg_rewind -D /home/coder/workspace/date_temp --source-server='host=localhost port=5433 user=system' -P
pg_rewind: connected to server
pg_rewind: servers diverged at WAL location 0/4901CD48 on timeline 1
pg_rewind: rewinding from last common checkpoint at 0/4901CCD0 on timeline 1
pg_rewind: reading source file list
pg_rewind: reading target file list
pg_rewind: reading WAL in target
pg_rewind: need to copy 855 MB (total source directory size is 1057 MB)
...
...
```

#### pg_standby

- 功能
- 示例

#### pg_test_fsync

- 功能
  - 为PostgreSQL判断最快的wal_sync_method；
  - 根据测试结果，修改postgresql.conf中的wal_sync_method配置，选择最快方式，将wal日志同步到磁盘；
- 示例

```shellag-0-1hlc17elnag-1-1hlc17eln
[coder@localhost build_postgres]$ ./bin/pg_test_fsync 
5 seconds per test
O_DIRECT supported on this platform for open_datasync and open_sync.

Compare file sync methods using one 8kB write:
(in wal_sync_method preference order, except fdatasync is Linux's default)
        open_datasync                      7541.810 ops/sec     133 usecs/op
        fdatasync                          7144.803 ops/sec     140 usecs/op
        fsync                              6231.270 ops/sec     160 usecs/op
        fsync_writethrough                              n/a
        open_sync                          6657.784 ops/sec     150 usecs/op

Compare file sync methods using two 8kB writes:
(in wal_sync_method preference order, except fdatasync is Linux's default)
        open_datasync                      4004.038 ops/sec     250 usecs/op
        fdatasync                          7363.566 ops/sec     136 usecs/op
        fsync                              5159.695 ops/sec     194 usecs/op
        fsync_writethrough                              n/a
        open_sync                          3190.544 ops/sec     313 usecs/op

Compare open_sync with different write sizes:
(This is designed to compare the cost of writing 16kB in different write
open_sync sizes.)
         1 * 16kB open_sync write          6909.855 ops/sec     145 usecs/op
         2 *  8kB open_sync writes         3510.708 ops/sec     285 usecs/op
         4 *  4kB open_sync writes         1838.501 ops/sec     544 usecs/op
         8 *  2kB open_sync writes          931.700 ops/sec    1073 usecs/op
        16 *  1kB open_sync writes          462.425 ops/sec    2163 usecs/op

Test if fsync on non-write file descriptor is honored:
(If the times are similar, fsync() can sync data written on a different
descriptor.)
        write, fsync, close                5145.644 ops/sec     194 usecs/op
        write, close, fsync                6185.307 ops/sec     162 usecs/op

Non-sync'ed 8kB writes:
        write                            282802.240 ops/sec       4 usecs/op
[coder@localhost build_postgres]$ 
```

#### pg_test_timing

- 功能
  - 度量计时开销;
  - 一种度量在你的系统上计时开销以及确认系统时间绝不会回退的工具;
- 示例

```shelll
ag-0-1hlc17elnag-1-1hlc17eln[coder@localhost build_postgres]$ ./bin/pg_test_timing 
Testing timing overhead for 3 seconds.
Per loop time including overhead: 42.59 ns
Histogram of timing durations:
  < us   % of total      count
     1     95.88529   67537504
     2      4.11194    2896273
     4      0.00132        928
     8      0.00018        128
    16      0.00011         80
    32      0.00023        159
    64      0.00025        179
   128      0.00041        291
   256      0.00020        144
   512      0.00004         29
  1024      0.00001          6
  2048      0.00000          3
  4096      0.00000          1
  8192      0.00000          1
 16384      0.00000          1
```

#### pg_upgrade

- 功能
  - 升级PostgreSQL集群到不同的主版本;
- 示例

```shell
pg_upgrade.exe
        --old-datadir "C:/Program Files/PostgreSQL/9.6/data"
        --new-datadir "C:/Program Files/PostgreSQL/12/data"
        --old-bindir "C:/Program Files/PostgreSQL/9.6/bin"
        --new-bindir "C:/Program Files/PostgreSQL/12/bin"
```

#### pg_waldump

- 功能
  - 以人类可读的形式显示一个PostgreSQL 数据库集簇的预写式日志;
  - 通过waldump，结合grep，可以从wal文件过滤是否存在特定的数据库操作；
- 示例

```shell
[coder@localhost build_postgres]$ ./bin/pg_waldump  data/pg_wal/000000010000000000000049 |head -n 10
rmgr: Standby     len (rec/tot):     50/    50, tx:          0, lsn: 0/49000028, prev 0/48000138, desc: RUNNING_XACTS nextXid 252504 latestCompletedXid 252503 oldestRunningXid 252504
rmgr: Standby     len (rec/tot):     50/    50, tx:          0, lsn: 0/49000060, prev 0/49000028, desc: RUNNING_XACTS nextXid 252504 latestCompletedXid 252503 oldestRunningXid 252504
rmgr: XLOG        len (rec/tot):    114/   114, tx:          0, lsn: 0/49000098, prev 0/49000060, desc: CHECKPOINT_ONLINE redo 0/49000060; tli 1; prev tli 1; fpw true; xid 0:252504; oid 24650; multi 1; offset 0; oldest xid 479 in DB 1; oldest multi 1 in DB 1; oldest/newest commit timestamp xid: 0/0; oldest running xid 252504; online
rmgr: Standby     len (rec/tot):     50/    50, tx:          0, lsn: 0/49000110, prev 0/49000098, desc: RUNNING_XACTS nextXid 252504 latestCompletedXid 252503 oldestRunningXid 252504
rmgr: XLOG        len (rec/tot):     30/    30, tx:          0, lsn: 0/49000148, prev 0/49000110, desc: NEXTOID 32842
rmgr: Storage     len (rec/tot):     42/    42, tx:          0, lsn: 0/49000168, prev 0/49000148, desc: CREATE base/13593/24650
rmgr: XLOG        len (rec/tot):     49/  8241, tx:     252504, lsn: 0/49000198, prev 0/49000168, desc: FPI_FOR_HINT , blkref #0: rel 1663/13593/1247 fork fsm blk 2 FPW
rmgr: Heap2       len (rec/tot):     60/    60, tx:     252504, lsn: 0/490021E8, prev 0/49000198, desc: NEW_CID rel 1663/13593/1247; tid 9/7; cmin: 0, cmax: 4294967295, combo: 4294967295
rmgr: Heap        len (rec/tot):     54/  1338, tx:     252504, lsn: 0/49002228, prev 0/490021E8, desc: INSERT off 7 flags 0x01, blkref #0: rel 1663/13593/1247 blk 9 FPW
rmgr: Btree       len (rec/tot):     53/  1073, tx:     252504, lsn: 0/49002768, prev 0/49002228, desc: INSERT_LEAF off 49, blkref #0: rel 1663/13593/2703 blk 2 FPW
```

#### postgres

- 功能
  - PostgreSQL数据库服务器；
- 示例

```shell
[coder@localhost build_postgres]$ /home/coder/workspace/build_postgres/bin/postgres -D data
2024-01-31 11:32:43.694 CST [2761] LOG:  starting PostgreSQL 12.17 on x86_64-pc-linux-gnu, compiled by gcc (GCC) 4.8.5 20150623 (Red Hat 4.8.5-44), 64-bit
2024-01-31 11:32:43.700 CST [2761] LOG:  listening on IPv6 address "::1", port 5433
2024-01-31 11:32:43.700 CST [2761] LOG:  listening on IPv4 address "127.0.0.1", port 5433
2024-01-31 11:32:43.701 CST [2761] LOG:  listening on Unix socket "/tmp/.s.PGSQL.5433"
2024-01-31 11:32:43.715 CST [2762] LOG:  database system was shut down at 2024-01-31 11:32:38 CST
2024-01-31 11:32:43.718 CST [2761] LOG:  database system is ready to accept connections
```

#### postmaster -> postgres

- 功能
  - `postmaster`是`postgres`的一个废弃的别名，postgres的一个软连接;
- 示例

#### psql

- 功能
  -  PostgreSQL的交互式终端;
- 示例

```shell
[coder@localhost build_postgres]$ ./bin/psql -p 5433 -U system -d postgres
psql (12.17)
Type "help" for help.

postgres=# ag-0-1hlc17elnag-1-1hlc17eln
```

#### reindexdb

- 功能
  - 重建PostgreSQL数据库索引；
- 示例

```shell
[coder@localhost build_postgres]$ ./bin/reindexdb -h localhost -p 5433 -U system -d postgres
[coder@localhost build_postgres]$
```

#### vacuumdb

- 功能
  - 对一个PostgreSQL数据库进行垃圾收集和分析;
- 示例

```shell
[coder@localhost build_postgres]$ ./bin/vacuumdb -h localhost -p 5433 -U system -d postgres -e
SELECT pg_catalog.set_config('search_path', '', false);
vacuumdb: vacuuming database "postgres"
RESET search_path;
SELECT c.relname, ns.nspname FROM pg_catalog.pg_class c
 JOIN pg_catalog.pg_namespace ns ON c.relnamespace OPERATOR(pg_catalog.=) ns.oid
 LEFT JOIN pg_catalog.pg_class t ON c.reltoastrelid OPERATOR(pg_catalog.=) t.oid
 WHERE c.relkind OPERATOR(pg_catalog.=) ANY (array['r', 'm'])
 ORDER BY c.relpages DESC;
SELECT pg_catalog.set_config('search_path', '', false);
VACUUM public.tb;
VACUUM pg_catalog.pg_proc;
VACUUM pg_catalog.pg_depend;
VACUUM pg_catalog.pg_attribute;
VACUUM pg_catalog.pg_description;
VACUUM pg_catalog.pgag-0-1hlc17elnag-1-1hlc17eln_collation;
VACUUM pg_catalog.pg_statistic;
VACUUM pg_catalog.pg_class;
VACUUM pg_catalog.pg_operator;
VACUUM pg_catalog.pg_rewrite;
VACUUM pg_catalog.pg_type;
VACUUM information_schema.sql_features;
VACUUM pg_catalog.pg_amop;
VACUUM pg_catalog.pg_index;
VACUUM pg_catalog.pg_ts_config_map;
VACUUM pg_catalog.pg_amproc;
VACUUM pg_catalog.pg_conversion;
```

#### vacuumlo

- 功能
  
  - Vacuumlo从数据库中删除未引用的大型对象;

- 示例

```shell
[coder@localhost build_postgres]$ ./bin/vacuumlo -h localhost -p 5433 -U system postgres
[coder@localhost build_postgres]$ag-0-1hlc17elnag-1-1hlc17eln
```

### 小结

参考资料：

- http://www.postgres.cn/docs/12/reference-client.html
- http://www.postgres.cn/docs/12/reference-server.html
