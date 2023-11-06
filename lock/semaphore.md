# Semaphore

## Introduction

信号量是内核比较常用的同步方式之一，它的特点如下：

* 它允许资源数大于1，当资源还有剩余时，申请者不用等待即可进入临界区，而像spinlock和mutex任意时刻只允许一个申请者进入临界区。根据资源数的多少，semaphore分为**binary semaphore**，对应最大资源数为1，以及**normal semaphore**，对应最大资源数大于1，
* 当剩余资源数为0时，新的申请者会被block，进入sleep状态，从而将cpu让渡出去给其他任务，通常适用于临界区时间不那么短的任务，
* 信号量并不要求资源的释放者必须是资源的获得者，这是binary semaphore与mutex的重要区别。

## The interface of semaphore

```c
// include/linux/semaphore.h

#define DEFINE_SEMAPHORE(_name, _n)     \
        ...

static inline void sema_init(struct semaphore *sem, int val)
{
        ...
}

extern void down(struct semaphore *sem);
extern int __must_check down_interruptible(struct semaphore *sem);
extern int __must_check down_killable(struct semaphore *sem);
extern int __must_check down_trylock(struct semaphore *sem);
extern int __must_check down_timeout(struct semaphore *sem, long jiffies);
extern void up(struct semaphore *sem);
```

以上是信号量对外接口，其中加锁的接口比较丰富（`down`以及其变种），用于支持对信号的不同需求。

## The implementation of semaphore

信号量的实现非常的简单易懂，其代码实现在_kernel/locking/semaphore.{c, h}_文件中。

### The definition of lock

```c
// include/linux/semaphore.h
struct semaphore {
        raw_spinlock_t          lock;
        unsigned int            count;
        struct list_head        wait_list;
};
```

其中_lock_是用来保护锁内部数据完整性的，_count_记录当前资源的剩余数目，_wait_list_是一个链表，这里是作为等待队列使用，队列中是所有等待资源的线程。

### The Initialization of lock

在Linux内核中，提供了以下两种方式定义和初始化信号量：

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

在定义变量的同时完成对变量的初始化。当前Linux主干代码已支持初始化资源数任意的信号量，更早一点的内核版本，`DEFINE_SEMAPHORE`并没有第二个参数，固定初始化为资源数为1的信号量，即binary semaphore。

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

该方式需要先定义变量，然后调用`sema_init`进行初始化，可以看到与静态方式并无二致。

### The lock operation

以下以`down`接口为例，讲解加锁操作的实现：

```c
void __sched down(struct semaphore *sem)              
{                                                     
        unsigned long flags;                          
                                                      
        might_sleep();                                
        raw_spin_lock_irqsave(&sem->lock, flags);     
        if (likely(sem->count > 0))                   
                sem->count--;                         
        else                                          
                __down(sem);                          
        raw_spin_unlock_irqrestore(&sem->lock, flags);
}                                                     

```

当还有剩余资源_sem->count > 0_时，直接_count_减1并返回，线程成功获得信号量。当剩余资源为0时，则调用`__down`接口进入等待流程，其实现如下：

```c
static noinline void __sched __down(struct semaphore *sem)             
{                                                                      
        __down_common(sem, TASK_UNINTERRUPTIBLE, MAX_SCHEDULE_TIMEOUT);
}                                                                      
```

可以看到它是`__down_common`的封装，而其他几种变种接口也是同样的：

```c
static noinline int __sched __down_interruptible(struct semaphore *sem)         
{                                                                               
        return __down_common(sem, TASK_INTERRUPTIBLE, MAX_SCHEDULE_TIMEOUT);    
}                                                                               
                                                                                
static noinline int __sched __down_killable(struct semaphore *sem)              
{                                                                               
        return __down_common(sem, TASK_KILLABLE, MAX_SCHEDULE_TIMEOUT);         
}                                                                               
                                                                                
static noinline int __sched __down_timeout(struct semaphore *sem, long timeout) 
{                                                                               
        return __down_common(sem, TASK_UNINTERRUPTIBLE, timeout);               
}                                                                               
```

区别在于：

1. 对信号的处理不同，设置线程状态为**TASK_UNINTERRUPTIBLE**后，等待线程不响应任何信号，直到有空闲信号量后主动唤起。**TASK_KILLABLE**和**TASK_INTERRUPTIBLE**这只是响应的信号不同。
2. 是否设置超时，**MAX_SCHEDULE_TIMEOUT**定义为**LONG_MAX**，即long整型定义的最大值，因为该值巨大，设置后意味着在合理范围，等待线程一直睡眠直到获得信号量，即不设置超时。

