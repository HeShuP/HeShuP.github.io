## bgwriter与checkpointer进程对比

bgwriter进程与checkpointer进程，虽然都会写出脏页，但是刷脏的目的、频率、控制参数等，都有不同，以下是两则的区别：

| 区别                | bgwriter进程                                                                         | checkpointer进程                                     |
|:----------------- |:---------------------------------------------------------------------------------- |:-------------------------------------------------- |
| 当前，该进程刷脏的目的       | 1、保证有足够多的干净页面可以使用，提供数据查询的性能。<br/>2、通过提前将一部分脏数据落盘，可以减少checkpoint操作时的IO操作，使系统IO趋于平稳。 | 1、保证数据的一致性，将所有脏块刷出到磁盘，并创建一个数据一致性的位点，以便在数据崩溃后进行恢复。  |
| 执行频率              | 默认200ms                                                                            | 默认5min                                             |
| 每次刷出脏块            | 默认100                                                                              | 全部脏块                                               |
| 脏页刷出策略            | Postgres 8.1版本之前是用的LRU算法，从8.1版开始，Postgres使用了Clock Sweep时钟扫描算法                      | 无，刷出全部脏块。                                          |
| 是否更新pg_control文件? | 否                                                                                  | 是                                                  |
| 该进程在哪个版本引入?       | Postgres 8.0                                                                       | Postgres 9.2 (在8.0-9.1版本中，由bgwriter进程处理checkpoint) |

### checkpointer进程分离出来前

每隔几分钟就要运行一次检查点，将全部脏页写入操作系统的缓存区，然后再将操作系统缓冲区的所有脏页刷新到磁盘，会发出一个全局的sync调用，这将导致磁盘IO使用量的周期性激增，通常会影响性能。

bgwriter进程既需要负责执行checkpoint操作，还需要保证缓冲区有足够的干净页面可以使用，越来越臃肿，不便于性能调优。

以下代码片段来自Postgres 9.1版本：

```c
/* checkpoint刷脏 */
void CheckPointBuffers(int flags)
{
    TRACE_POSTGRESQL_BUFFER_CHECKPOINT_START(flags);
    CheckpointStats.ckpt_write_t = GetCurrentTimestamp();
    BufferSync(flags);
    CheckpointStats.ckpt_sync_t = GetCurrentTimestamp();
    TRACE_POSTGRESQL_BUFFER_CHECKPOINT_SYNC_START();
    smgrsync();
    CheckpointStats.ckpt_sync_end_t = GetCurrentTimestamp();
    TRACE_POSTGRESQL_BUFFER_CHECKPOINT_DONE();
}
/* 全局sync,极大的增加了IO */
void smgrsync(void)
{
    int            i;

    for (i = 0; i < NSmgr; i++)
    {
        if (smgrsw[i].smgr_sync)
            (*(smgrsw[i].smgr_sync)) ();
    }

}
```

### checkpointer进程分离出来后

bgwriter以稳定的速度进行脏块的写入操作。当检查点触发时，需要写的脏页会少很多，fsync也只需考虑只上一次检查点以来产生的脏块，提高了性能，并且最大限度的减少检查点期间的性能降低。

由于checkpointer进程和bgwriter进程之间的职责分离，系统管理员可以更方便地进行优化和调整，以满足特定的性能要求。

## 关于bgwriter进程

在Postgres 8.0版本中引入该进程。

### 刷脏的目的

- ​    保证有足够多的干净页面可以使用，提供数据查询的性能。
- ​    通过提前将一部分脏数据落盘，可以减少checkpoint操作时的IO操作，使系统IO趋于平稳。

​    仅将部分脏块刷出到磁盘。

### guc控制参数

```shell
# - Background Writer -

bgwriter_delay = 200ms                  # 10-10000ms between rounds
bgwriter_lru_maxpages = 100             # max buffers written/round, 0 disables
bgwriter_lru_multiplier = 2.0           # 0-10.0 multiplier on buffers scanned/round
bgwriter_flush_after = 512kB            # measured in pages, 0 disables
```

