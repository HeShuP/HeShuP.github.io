## 目的

本文主要介绍PostgreSQL数据库data目录下的各个文件夹的作用，仅做简单解释，并对部分举例。

## 详解

### 整体结构

PostgreSQL数据库的data目录，整个结构如下:

```shell
[coder@localhost build_postgres]$ tree data/ -L 1
data/
├── base
├── global
├── log
├── pg_commit_ts
├── pg_dynshmem
├── pg_hba.conf
├── pg_ident.conf
├── pg_logical
├── pg_multixact
├── pg_notify
├── pg_replslot
├── pg_serial
├── pg_snapshots
├── pg_stat
├── pg_stat_tmp
├── pg_subtrans
├── pg_tblspc
├── pg_twophase
├── PG_VERSION
├── pg_wal
├── pg_xact
├── postgresql.auto.conf
├── postgresql.conf
├── postmaster.opts
└── postmaster.pid
```

### 子目录

#### base

- 作用
  
  - 包含每个数据库目录，数据库目录以数据库的OID编号命名；
  - 其中命名为1的目录，对应模板数据库template1;

- 示例

```shell
  [coder@localhost data]$ ll base/
  总用量 76
  drwx------. 2 coder coder  8192 1月  31 11:22 1
  drwx------. 2 coder coder  8192 1月  17 14:50 13592
  drwx------. 2 coder coder 12288 1月  31 13:48 13593
  drwx------. 2 coder coder  8192 1月  31 13:47 24579
  drwx------. 2 coder coder  8192 1月  31 09:52 24648
  drwx------. 2 coder coder  8192 1月  31 11:22 24649

  postgres=# select * from pg_database;
  oid   |  datname  | datdba | encoding | datcollate  |  datctype   | datistemplate | datallowconn | datconnlimit | datlastsysoid | datfrozenxid | datminmxid | dattablespace |            datacl             
  -------+-----------+--------+----------+-------------+-------------+---------------+--------------+--------------+---------------+--------------+------------+---------------+-------------------------------
  13593 | postgres  |     10 |        6 | zh_CN.UTF-8 | zh_CN.UTF-8 | f             | t            |           -1 |         13592 |          479 |          1 |          1663 | 
  24579 | bm        |     10 |        6 | zh_CN.UTF-8 | zh_CN.UTF-8 | f             | t            |           -1 |         13592 |          479 |          1 |          1663 | 
     1  | template1 |     10 |        6 | zh_CN.UTF-8 | zh_CN.UTF-8 | t             | t            |           -1 |         13592 |          479 |          1 |          1663 | {=c/system,system=CTc/system}
  13592 | template0 |     10 |        6 | zh_CN.UTF-8 | zh_CN.UTF-8 | t             | f            |           -1 |         13592 |          479 |          1 |          1663 | {=c/system,system=CTc/system}
  24648 | database  |     10 |        6 | zh_CN.UTF-8 | zh_CN.UTF-8 | f             | t            |           -1 |         13592 |          479 |          1 |          1663 | 
  24649 | system    |     10 |        6 | zh_CN.UTF-8 | zh_CN.UTF-8 | f             | t            |           -1 |         13592 |          479 |          1 |          1663 | 
  (6 rows) 
```

#### global

- 作用
  - 存放整个cluster共享的全局表、控制文件等；
- 示例