当前Linux主干代码又新增一层封装，`__down_common`是`___down_common`的封装：

```c
static inline int __sched __down_common(struct semaphore *sem, long state,
                                        long timeout)                     
{                                                                         
        int ret;                                                          
                                                                          
        trace_contention_begin(sem, 0);                                   
        ret = ___down_common(sem, state, timeout);                        
        trace_contention_end(sem, ret);                                   
                                                                          
        return ret;                                                       
}                                                                         
```

`trace_contention_begin`和`trace_contention_end`用于debug，不影响功能。`___down_common`是真正的等锁流程，其实现如下：

```c
static inline int __sched ___down_common(struct semaphore *sem, long state,  
                                                                long timeout)
{                                                                            
        struct semaphore_waiter waiter;                                      
                                                                             
        list_add_tail(&waiter.list, &sem->wait_list);                        
        waiter.task = current;                                               
        waiter.up = false;                                                   
                                                                             
        for (;;) {                                                           
                if (signal_pending_state(state, current))                    
                        goto interrupted;                                    
                if (unlikely(timeout <= 0))                                  
                        goto timed_out;                                      
                __set_current_state(state);                                  
                raw_spin_unlock_irq(&sem->lock);                             
                timeout = schedule_timeout(timeout);                         
                raw_spin_lock_irq(&sem->lock);                               
                if (waiter.up)                                               
                        return 0;                                            
        }                                                                    
                                                                             
 timed_out:                                                                  
        list_del(&waiter.list);                                              
        return -ETIME;                                                       
                                                                             
 interrupted:                                                                
        list_del(&waiter.list);                                              
        return -EINTR;                                                       
}                                                                            
```

首先，通过类型为`struct semaphore_waiter`的结构体变量_waiter_将当前线程加入了信号量的等待队列_sem->wait_list_，其次，通过`__set_current_state`接口将线程设置为指定状态，然后，通过`schedule_timeout`设置超时时间，调度出去进入睡眠，直到被`wake_up_process`流程主动唤起或者超时时间到了，更多细节请参考[schedule_timeout](https://linuxtv.org/downloads/v4l-dvb-internals/device-drivers/API-schedule-timeout.html) API。

当`up`流程释放信号量时，首先，队首等待线程的_waiter.up_域会被置位**true**，接着线程会被重新调度到并从`schedule_timeout`函数会返回接着运行（详见下一小节），此时会因为_waiter.up = true_而跳出循环，获得信号量。

当_timeout_设置的超时时间到达后，同样的，等待线程会被重新加入调度队列并从`schedule_timeout`返回，此时会走_**time_out**_流程，将自己从等待队列中删除。

当收到信号时，如果线程状态为**TASK_INTERRUPTIBLE**或者**TASK_KILLABLE**时，线程会在执行完对应信号的信号处理函数后被唤起，此时会进入对应_**interrupted**_流程，将自己从收到需要处理的预定信号时，等待队列中删除，如果线程状态为**TASK_UNINTERRUPTIBLE**，此时即便收到信号，也不会响应信号直到被主动唤起。

### The unlock operation

解锁的操作为`up`，它的实现如下：

```c
void __sched up(struct semaphore *sem)                
{                                                     
        unsigned long flags;                          
                                                      
        raw_spin_lock_irqsave(&sem->lock, flags);     
        if (likely(list_empty(&sem->wait_list)))      
                sem->count++;                         
        else                                          
                __up(sem);                            
        raw_spin_unlock_irqrestore(&sem->lock, flags);
}                                                     
```

如果锁的等待队列_sem->wait_list_为空，此时资源得到真正释放，_sem->count_自增，如果等待队列非空，则通过`__up`唤醒等待队列的第一个任务，使其获得信号量，如下所示：

```c
static noinline void __sched __up(struct semaphore *sem)                       
{                                                                              
        struct semaphore_waiter *waiter = list_first_entry(&sem->wait_list,    
                                                struct semaphore_waiter, list);
        list_del(&waiter->list);                                               
        waiter->up = true;                                                     
        wake_up_process(waiter->task);                                         
}                                                                              
```

首先，通过`list_first_entry`宏找到等待队列的第一个任务，其次，通过`list_del`宏将其从等待队列上摘下，然后置位_waiter->up_域，使得任务再次调度后可以退出`down`流程获得信号量，最后，做完以上准备工作后，通过`wake_up_process`将任务唤起。整个过程，清晰明了。

## The priority inversion issue

## The deadlock issue

## History

| Date       | Adance                   |
| ---------- | ------------------------ |
| 2023/11/01 | 初始化文档               |
| 2023/11/06 | 完成初始化/加锁/解锁小节 |
|            |                          |

