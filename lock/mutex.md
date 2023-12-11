# Mutex's Mechanism and Implementation

## Introduction

## The interface of mutex

## The implementation of mutex

### The definition of `struct mutex`

```c
struct mutex {
        atomic_long_t           owner;
        raw_spinlock_t          wait_lock;
#ifdef CONFIG_MUTEX_SPIN_ON_OWNER
        struct optimistic_spin_queue osq; /* Spinner MCS lock */
#endif    
        struct list_head        wait_list;
#ifdef CONFIG_DEBUG_MUTEXES
        void                    *magic;
#endif
#ifdef CONFIG_DEBUG_LOCK_ALLOC
        struct lockdep_map      dep_map;
#endif
};
```

其中，_owner_保存了当前持锁的任务的`struct task_struct`信息，_wait_lock_用来保护睡眠队列_wait_list_，该队列入队了所有需要睡眠的等锁的任务，_osq_指向一个mcs队列的队尾，该队列入队了暂时可以自旋等锁的任务，剩下的成员变量都是用于调试的，本文暂不涉及。

可以看出`struct mutex`的定义与`struct semaphore`有较大的相似性，区别在_osq_域。

### The optimistic spin mechanism

**optimistic spin**直译过来是乐观自旋，它引入的目的是提升**binary semaphore**性能。在满足持锁任务能较快释放锁的情况下，通过短时间的自旋代替更高代价的睡眠从而更快的获得锁，提升系统性能，这也optimistic的由来。

#### Why introduce optimistic spin path?

主要是以下两个方面：

1. 获取不到**binary semaphore**的任务都会直接入队并进行睡眠，它的代价包括：两次上下文的切换开销，切换出去以及切换回来。并且，睡眠任务被唤醒后并不能立刻得到执行，需要被调度器调度到后才可以，存在调度产生的延迟。而自旋则没有这些开销，如果锁的当前持有者能很快释放，那么自旋稍作等待则显然更能提升性能。

2. 另外，从cache角度，上下文切换会导致cache的刷新，从而导致性能下降，而采用自旋的方式，cache是hot的，命中率更高。

#### The condition of optimistic spin?

那么什么条件下进行乐观自旋呢？合理的假设是**如果当前持锁线程正在运行的话，那么它大概率会很快释放锁**。



### The handoff mechanism

### The lock operation

### The unlock operation



## The priority inversion problem

![priority inversion problem](../figures/priority_inversion.png)

假设存在三个独立的线程P1，P2和P3，优先级满足P1 > P2 > P3，并且P1和P3共享一个通过mutex保护的资源。

某时刻，P3得到调度执行，在执行**P(mutex)**操作后进入临界区_CS-3_。时刻a，P2就绪，因为优先级高于P3，因此P2抢占P3运行。时刻b，P1就绪，因为优先级高于P2，因此抢占P2接着运行。时刻c，P1尝试执行**P(mutex)**进入临界区_CS-1_，但因为此时mutex在P3手上，因此P1阻塞，调度器选择此时优先级最高的任务P2继续执行。时刻d，P2执行完，在P2执行完后，调度器接着选择此时优先级最高的任务P3执行。e时刻，P3执行**V(mutex)**操作退出临界区，并唤醒P1，此时P1抢占P3执行，得以进入_CS-1_。后面的预期行为是P1完成剩余任务，然后P3得到调度直到执行完。

可以看到整个过程中，任务执行完成的顺序为P2 > P1 > P3，而根据优先级原则，高优先级的任务应当先执行，预期的执行顺序应该是P1> P2 > P3，因此出现了**priority inversion problem**。