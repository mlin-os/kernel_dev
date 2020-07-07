### Reference

1. https://elinux.org/images/2/2d/ELC2010-gc-sections_Denys_Vlasenko.pdf

### Basics

1. What's linking?

> Linking is the process of collecting and combining various pieces of code and data into a single file.
> Linking can be performed at _compile time_, when the source code is translated into machine code; at _load time_, when the program is loaded into memory and executed by the _loader_; and even at _run time_, by application programs.
>
> To build the executable, the linker must perform two main tasks:
> _Step 1. Symbol resolution._ The purpose of symbol resolution is to associate each symbol _reference_ with exactly one symbol _definition_.
> _step 2. Relocation._ Compilers and assemblers generate code and data sections that start at address 0. The linker _relates_ these sections by associating a memory location with each symbol definition, and then modifying all of the references to those symbols so that they point to this memory location.
> 
> Object files come in three forms:
> * _Relocatable object file._ Contains binary code and data in a form that can be combined with other relocatable object files at compile time to crate an executable object file.
> * _Executable object file._ Contains binary code and data in a form that can be copied directly into memory and executed.
> * _Shared object file._ - A special type of relocatable object file that can be loaded into memory and linked dynamically, at either load time or run time.


### Examples

1. How to distinguish between static local variables with the same name?

```C
/* fun.c */
int f()
{
    static int x = 0;
    return x;
}

int g()
{
    static int x = 1;
    return x;
}
```
```Bash
[l21901@cr-soft-021 static]$ readelf -s fun.o  

Symbol table '.symtab' contains 12 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
     0: 0000000000000000     0 NOTYPE  LOCAL  DEFAULT  UND 
     1: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS fun.c
     2: 0000000000000000     0 SECTION LOCAL  DEFAULT    1 
     3: 0000000000000000     0 SECTION LOCAL  DEFAULT    3 
     4: 0000000000000000     0 SECTION LOCAL  DEFAULT    4 
     5: 0000000000000000     4 OBJECT  LOCAL  DEFAULT    4 x.1723
     6: 0000000000000000     4 OBJECT  LOCAL  DEFAULT    3 x.1726
     7: 0000000000000000     0 SECTION LOCAL  DEFAULT    6 
     8: 0000000000000000     0 SECTION LOCAL  DEFAULT    7 
     9: 0000000000000000     0 SECTION LOCAL  DEFAULT    5 
    10: 0000000000000000    12 FUNC    GLOBAL DEFAULT    1 f
    11: 000000000000000c    12 FUNC    GLOBAL DEFAULT    1 g
```

Ndx is an integer index which identifies section. Ndx=1 denotes the .text section, Ndx=3 denots the .data section.

2. What's the purpose of `-ffunction-sections`, `-fdata-sections` and `gc-sections` options?

```C
/* fun.c */
#include "fun.h"

int fun1(int a, int b)
{
        return a + b + 1;
}

int fun2(int a, int b)
{
        return a + b + 2;
}

/* main.c */
#include <stdio.h>
#include "fun.h"

int main()
{
        int a = 1;
        int b = 2;
        int c;

        c = fun1(a, b); 
        printf("The result is %d\n", c); 
        return 0;
}
```

`gcc -`

```C
mu@ustc:~/test/c/link$ objdump -d fun1.o 

fun1.o:     file format elf64-x86-64


Disassembly of section .text:

0000000000000000 <fun1>:
   0:	55                   	push   %rbp
   1:	48 89 e5             	mov    %rsp,%rbp
   4:	89 7d fc             	mov    %edi,-0x4(%rbp)
   7:	89 75 f8             	mov    %esi,-0x8(%rbp)
   a:	8b 55 fc             	mov    -0x4(%rbp),%edx
   d:	8b 45 f8             	mov    -0x8(%rbp),%eax
  10:	01 d0                	add    %edx,%eax
  12:	83 c0 01             	add    $0x1,%eax
  15:	5d                   	pop    %rbp
  16:	c3                   	retq   

0000000000000017 <fun2>:
  17:	55                   	push   %rbp
  18:	48 89 e5             	mov    %rsp,%rbp
  1b:	89 7d fc             	mov    %edi,-0x4(%rbp)
  1e:	89 75 f8             	mov    %esi,-0x8(%rbp)
  21:	8b 55 fc             	mov    -0x4(%rbp),%edx
  24:	8b 45 f8             	mov    -0x8(%rbp),%eax
  27:	01 d0                	add    %edx,%eax
  29:	83 c0 02             	add    $0x2,%eax
  2c:	5d                   	pop    %rbp
  2d:	c3                   	retq   
mu@ustc:~/test/c/link$ objdump -d fun2.o

fun2.o:     file format elf64-x86-64


Disassembly of section .text.fun1:

0000000000000000 <fun1>:
   0:	55                   	push   %rbp
   1:	48 89 e5             	mov    %rsp,%rbp
   4:	89 7d fc             	mov    %edi,-0x4(%rbp)
   7:	89 75 f8             	mov    %esi,-0x8(%rbp)
   a:	8b 55 fc             	mov    -0x4(%rbp),%edx
   d:	8b 45 f8             	mov    -0x8(%rbp),%eax
  10:	01 d0                	add    %edx,%eax
  12:	83 c0 01             	add    $0x1,%eax
  15:	5d                   	pop    %rbp
  16:	c3                   	retq   

Disassembly of section .text.fun2:

0000000000000000 <fun2>:
   0:	55                   	push   %rbp
   1:	48 89 e5             	mov    %rsp,%rbp
   4:	89 7d fc             	mov    %edi,-0x4(%rbp)
   7:	89 75 f8             	mov    %esi,-0x8(%rbp)
   a:	8b 55 fc             	mov    -0x4(%rbp),%edx
   d:	8b 45 f8             	mov    -0x8(%rbp),%eax
  10:	01 d0                	add    %edx,%eax
  12:	83 c0 02             	add    $0x2,%eax
  15:	5d                   	pop    %rbp
  16:	c3                   	retq   
```