```shell
  /* global目录下，存在如下文件 */
  [coder@localhost global]$ ls
  1136      1213      1214      1260      1261      1262      2396      2846  32856  32864  32934  32937  32940  32943  32946  32949  32973  3592  4177  4183  6100             pg_internal.init
  1136_fsm  1213_fsm  1214_fsm  1260_fsm  1261_fsm  1262_fsm  2396_fsm  2964  32857  32865  32935  32938  32941  32944  32947  32971  32981  4060  4179  4185  pg_control
  1136_vm   1213_vm   1214_vm   1260_vm   1261_vm   1262_vm   2396_vm   2966  32858  32866  32936  32939  32942  32945  32948  32972  32982  4175  4181  6000  pg_filenode.map
  [coder@localhost global]$

/* 查看文件1136对应的信息 */
postgres=# select * from pg_class where oid = 1136;
-[ RECORD 1 ]-------+----------------------------------
oid                 | 1136
relname             | pg_pltemplate
relnamespace        | 11
reltype             | 12028
reloftype           | 0
relowner            | 10
relam               | 2
relfilenode         | 0
reltablespace       | 1664
relpages            | 1
reltuples           | 8
relallvisible       | 1
reltoastrelid       | 4179
relhasindex         | t
relisshared         | t
relpersistence      | p
relkind             | r
relnatts            | 8
relchecks           | 0
relhasrules         | f
relhastriggers      | f
relhassubclass      | f
relrowsecurity      | f
relforcerowsecurity | f
relispopulated      | t
relreplident        | n
relispartition      | f
relrewrite          | 0
relfrozenxid        | 479
relminmxid          | 1
relacl              | {system=arwdDxt/system,=r/system}
reloptions          | 
relpartbound        | 
postgres=#
```

#### log

- 作用
  
  - 存储PostgreSQL数据库的运行日志；

- 示例

```shell
  [coder@localhost log]$ ll
  总用量 192
  -rw-------. 1 coder coder   168 1月  17 16:22 postgresql-2024-01-17_162255.log
  -rw-------. 1 coder coder  6589 1月  17 16:31 postgresql-2024-01-17_162537.csv
  -rw-------. 1 coder coder   168 1月  17 16:25 postgresql-2024-01-17_162537.log
  -rw-------. 1 coder coder  1216 1月  17 17:22 postgresql-2024-01-17_163111.csv
  -rw-------. 1 coder coder   168 1月  17 16:31 postgresql-2024-01-17_163111.log
  -rw-------. 1 coder coder  5829 1月  17 19:30 postgresql-2024-01-17_172203.csv
  -rw-------. 1 coder coder   168 1月  17 17:22 postgresql-2024-01-17_172203.log
  -rw-------. 1 coder coder  4266 1月  17 20:18 postgresql-2024-01-17_193718.csv
  -rw-------. 1 coder coder   168 1月  17 19:37 postgresql-2024-01-17_193718.log
```

#### pg_commit_ts

- 作用
  
  - 事务提交时间戳数据的子目录；
  - 需将GUC参数track_commit_timestamp，设置为on;

- 示例

```shell
  [coder@localhost build_postgres]$ ll data/pg_commit_ts/
  总用量 8
  -rw-------. 1 coder coder 172032 1月  31 14:54 0009
  [coder@localhost build_postgres]$
```

#### pg_dynshmem

- 作用
  
  - 包含被动态共享内存子系统所使用的文件的子目录；
  - 与GUC参数dynamic_shared_memory_type存在关联；

- 示例

```shell
/* 查看dynamic_shared_memory_type参数 */
postgres=# show dynamic_shared_memory_type;
dynamic_shared_memory_type 
 mmap
(1 row)
postgres=#

/* 查看pg_dynshmem目录 */
[coder@localhost build_postgres]$ ll data/pg_dynshmem/
总用量 16
-rw-------. 1 coder coder 16192 1月  31 15:04 mmap.2033756003
[coder@localhost build_postgres]$
```

#### pg_hba.conf

- 作用
  
  - 基于主机的访问控制文件，保存对客户端认证方式的配置信息；

- 示例

```shell
/* pg_hba.cof的主要内容如下 */
[coder@localhost data]$ cat pg_hba.conf
# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     trust
# IPv4 local connections:
host    all             all             127.0.0.1/32            trust
# IPv6 local connections:
host    all             all             ::1/128                 trust
# Allow replication connections from localhost, by a user with the
# replication privilege.
local   replication     all                                     trust
host    replication     all             127.0.0.1/32            trust
host    replication     all             ::1/128                 trust
```

#### pg_ident.conf

