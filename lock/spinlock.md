# spinlock
## Introduction
## The evoluation of the implementation of spinlock

spinlock在同步机制中虽然比较简单，但是也经历过多次迭代，从而逐步提升其性能。主要的阶段可以归纳如下：

| Stage            | Implementation                                               | Adavantages and Disadvantages                                |
| ---------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 单整型变量的自旋 | 基于Compare and Swap、Test and Set等原子指令                 | 1. 实现简单，2. 不保证公平性，先到并不一定先获得，3. 存在cache-line bounce问题。 |
| ticket spinlocks | 队列化，类似银行柜台叫号，只有柜台号码与客户号码一致时才被服务 | 1. 解决公平性问题，2. 存在cache-line bounce问题。            |
| MCS locks        | 本地化，实现上是单链表，自旋于本地变量                       | 1. 解决cache-line bounce问题，2. 但锁头结构体增大，相比ticket spinlocks，从4字节增加到16字节。 |
| qspinlocks       | 基于MCS locks，压缩队尾节点信息，保持锁头大小为4字节         | 1. 解决锁头增大问题，2. 对少竞争场景并不建立MCS队列。        |
| ...              | ...                                                          | ...                                                          |



### Spin on an integer

思路比较简单，需要进入临界区的线程，在一个原子操作中，完成检查整型变量是否是解锁状态，如果是，则将整型变量修改为解锁状态，如果不是，则重新开始检查过程。

当前硬件上支持的主要有两种，一种是基于Compare and Swap操作，另一种是基于Test and Set操作。

#### What is Compare and Swap?
`cas`是compare and swap的缩写，根据[wikipedia](https://en.wikipedia.org/wiki/Compare-and-swap)上的定义：

> It compares the contents of a memory location with a given value and, only if they are the same, modifies the contents of that memory location to a new given value. This is done as a single atomic operation.

它的作用是比较给定的某个内存地址上的值与给定值是否相等，只有在相等的时候，将新值保存在该内存地址上，整个过程是一个原子操作。它常常用在多线程环境，用来实现同步。另外，它也有两种方式来判断是否成功执行了替换：

* 使用true或者false返回值，可以使用形如以下[伪代码](https://en.wikipedia.org/wiki/Compare-and-swap)来表示：
```pseudocode
function cas(p: pointer to int, old: int, new: int) is
    if *p ≠ old
        return false

    *p ← new

    return true
```

* 返回old值，形如以下伪代码：
```pseudocode
function cas(p: pointer to int, old: int, new: int) is
    ret: int ← old
    if *p = old
        *p ← new

    return ret
```

#### The implementation on sh platform
当前Linux内核中，只有[SuperH](https://en.wikipedia.org/wiki/SuperH)芯片平台上的spinlock是使用该方式实现的，它的加锁/解锁过程代码如下：

```c
// arch/sh/include/asm/spinlock-cas.h
static inline void arch_spin_lock(arch_spinlock_t *lock)
{
        while (!__sl_cas(&lock->lock, 1, 0));
}

static inline void arch_spin_unlock(arch_spinlock_t *lock)
{
        __sl_cas(&lock->lock, 0, 1);
}
```

可以看到都是基于`__sl_cas`函数，而它的实现如下：

```c
// arch/sh/include/asm/spinlock-cas.h
static inline unsigned __sl_cas(volatile unsigned *p, unsigned old, unsigned new)
{
        __asm__ __volatile__("cas.l %1,%0,@r0"
                : "+r"(new)
                : "r"(old), "z"(p)
                : "t", "memory" );
        return new;
}
```

查阅[Inline Assembly Language in C code](https://gcc.gnu.org/onlinedocs/gcc/extensions-to-the-c-language-family/how-to-use-inline-assembly-language-in-c-code.html)可知，整个函数只有一条汇编语句`cas.l %1, %0, @r0`，`%1`对应old，`%0`对应new，`%2`对应p。根据[J-core](https://lists.j-core.org/pipermail/j-core/2016-August/000346.html)论坛上的解释：

> There is an atomic compare-and-swap instruction cas.l Rm,Rn, at R0 that
>   compares the value at address R0 with Rm and, if equal, stores the
>   value Rn. Either way, afterwards Rn contains the old value that was
>   read. The T flag is also set to indicate success/failure (I believe
>   T=1 on success but I'd have to check)

它执行的是将R0寄存器（函数传参的第一个参数，也就是p）指向地址上的值与old比较，如果相等，则将new填入p指向的地址上。另外，无论是否执行替换，new中都保存的是p指向地址上的旧值（对应前述第二种判断是否替换的方式），这正是`cas`指令的定义。

基于以上的理解，我们回头看`arch_spin_lock`表示的意义：

当`lock->lock`中的值等于1时，将0填入`lock->lock`，并返回1，`arch_spin_lock`跳出while循环并返回，表示加锁成功。当`lock->lock`中的值不等于1时（那么必然为0），则不修改`lock->lock`值，并返回0，`arch_spin_lock`while死循环，表示等锁。

而`arch_spin_unlock`表示的意义：

当`lock->lock`中的值等于0时，将1填入`lock->lock`，并返回0，`arch_spin_unlock`返回，表示解锁成功。当`lock->lock`中的值等于1时，则直接返回1，`arch_spin_unlock`返回，无事发生，符合对处于解锁状态的spinlock再执行解锁时的预期。

#### What is Test and Set?

略，详见[Test and Set](https://en.wikipedia.org/wiki/Test-and-set)，Windows的`xchg`指令也是这一用途，请参考[spinlock](https://en.wikipedia.org/wiki/Spinlock)给出的x86上的例子：

```assembly
; Intel syntax

locked:                      ; The lock variable. 1 = locked, 0 = unlocked.
     dd      0

spin_lock:
     mov     eax, 1          ; Set the EAX register to 1.
     xchg    eax, [locked]   ; Atomically swap the EAX register with
                             ;  the lock variable.
                             ; This will always store 1 to the lock, leaving
                             ;  the previous value in the EAX register.
     test    eax, eax        ; Test EAX with itself. Among other things, this will
                             ;  set the processor's Zero Flag if EAX is 0.
                             ; If EAX is 0, then the lock was unlocked and
                             ;  we just locked it.
                             ; Otherwise, EAX is 1 and we didn't acquire the lock.
     jnz     spin_lock       ; Jump back to the MOV instruction if the Zero Flag is
                             ;  not set; the lock was previously locked, and so
                             ; we need to spin until it becomes unlocked.
     ret                     ; The lock has been acquired, return to the calling
                             ;  function.

spin_unlock:
     xor     eax, eax        ; Set the EAX register to 0.
     xchg    eax, [locked]   ; Atomically swap the EAX register with
                             ;  the lock variable.
     ret                     ; The lock has been released.
```

#### Advantages and Disadvantages

显然，该方式非常简单，容易理解。但是它也有两个比较大的缺陷：

1. 不保证公平性，所有参与自旋的线程谁先将公共变量置为加锁状态谁得到锁，而非谁先到先获取锁，极端情况可能出现长时间获取不到锁。
2. 存在cache-line bounce问题，高竞争下，性能大幅降低。

### ticket spinlock

为了解决公平性问题，思路就是队列化。

### MCS locks
### qspinlocks
### more ...
## History
Date | Adance
----|-----
2023/10/03 | 完成基于`cas`指令小节 
2023/10/04 | 补充test and set内容 