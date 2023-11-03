# Semaphore

## Introduction

信号量是内核比较常用的同步方式之一，它的特点如下：

* 它允许资源数大于1，当资源还有剩余时，申请者不用等待即可进入临界区，而像spinlock和mutex任意时刻只允许一个申请者进入临界区。根据资源数的多少，semaphore分为binary semaphore，对应最大资源数为1，以及normal semaphore，对应最大资源数大于1，
* 当剩余资源数为0时，新的申请者会被block，进入sleep状态，从而把cpu让渡出去给其他任务，通常适用于临界区时间不那么短的任务，
* semaphore并不要求锁的释放者必须是锁的获得者，这是binary semaphore与mutex的重要区别。

## The interface of semaphore



## The implementation of semaphore

### The definition of lock

```c
// include/linux/semaphore.h
struct semaphore {
        raw_spinlock_t          lock;
        unsigned int            count;
        struct list_head        wait_list;
};
```

其中_lock_是用来保护锁内部数据完整性的，_count_记录当前资源的剩余数目，_wait_list_是个链表，这里是作为等待队列使用，队列中是所有等锁的线程。

### The Initialization of lock

* 静态方式

```c
// include/linux/semaphore.h
#define __SEMAPHORE_INITIALIZER(name, n)                                \
{                                                                       \
        .lock           = __RAW_SPIN_LOCK_UNLOCKED((name).lock),        \
        .count          = n,                                            \
        .wait_list      = LIST_HEAD_INIT((name).wait_list),             \
}

#define DEFINE_SEMAPHORE(_name, _n)     \
        struct semaphore _name = __SEMAPHORE_INITIALIZER(_name, _n)
```

* 动态方式

```c
// include/linux/semaphore.h
static inline void sema_init(struct semaphore *sem, int val)
{
        static struct lock_class_key __key;
        *sem = (struct semaphore) __SEMAPHORE_INITIALIZER(*sem, val);
        lockdep_init_map(&sem->lock.dep_map, "semaphore->lock", &__key, 0);
}
```

一目了然，现在两种方式都可以初始化资源数任意的semaphore。

## The priority inversion issue

## The deadlock issue

## History

| Date       | Adance     |
| ---------- | ---------- |
| 2023/11/01 | 初始化文档 |
|            |            |
|            |            |

