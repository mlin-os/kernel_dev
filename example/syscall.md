#### [How to get the location of C standard libary](https://stackoverflow.com/questions/5925678/location-of-c-standard-library?answertab=votes#tab-top)?
```Bash
mu@ustc:~$ gcc --print-file-name=libc.a
/usr/lib/gcc/x86_64-linux-gnu/9/../../../x86_64-linux-gnu/libc.a
```

#### How to get the assembly code from a static lib?
```Bash
objdump -d /usr/lib/gcc/x86_64-linux-gnu/9/../../../x86_64-linux-gnu/libc.a

# e.g. Here is the assembly code got from libc.a
getpid.o:     file format elf64-x86-64


Disassembly of section .text:

0000000000000000 <__getpid>:
   0:   b8 27 00 00 00          mov    $0x27,%eax
   5:   0f 05                   syscall
   7:   c3                      retq

```
