#### module related

##### [How to find the line in souce file of kdb error](https://www.opensourceforu.com/2011/01/understanding-a-kernel-oops/)

Here is an excerpt from above blog.

The defective code is as follows:
```C
#include <linux/kernel.h>
#include <linux/module.h>
#include <linux/init.h>

static void create_oops() {
        *(int *)0 = 0;       // line 6
}

static int __init my_oops_init(void) {
        printk("oops from the module\n");
        create_oops();
       return (0);
}
static void __exit my_oops_exit(void) {
        printk("Goodbye world\n");
}

module_init(my_oops_init);
module_exit(my_oops_exit);
```

The oops information is as follows:
```
BUG: unable to handle kernel NULL pointer dereference at (null)
IP: [<ffffffffa03e1012>] my_oops_init+0x12/0x21 [oops]
PGD 7a719067 PUD 7b2b3067 PMD 0
Oops: 0002 [#1] SMP
last sysfs file: /sys/devices/virtual/misc/kvm/uevent
CPU 1
Pid: 2248, comm: insmod Tainted: P           2.6.33.3-85.fc13.x86_64
RIP: 0010:[<ffffffffa03e1012>]  [<ffffffffa03e1012>] my_oops_init+0x12/0x21 [oops]
RSP: 0018:ffff88007ad4bf08  EFLAGS: 00010292
RAX: 0000000000000018 RBX: ffffffffa03e1000 RCX: 00000000000013b7
RDX: 0000000000000000 RSI: 0000000000000046 RDI: 0000000000000246
RBP: ffff88007ad4bf08 R08: ffff88007af1cba0 R09: 0000000000000004
R10: 0000000000000000 R11: ffff88007ad4bd68 R12: 0000000000000000
R13: 00000000016b0030 R14: 0000000000019db9 R15: 00000000016b0010
FS:  00007fb79dadf700(0000) GS:ffff880001e80000(0000) knlGS:0000000000000000
CS:  0010 DS: 0000 ES: 0000 CR0: 000000008005003b
CR2: 0000000000000000 CR3: 000000007a0f1000 CR4: 00000000000006e0
DR0: 0000000000000000 DR1: 0000000000000000 DR2: 0000000000000000
DR3: 0000000000000000 DR6: 00000000ffff0ff0 DR7: 0000000000000400
Process insmod (pid: 2248, threadinfo ffff88007ad4a000, task ffff88007a222ea0)
Stack:
ffff88007ad4bf38 ffffffff8100205f ffffffffa03de060 ffffffffa03de060
 0000000000000000 00000000016b0030 ffff88007ad4bf78 ffffffff8107aac9
 ffff88007ad4bf78 00007fff69f3e814 0000000000019db9 0000000000020000
Call Trace:
[<ffffffff8100205f>] do_one_initcall+0x59/0x154
[<ffffffff8107aac9>] sys_init_module+0xd1/0x230
[<ffffffff81009b02>] system_call_fastpath+0x16/0x1b
Code: <c7> 04 25 00 00 00 00 00 00 00 00 31 c0 c9 c3 00 00 00 00 00 00 00
RIP  [<ffffffffa03e1012>] my_oops_init+0x12/0x21 [oops]
RSP <ffff88007ad4bf08>
CR2: 0000000000000000
```

1. load the defective module into the GDB debugger

```gdb
[root@DELL-RnD-India oops]# gdb oops.ko
GNU gdb (GDB) Fedora (7.1-18.fc13)
Reading symbols from /code/oops/oops.ko...done.
```

2. add the symbol file to the debugger

```gdb
(gdb) add-symbol-file oops.o 0xffffffffa03e1000
add symbol table from file "oops.o" at
    .text_addr = 0xffffffffa03e1000
(y or n) y
Reading symbols from /code/oops/oops.o...done.
```
Here, the second argument is the address of the text section of the module. You can obtain it by execute `cat /sys/module/oops/sections/.init.text`.

3. disassemble the defective function

```gdb
(gdb) disassemble my_oops_init
Dump of assembler code for function my_oops_init:
   0x0000000000000038 <+0>:    push   %rbp
   0x0000000000000039 <+1>:    mov    $0x0,%rdi
   0x0000000000000040 <+8>:    xor    %eax,%eax
   0x0000000000000042 <+10>:    mov    %rsp,%rbp
   0x0000000000000045 <+13>:    callq  0x4a <my_oops_init+18>
   0x000000000000004a <+18>:    movl   $0x0,0x0
   0x0000000000000055 <+29>:    xor    %eax,%eax
   0x0000000000000057 <+31>:    leaveq
   0x0000000000000058 <+32>:    retq
End of assembler dump.
```
0x0000000000000038 is the start offset of the `.text` of `my_oops_init` function. And the offset of abnormal instruction (RIP) can be found in oops infomation: 
```
RIP  [<ffffffffa03e1012>] my_oops_init+0x12/0x21 [oops]
```

4. calculate the actual offset of RIP

0x0000000000000038 + 0x12 = 0x000000000000004a

5. find the line in the source file

```gdb
(gdb) list *0x000000000000004a
0x4a is in my_oops_init (/code/oops/oops.c:6).
1    #include <linux/kernel.h>
2    #include <linux/module.h>
3    #include <linux/init.h>
4
5    static void create_oops() {
6        *(int *)0 = 0;
7    }
```
The line 6 in oops.c is actually the error line. That's it!

```Bash
cat /proc/modules
cat /sys/module/XXX/section/.init.text
```