- 作用
  
  - 用户名映射文件，定义了操作系统用户名和PostgreSQL用户名之间的映射关系；
  
  - 这些映射关系，会被pg_hba.conf使用到；

- 示例

#### pg_logical

- 作用
  
  - 用于逻辑复制的状态数据的子目录；

- 示例

```shell
  [coder@localhost build_postgres]$ ll data/pg_logical/
  总用量 4
  drwx------. 2 coder coder  6 1月  17 14:50 mappings
  -rw-------. 1 coder coder  8 1月  31 15:09 replorigin_checkpoint
  drwx------. 2 coder coder 29 1月  31 09:36 snapshots
  [coder@localhost build_postgres]$
```

#### pg_multixact

- 作用
  
  - 多事务（multi-transaction）状态数据的子目录（用于共享的行锁）;
  - 当多个事务，同时对一行记录添加行级锁时，就是多事务；

- 示例

```shell
[coder@localhost data]$ tree pg_multixact/
pg_multixact/
├── members
│   └── 0000
└── offsets
    └── 0000

2 directories, 2 files
[coder@localhost data]$
```

#### pg_notify

- 作用
  
  - LISTEN/NOTIFY状态数据的子目录；
  - 在 SQL 标准中没有`NOTIFY`语句；

- 示例

```shell
[coder@localhost build_postgres]$ ll data/pg_notify/
总用量 8
-rw-------. 1 coder coder 8192 1月  31 15:04 0000

/* 在session1中，监听test_channel */
[coder@localhost build_postgres]$ ./bin/psql -p 5433 -U system -d postgres
psql (12.17)
Type "help" for help.

postgres=# LISTEN test_channel;
LISTEN
postgres=# LISTEN test_channel;
LISTEN
Asynchronous notification "test_channel" with payload "I am hesp" received from server process with PID 6164.
postgres=#

/* 在session2中，向监听test_channel通道发送信息 */
[coder@localhost build_postgres]$ ./bin/psql -p 5433 -U system -d postgres
psql (12.17)
Type "help" for help.

postgres=# NOTIFY test_channel, 'I am hesp';           
NOTIFY
postgres=#
```

#### pg_replslot

- 作用
  
  - 复制槽数据的子目录；
  - 复制操分为逻辑复制操和物理复制操，分别对应逻辑复制和物理复制；

- 示例

```
/* 创建复制槽test_slot */
postgres=# SELECT * FROM pg_create_physical_replication_slot('test_slot');   
 slot_name | lsn 
-----------+-----
 test_slot | 
(1 row)

postgres=# select * from pg_replication_slots;
 slot_name |    plugin     | slot_type | datoid | database | temporary | active | active_pid | xmin | catalog_xmin | restart_lsn | confirmed_flush_lsn 
-----------+---------------+-----------+--------+----------+-----------+--------+------------+------+--------------+-------------+---------------------
 node_slot | test_decoding | logical   |  13593 | postgres | f         | f      |            |      |       252502 | 0/43EFBDE8  | 0/43EFBDE8
 test_slot |               | physical  |        |          | f         | f      |            |      |              |             | 
(2 rows)

postgres=#

/* 查看pg_replslot目录 */
[coder@localhost build_postgres]$ tree  data/pg_replslot/
data/pg_replslot/
├── node_slot
│   └── state
└── test_slot
    └── state

2 directories, 2 files
```

#### pg_serial

- 作用
  - 已提交的可序列化事务信息的子目录；
- 示例

#### pg_snapshots

- 作用
  - 导出的快照的子目录；
- 示例

#### pg_stat

- 作用
  - 用于统计子系统的永久文件的子目录；
- 示例

#### pg_stat_tmp

- 作用
  - 用于统计信息子系统的临时文件的子目录；
- 示例

```shell
[coder@localhost data]$ ll pg_stat_tmp
总用量 64
-rw-------. 1 coder coder  7948 1月  31 16:35 db_0.stat
-rw-------. 1 coder coder 39889 1月  31 16:35 db_13593.stat
-rw-------. 1 coder coder  8455 1月  31 16:35 db_24579.stat
-rw-------. 1 coder coder   840 1月  31 16:35 global.stat
```

