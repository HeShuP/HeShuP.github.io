postgresql的两阶段提交；

在什么场景下会用到？？ 分布式部署？？

两阶段提交：支持分布式数据库的事务处理；

分布式处理：预提交阶段和全局提交阶段；

GID,全局ID；


PG 提供了一组命令用于 2PC 事务，如下：
prepare transaction 'xxx'
commit prepared 'xxx'
rollback prepared 'xxx'
