## PostgreSQL的锁

pg中的锁可以分为3个层次：

### 自旋锁（Spin Lock）
1. 介绍

&emsp;&emsp;自旋锁顾名思义就是一直原地旋转等待的锁。一个进程如果想要访问临界区，必须先获得锁资源，如果不能获得，就会一直自旋，直到获取到锁资源。

&emsp;&emsp;显然这种自旋会造成CPU浪费，但是通常它保护的临界区非常小，封锁时间很短，因此通常自旋比释放CPU资源带来的上下文切换消耗要小。

&emsp;&emsp;如果机器拥有TAS（test-and-set）指令集，则pg会使用s_lock.h、s_lock.c中定义的实现机制。如果没有，则要用pg定义的信号量PGSemaphore来仿真SpinLock（在spin.h、spin.c文件中），显然后者效率不如前者。

2. 常用接口   
   1. 不依赖机器
        - #define SpinLockInit(lock)	S_INIT_LOCK(lock)
        - #define SpinLockAcquire(lock) S_LOCK(lock)
        - #define SpinLockRelease(lock) S_UNLOCK(lock)
        - #define SpinLockFree(lock)	S_LOCK_FREE(lock)

     1. 依赖机器
        - int S_LOCK(slock_t *lock)
        - void S_UNLOCK(slock_t *lock)
        - bool S_LOCK_FREE(slock_t *lock)
        - void SPIN_DELAY(void)
> 通常，S_LOCK()是通过更低级别的宏TAS()和TAS_SPIN()实现的：
> int TAS(slock_t *lock)
> 	原子测试并设置指令。尝试获取锁，但不会等待。如果成功，返回0；如果无法获取锁，返回非零值。
> int TAS_SPIN(slock_t *lock)
> 	与TAS()类似，但该版本用于等待先前被认为有争用的锁。默认情况下，与TAS()相同，
>     但在某些体系结构上，最好使用未锁定的指令轮询有争用的锁，并在它看起来空闲时再次尝试原子测试并设置。
> TAS()和TAS_SPIN()不是API的一部分，不应直接调用。

1. 特点
    - 是一种和硬件结合的互斥锁；
    - 在pg中，提供两种自旋锁，一种是硬件相关的，一种是硬件无关的；
    - 适用于临界区比较小的情况；
    - 封锁时间端；
    - 无死锁检测机制；
    - 无等待队列；
    - 事务结束时，不会自动释放锁；

### 轻量锁（Lightweight Lock）
1. 介绍
负责保护共享内存中的数据结构，有共享和排他两种模式，类似Oracle中的latch。特点是：封锁时间较短、无死锁检测机制、有等待队列、事务结束时会自动释放。

2. 常用接口

3. 特点


### 常规锁（Regular Lock）
1. 介绍
就是通常说的对数据库对象的锁。按照锁粒度，可以分为表锁、页锁、行锁等；按照等级，pg锁一共有8个等级。特点是：封锁时间可以很长、有死锁检测机制和等待队列、事务结束时会自动释放。

2. 常用接口
   
3. 特点
   