#### pg_subtrans

- 作用
  - 子事务状态数据的子目录；
  - 子事务是实现保存点的主要技术来了；
- 示例

```shell
[coder@localhost data]$ ll pg_subtrans
总用量 224
-rw-------. 1 coder coder 229376 1月  31 15:09 0003
[coder@localhost data]$
```

#### pg_tblspc

- 作用
  - 包含指向表空间的符号链接的子目录；
- 示例

#### pg_twophase

- 作用
  - 用于预备事务状态文件的子目录
- 示例

#### PG_VERSION

- 作用
  - 包含PostgreSQL主版本号的文件
- 示例

```shell
[coder@localhost data]$ cat PG_VERSION 
12
[coder@localhost data]$
```

#### pg_wal

- 作用
  - WAL （预写日志）文件的子目录；
- 示例

```shell
[coder@localhost data]$ ll pg_wal/ | head -n 10
总用量 868352
-rw-------. 1 coder coder 16777216 1月  31 09:36 000000010000000000000043
-rw-------. 1 coder coder 16777216 1月  31 09:36 000000010000000000000044
-rw-------. 1 coder coder 16777216 1月  31 10:04 000000010000000000000045
-rw-------. 1 coder coder 16777216 1月  31 10:04 000000010000000000000046
-rw-------. 1 coder coder 16777216 1月  31 10:07 000000010000000000000047
-rw-------. 1 coder coder 16777216 1月  31 10:07 000000010000000000000048
-rw-------. 1 coder coder 16777216 1月  31 16:39 000000010000000000000049
-rw-------. 1 coder coder 16777216 1月  30 19:22 00000001000000000000004A
-rw-------. 1 coder coder 16777216 1月  30 19:22 00000001000000000000004B
[coder@localhost data]$
```

#### pg_xact

- 作用
  
  - 事务提交状态数据的子目录；

- 示例
  
  ```shell
  [coder@localhost data]$ ll pg_xact/
  总用量 64
  -rw-------. 1 coder coder 65536 1月  31 16:38 0000
  [coder@localhost data]$
  ```

#### postgresql.auto.conf

- 作用
  - 存储由`ALTER SYSTEM` 设置的配置参数的文件；
  - 其配置参数的优先级，比postgresql.conf更高；
- 示例

```shell
[coder@localhost data]$ cat postgresql.auto.conf 
# Do not edit this file manually!
# It will be overwritten by the ALTER SYSTEM command.
full_page_writes = 'on'
dynamic_shared_memory_type = 'mmap'
[coder@localhost data]$
```

#### postgresql.conf

- 作用
  - PostgreSQL数据库的控制文件；
- 示例

#### postmaster.opts

- 作用
  - 记录服务器最后一次启动时使用的命令行参数的文件
- 示例

```shell
[coder@localhost data]$ cat postmaster.opts 
/home/coder/workspace/build_postgres/bin/postgres "-D" "data"
[coder@localhost data]$
```

#### postmaster.pid

- 作用
  - 一个锁文件，记录着当前的 postmaster 进程ID（PID）、集簇数据目录路径、postmaster启动时间戳、端口号、Unix域套接字目录路径（Windows上为空）、第一个可用的listen_address（IP地址或者`*`，或者为空表示不在TCP上监听）以及共享内存段ID（服务器关闭后该文件不存在）
- 示例

```shell
[coder@localhost data]$ cat postmaster.pid 
5538
/home/coder/workspace/build_postgres/data
1706684694
5433
/tmp
localhost
  5433001         5
ready   
[coder@localhost data]$
```

## 小结

- PostgreSQL数据库文件布局，可参考：[ http://www.postgres.cn/docs/12/storage-file-layout.html]([68.1. 数据库文件布局](http://www.postgres.cn/docs/12/storage-file-layout.html))
