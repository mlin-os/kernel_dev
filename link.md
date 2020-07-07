### Reference

1. https://elinux.org/images/2/2d/ELC2010-gc-sections_Denys_Vlasenko.pdf


### Basics Concepts
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

### ELF file format
![Typical ELF relocatable object file](./figures/elf_relocatable_obj_file.png)

### Symbols and Symbol Table

#### Three types of linker symbols
* _Global symbols_ that are defined by module m and that can be referenced by other modules, including _non-static_ C functions and global variables.
* _Global symbols_ that are referenced by module m but defined by some other module. Such symbols are called _externals_.
* _Local symbols_ that are defined and referenced exclusively by module m. There correspond to static C functions and global variables that are defined with the static attribute.

Here the local linker symbols are not the same as local program variables.

#### COMMON vs. .bss section
Modern versions of GCC assign symbols in relocatable object files to COMMON and .bss using following convention:
* _COMMON_ for uninitialized global variables;
* _.bss_ for uninitialized static variables, and global or static variables that are initialized to zero;

This convention is due to the fact that in some cases the linker allows multiple modules to define global symbols with the same name. When the compiler is translating some module and encounters a weak global symbol, say, x, it does not know if other modules also define x, and if so, it cannot predict which of the multiple instances of x the linker might choose. So the compiler defers the decision to the linker by assigning x to COMMON.

If x is initialized to zero, then it is a strong symbol, so the compiler can confidently assign it to .bss. Similarly, static symbols are unique by construction, so the compiler can confidently assign them to either .data or .bss.

You can invoke the linker with a flag such as the GCC -fno-common flag to trigger an error if it encounters multiply-defined global symbols.

### Symbol Resolution
At compile time, the compiler exports each global symbol to the assembler as either _strong_ or _weak_, and the assembler encodes this information implicitly in the symbol table of the relocatable object file. __Functions and initialized global variables get strong symbols, Uninitialized global variables get weak symbols.__

Linux linkers use the following rules for dealing with duplicate symbol names:
Rule 1. Multiple strong symbols with the same name are not allowed.
Rule 2. Given a strong symbol and multiple weak symbols with the same name, choose the strong symbol.
Rule 3. Given multiple weak symbols with the same name, choose any of the weak symbols.

### Static Libraries

#### What is a static library?
A static libraries are stored on disk in a particular file format known as an _archive_. An archive is a collection of concatenated relocatable object files, with a header that describes the size and location of each member object file.

#### What's the process of linking a static libraries?

> During the symbol resolution phase, the linker scans the relocatable object files and archives left to right in the same sequential order that they appear on the compiler driver’s command line. (The driver automatically translates any .c files on the command line into .o files.) During this scan, the linker maintains a set _E_ of relocatable object files that will be merged to form the executable, a set _U_ of unresolved symbols (i.e., symbols referred to but not yet defined), and a set _D_ of symbols that have been defined in previous input files. Initially, _E_, _U_, and _D_ are empty.
> * For each input file _f_ on the command line, the linker determines if _f_ is an object file or an archive. If _f_ is an object file, the linker adds _f_ to _E_, updates _U_ and _D_ to reflect the symbol definitions and references in _f_ , and proceeds to the next input file.
> * If _f_ is an archive, the linker attempts to match the unresolved symbols in _U_ against the symbols defined by the members of the archive. If some archive member _m_ defines a symbol that resolves a reference in _U_, then _m_ is added to _E_, and the linker updates _U_ and _D_ to reflect the symbol definitions and references in _m_. This process iterates over the member object files in the archive until a fixed point is reached where _U_ and _D_ no longer change. At this point, any member object files not contained in _E_ are simply discarded and the linker proceeds to the next input file.
> If _U_ is nonempty when the linker finishes scanning the input files on the command line, it prints an error and terminates. Otherwise, it merges and relocates the object files in _E_ to build the output executable file.

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

Ndx is an integer index which identifies section. Ndx=1 denotes the .text section, Ndx=3 denotes the .data section.

2. COMMON vs. .bss vs. .data
```C
/* main.c */
#include <stdio.h>

int x;
int y = 10;
int z = 0;

int main()
{
    printf("x = %d\n", x);
    return 0;
}

```

```Bash
[l21901@cr-soft-021 extern]$ readelf -s main.o

Symbol table '.symtab' contains 14 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
     0: 0000000000000000     0 NOTYPE  LOCAL  DEFAULT  UND 
     1: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS main.c
     2: 0000000000000000     0 SECTION LOCAL  DEFAULT    1 
     3: 0000000000000000     0 SECTION LOCAL  DEFAULT    3 
     4: 0000000000000000     0 SECTION LOCAL  DEFAULT    4 
     5: 0000000000000000     0 SECTION LOCAL  DEFAULT    5 
     6: 0000000000000000     0 SECTION LOCAL  DEFAULT    7 
     7: 0000000000000000     0 SECTION LOCAL  DEFAULT    8 
     8: 0000000000000000     0 SECTION LOCAL  DEFAULT    6 
     9: 0000000000000004     4 OBJECT  GLOBAL DEFAULT  COM x
    10: 0000000000000000     4 OBJECT  GLOBAL DEFAULT    3 y
    11: 0000000000000000     4 OBJECT  GLOBAL DEFAULT    4 z
    12: 0000000000000000    34 FUNC    GLOBAL DEFAULT    1 main
    13: 0000000000000000     0 NOTYPE  GLOBAL DEFAULT  UND printf
````

3. "weak" vs. "strong"
```C
/* main.c */
#include <stdio.h>

int x;
int x;