## 关于checkpointer进程

在Postgres 9.2中，将checkpoint处理的部分从bgwriter中剥离出来，引入checkpointer进程，专门用于处理checkpoint，而bgwriter进程不再处理检查点。

每次会将全部脏块集中刷出盘磁盘，对数据库性能影响较大。

### 刷脏的目的

保证数据的一致性，将所有脏块刷出到磁盘，并创建一个数据一致性的位点，以便在数据崩溃后进行恢复。

### guc控制参数

```shell
# - Checkpoints -

#checkpoint_timeout = 5min              # range 30s-1d
max_wal_size = 1GB
min_wal_size = 80MB
#checkpoint_completion_target = 0.5     # checkpoint target duration, 0.0 - 1.0
#checkpoint_flush_after = 256kB         # measured in pages, 0 disables
#checkpoint_warning = 30s               # 0 disables
```

## 为什么将checkpointer从bgwriter进程中分离出来？

- 早前版本，bgwriter进程同时执行后台写入、检查点和一些其他任务。因为只有一个进程，在做检查点的fsync的时候，不能进行bgwriter的写操作，两个操作不能同时进行，带来负面的性能影响。
- 此外，再9.2版本中，开始使用锁存器(latch)替代轮询的循环来降低功耗，而bgwriter循环的复杂性很高，不太可能使用锁存器替代循环。

关于分离checkpointer进程与bgwriter进程的讨论：
<https://www.postgresql.org/message-id/CA%2BU5nMLv2ah-HNHaQ%3D2rxhp_hDJ9jcf-LL2kW3sE4msfnUw9gA%40mail.gmail.com>

![split](https://github.com/HeShuP/HeShuP.github.io/raw/gh-pages/_posts/images/postgresql/split.png)

## 后续

由于bgwriter和checkpointer进程演进的历史原因，导致即便是在当前的Postgres 12版本中，bgwrter进程的代码中仍有大量checkpointer进程的影子，checkpoint的状态统计信息，仍在bgwriter的状态统计视图pg_stat_bgwriter中。但两个进程在Postgres版本的不断演进中，代码在不断的分离，演化成两个相对独立的进程。

在Postgres 12版本中，bgwriter的状态统计数据结构如下，从中能看到checkpoint的状态信息，两者并未完全分离。

```
typedef struct PgStat_MsgBgWriter
{
    PgStat_MsgHdr m_hdr;

    PgStat_Counter m_timed_checkpoints;
    PgStat_Counter m_requested_checkpoints;
    PgStat_Counter m_buf_written_checkpoints;
    PgStat_Counter m_buf_written_clean;
    PgStat_Counter m_maxwritten_clean;
    PgStat_Counter m_buf_written_backend;
    PgStat_Counter m_buf_fsync_backend;
    PgStat_Counter m_buf_alloc;
    PgStat_Counter m_checkpoint_write_time; /* times in milliseconds */
    PgStat_Counter m_checkpoint_sync_time;

} PgStat_MsgBgWriter;
```

在Postgres 15版本中，可以发现bgwriter和checkpointer的统计信息已经完全剥离。

```
typedef struct PgStat_BgWriterStats
{
    PgStat_Counter buf_written_clean;
    PgStat_Counter maxwritten_clean;
    PgStat_Counter buf_alloc;
    TimestampTz stat_reset_timestamp;
} PgStat_BgWriterStats;

typedef struct PgStat_CheckpointerStats
{
    PgStat_Counter timed_checkpoints;
    PgStat_Counter requested_checkpoints;
    PgStat_Counter checkpoint_write_time;    /* times in milliseconds */
    PgStat_Counter checkpoint_sync_time;
    PgStat_Counter buf_written_checkpoints;
    PgStat_Counter buf_written_backend;
    PgStat_Counter buf_fsync_backend;
} PgStat_CheckpointerStats;
```