### Basic backgroud

#### What's the purposes of system calls?

> First, it provides an abstracted hardware interface for userspace.
> 
> Second, system calls ensure system security and stability.
> 
> Finally, a single common layer between user-sapce and the rest of the system allows for the virtualized system provided to processes.

### Details by an example  

#### An example C source code

```C
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>

int main(void)
{
        pid_t pid;
        pid = getpid();
        printf("the current pid is %ld\n", pid);

        return 0;
}
```

#### [How to get the location of C standard libary](https://stackoverflow.com/questions/5925678/location-of-c-standard-library?answertab=votes#tab-top)?

```Bash
mu@ustc:~$ gcc --print-file-name=libc.a
/usr/lib/gcc/x86_64-linux-gnu/9/../../../x86_64-linux-gnu/libc.a
```

### How to detect which dynamic libraries are linked?

```Bash
mu@ustc:~/test/c/strace$ ldd test 
	linux-vdso.so.1 (0x00007ffe8f917000)
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007fc70db08000)
	/lib64/ld-linux-x86-64.so.2 (0x00007fc70dcec000)
```

#### How to get the assembly code from a lib?

```Bash
objdump -d /lib/x86_64-linux-gnu/libc.so.6

# e.g. Here is the assembly code of function __getpid() got from libc.so.6
00000000000cbb80 <__getpid@@GLIBC_2.2.5>:
   cbb80:       b8 27 00 00 00          mov    $0x27,%eax
   cbb85:       0f 05                   syscall
   cbb87:       c3                      retq
   cbb88:       0f 1f 84 00 00 00 00    nopl   0x0(%rax,%rax,1)
   cbb8f:       00
```
Here 0x27 is the system call number of `getpid()` and passed to kernel space by register __eax__. The switch from user space to kernel is completed by __syscall__ instruction.

#### What happened after entering kernel space?

Here is an awesome explaination from [linux-insides progject](https://github.com/0xAX/linux-insides/blob/master/SysCall/linux-syscall-2.md).