int main()
{
    printf("x = %d\n", x);
    return 0;
}
/* fun.c */
int x = 10;
```

```Bash
[l21901@cr-soft-021 extern]$ readelf -s main.o

Symbol table '.symtab' contains 12 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
     0: 0000000000000000     0 NOTYPE  LOCAL  DEFAULT  UND 
     1: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS main.c
     2: 0000000000000000     0 SECTION LOCAL  DEFAULT    1 
     3: 0000000000000000     0 SECTION LOCAL  DEFAULT    3 
     4: 0000000000000000     0 SECTION LOCAL  DEFAULT    4 
     5: 0000000000000000     0 SECTION LOCAL  DEFAULT    5 
     6: 0000000000000000     0 SECTION LOCAL  DEFAULT    7 
     7: 0000000000000000     0 SECTION LOCAL  DEFAULT    8 
     8: 0000000000000000     0 SECTION LOCAL  DEFAULT    6 
     9: 0000000000000004     4 OBJECT  GLOBAL DEFAULT  COM x
    10: 0000000000000000    34 FUNC    GLOBAL DEFAULT    1 main
    11: 0000000000000000     0 NOTYPE  GLOBAL DEFAULT  UND printf
```
Here x is in COMMON section. And only one exists.

4. What does `extern` do?
```C
/* main.c */
#include <stdio.h>

extern int x;

int main()
{
    printf("x = %d\n", x);
    return 0;
}
```

```Bash
[l21901@cr-soft-021 extern]$ readelf -s main.o

Symbol table '.symtab' contains 12 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
     0: 0000000000000000     0 NOTYPE  LOCAL  DEFAULT  UND 
     1: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS main.c
     2: 0000000000000000     0 SECTION LOCAL  DEFAULT    1 
     3: 0000000000000000     0 SECTION LOCAL  DEFAULT    3 
     4: 0000000000000000     0 SECTION LOCAL  DEFAULT    4 
     5: 0000000000000000     0 SECTION LOCAL  DEFAULT    5 
     6: 0000000000000000     0 SECTION LOCAL  DEFAULT    7 
     7: 0000000000000000     0 SECTION LOCAL  DEFAULT    8 
     8: 0000000000000000     0 SECTION LOCAL  DEFAULT    6 
     9: 0000000000000000    34 FUNC    GLOBAL DEFAULT    1 main
    10: 0000000000000000     0 NOTYPE  GLOBAL DEFAULT  UND x
    11: 0000000000000000     0 NOTYPE  GLOBAL DEFAULT  UND printf
```
Here x is undefined and type changes to NOTYPE.

5. [What's the purpose of `__attribute__((weak))`](https://gcc.gnu.org/onlinedocs/gcc-4.7.2/gcc/Function-Attributes.html)？
```C
/* weak.c */
__attribute__((weak)) int add(int x, int y)
{
    return x + y;
}
```

```Bash
[l21901@cr-soft-021 extern]$ readelf -s weak.o

Symbol table '.symtab' contains 9 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
     0: 0000000000000000     0 NOTYPE  LOCAL  DEFAULT  UND 
     1: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS weak.c
     2: 0000000000000000     0 SECTION LOCAL  DEFAULT    1 
     3: 0000000000000000     0 SECTION LOCAL  DEFAULT    2 
     4: 0000000000000000     0 SECTION LOCAL  DEFAULT    3 
     5: 0000000000000000     0 SECTION LOCAL  DEFAULT    5 
     6: 0000000000000000     0 SECTION LOCAL  DEFAULT    6 
     7: 0000000000000000     0 SECTION LOCAL  DEFAULT    4 
     8: 0000000000000000    20 FUNC    WEAK   DEFAULT    1 add
```

6. The influence of the location of static libraries on command line?
```C
/* addvec.c */
int addcnt = 0;

void addvec(int *x, int *y, int *z, int n)
{
    int i;

    addcnt++;

    for (i = 0; i < n; i++)
        z[i] = x[i] + y[i];
}

/* multvec.c */
int multcnt = 0;

void multvec(int *x, int *y, int *z, int n)
{
    int i;

    multcnt++;

    for (i = 0; i < n; i++)
        z[i] = x[i] * y[i];
}
```

```Bash
[l21901@cr-soft-021 ex1]$ gcc -c addvec.c multvec.c
[l21901@cr-soft-021 ex1]$ ar rcs libvector.a addvec.o multvec.o
# it has problem
[l21901@cr-soft-021 ex1]$ gcc -o test libvector.a main.c                
/tmp/ccIlXC2A.o: In function `main':
main.c:(.text+0x19): undefined reference to `addvec'
collect2: error: ld returned 1 exit status
# it's ok
[l21901@cr-soft-021 ex1]$ gcc -o test main.c ./libvector.a
```

7. What's the purpose of `-ffunction-sections`, `-fdata-sections` and `--gc-sections` options?

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


```Bash
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

```Bash
[l21901@cr-soft-021 gc]$ gcc -Wl,--gc-sections -Wl,--print-gc-sections -o test main.o fun.o   
/usr/bin/ld: Removing unused section '.rodata.cst4' in file '/usr/lib/gcc/x86_64-redhat-linux/4.8.5/../../../../lib64/crt1.o'
/usr/bin/ld: Removing unused section '.data' in file '/usr/lib/gcc/x86_64-redhat-linux/4.8.5/../../../../lib64/crt1.o'
/usr/bin/ld: Removing unused section '.rodata' in file '/usr/lib/gcc/x86_64-redhat-linux/4.8.5/crtbegin.o'
/usr/bin/ld: Removing unused section '.text.fun2' in file 'fun.o'
```