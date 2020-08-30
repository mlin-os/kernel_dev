### Reading List

1. [CFS design](https://www.kernel.org/doc/html/latest/scheduler/sched-design-CFS.html)
2. [CFS rbtree](http://www.cs.montana.edu/~chandrima.sarkar/AdvancedOS/CSCI560_Proj_main/index.html)

### Question list

1. io scheduler type

```Bash
more /sys/block/sdX/queue/scheduler
noop [dealine] cfq
```

What are these above separately?

2. What are the  purposes of these config options?

* CONFIG_PREEMPT_NONE
* CONFIG_PREEMPT_VOLUNTARY
* CONFIG_PREEMPT
* CONFIG_PREEMPT_RT

3. What's PREEMPT_RT?

4. What's scheduling class and How does it work?

see [this page](https://developer.toradex.com/knowledge-base/real-time-linux) for basic knowledge

### FAQ

1. [What are the differencies of __CONFIG_PREEMPT_NONE__, __CONFIG_PREEMPT_VOLUNTARY__ and __CONFIG_PREEMPT__](https://www.oreilly.com/library/view/mastering-embedded-linux/9781784392536/ch14s04.html)?

> * __CONFIG_PREEMPT_NONE__: no preemption
> * __CONFIG_PREEMPT_VOLUNTARY__: enables additional checks for requests for preemption
> * __CONFIG_PREEMPT__: allows the kernel to be preempted
>
> With preemption set to __none__, kernel code will continue without rescheduling until it either returns via as `syscall` back to user space where preemption is always allowed, or it encounters a sleeping wait which stops the current thread.
>
> The second option enables explicit preemption points, where the scheduler is called if the `need_resched` flag is set, which reduces the worst-case preemption latencies at the expense of slightly lower throughput.
>
> The third option makes the kernel preemptible, meaning that an interrupt can result in an immediate reschedule so long as the kernel is not executing in an atomic context.

A few comparisons of latency and throughput of these options can be found in [this blog](https://www.codeblueprint.co.uk/2019/12/23/linux-preemption-latency-throughput.html).

2. [How to check the target latency (minimum time slice) on a linux system](https://www.kernel.org/doc/html/latest/scheduler/sched-design-CFS.html)?

```Bash
cat /proc/sys/kernel/sched_min_granularity_ns
30000000
```

3. [what's the difference between process scheduler and IO scheduler](https://unix.stackexchange.com/questions/199265/relationship-between-io-scheduler-and-cpu-process-scheduler)?

> They schedule different shared resources. IO scheduler orders the requests going to the disks, and CPU scheduler schedules the 'requests' to the CPU.

### [Linux scheduling goals](https://people.eecs.berkeley.edu/~kubitron/courses/cs194-24-S14/hand-outs/linuxKernelUnderstandingQueudet.pdf)

1. Linux's target market and their effects on its scheduler

Linux is known as a server operating system at the beginning and as the time goes on, it makes a success on the desktop area.

2. Efficiency

Efficiency is an important goal of Linux scheduler, which means it tries to have as much real work as possible to be done while staying within the restraints of other requirements. Generally, since context switching is expensive, allowing tasks to run for longer periods of time increases efficiency.

3. Interactivity

Interactivity is another important goal for the Linux scheduler, espeically given the growing demands from desktop environments. Interactivity often flies in the face of efficiency.

4. Fairness and Preventing Starvation

Starvations means a thread is not allowed to run for an unacceptably period of time due the prioritization of other thread over it. Fairness does not mean that every thread should have the same degree of access to the CPU time with the same priority, but it means that no thread should ever starve.

5. SMP scheduling

There is little reason to prefer one CPU over another in terms of choosing where to schedule a task. The most conspicuous consideration is caching - by scheduling a given task on the same CPU as often as possible.

6. etc.

### [O(1) scheduler](http://books.gigatux.nl/mirror/kerneldevelopment/0672327201/ch04lev1sec2.html)

1. Runqueues

The runqueue data structure is at the core of O(1) scheduler whose job is to keep track of a CPU's special thread information and to handle its two priority arrays. It works as
* each runqueue contains two priority arrays, the active one and the expired one;
* one runqueue is created and maintained for each CPU and each process runs exactly on one runqueue;
* At the beginning, all tasks begin in one priority array, the active one, and as they run out of their timeslices they are moved to the expired priority array;
* During the move, a new timeslice is calculated;
* When there are no more runnable tasks in the active array, it's simply swapped with the expired priority array.

```C
struct runqueue {
        spinlock_t          lock;   /* spin lock that protects this runqueue */
        unsigned long       nr_running;         /* number of runnable tasks */
        unsigned long       nr_switches;        /* context switch count */
        unsigned long       expired_timestamp;    /* time of last array swap */
        unsigned long       nr_uninterruptible;   /* uninterruptible tasks */
        unsigned long long  timestamp_last_tick;  /* last scheduler tick */
        struct task_struct  *curr;                /* currently running task */
        struct task_struct  *idle;           /* this processor's idle task */
        struct mm_struct    *prev_mm;        /* mm_struct of last ran task */
        struct prio_array   *active;         /* active priority array */
        struct prio_array   *expired;        /* the expired priority array */
        struct prio_array   arrays[2];       /* the actual priority arrays */
        struct task_struct  *migration_thread; /* migration thread */
        struct list_head    migration_queue;   /* migration queue*/
        atomic_t            nr_iowait; /* number of tasks waiting on I/O */
};
```

2. Priority Arrays

This data strcture achieves the scheduler's O(1) time performance. It works as

* The scheduler always schedules the task with the highest priority;
* If multiple tasks exist at the same priority level, they are scheduled in round-robin order;
* Priority arrays are an array of linked lists, one for each priority level. Thre are 140 priority levels in total. When a new task is added to a priority array, it's added to the list for its priority level;
* A bitmap is used with each bits set for each priority level that contains active tasks. When choosing a new task to run, the bitmap is checked first to find the first bit set in the bitmap;

```C
struct prio_array {
        int               nr_active;         /* number of tasks in the queues */
        unsigned long     bitmap[BITMAP_SIZE];  /* priority bitmap */
        struct list_head  queue[MAX_PRIO];      /* priority queues */
};
```

![How priority array work](./figures/O1_scheduler.png)

The following code determines the highest priority task:
```C
struct task_struct *prev, *next;
struct list_head *queue;
struct prio_array *array;
int idx;


prev = current;
array = rq->active;
idx = sched_find_first_bit(array->bitmap);
queue = array->queue + idx;
next = list_entry(queue->next, struct task_struct, run_list);
```

The following code do the priority array swapping:
```C
struct prio_array *array = rq->active;
if (!array->nr_active) {
        rq->active = rq->expired;
        rq->expired = array;
}
```

3. Calculating priority and timeslice

* Static task priority
* Dynamic task priority
* I/O-bound vs. CPU-bound Heuristics
* Calucting timeslice

### Comletely fair scheduler (CFS)

#### Reading list

1. https://developer.ibm.com/tutorials/l-completely-fair-scheduler/

#### Overview

> CFS basically model an "ideal, prescise multi-tasking CPU" on real hardware.
>
> "Ideal multi-tasking CPU" is a CPU that has 100% physical power and which can run each task at precise equal speed, in parallel, each at 1/nr_running speed.
>
> "Virual runtime" specifies when its next timeslice would start execution on the ideal multi-tasking CPU. In practicw, the virtual runtime of a task is its`actual runtime normalized to the total number of running tasks.

#### The rbtree

![rbtree](./figures/cfs_rbtree.png)
