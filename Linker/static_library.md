# 静态链接

```c
/* a.c */
extern int shared;

int main()
{
    int a = 100;
    swap(&a, &shared);
}

/* b.c */
int shared = 1;

void swap(int *a, int *b)
{
    *a ^= *b ^= *a ^= *b;
}
```

在熟悉了ELF文件格式之后，下一个问题是：当我们有多个目标文件时，如何将它们链接起来形成一个可执行文件。本章节将以上述两个C语言程序为例展开分析”静态链接”的过程。

## 空间与地址分配

对于链接器来说，整个链接过程中，它就是将几个输入目标文件加工后合并成一个输出文件。

### 按序叠加

最简单的方法就是将输入的目标文件按照次序叠加起来，但是在有很多输入文件的情况下，输出文件将会有很多零散的段，这种做法非常浪费空间，因为每个段都会有一定的地址和空间对其要求，会造成内存空间大量的内部碎片。

### 相似段合并

更实际的方法是将相同性质的段合并到一起：

- 空间与地址分配：扫描所有的输入目标文件，并且获得它们的各个段的产犊、属性和位置，并且将输入目标文件中的符号表中所有的符号定义和符号收集起来，统一放到一个**全局符号表**。在这一步中链接器能够获取所有输入目标文件的段长度，并且将他们合并，计算出输出文件中各个段合并后的长度与位置，并建立映射关系。
- 符号解析与重定位

```shell
# 在编译a.o和b.o时需要使用-fno-stack-protector选项，否则会导致链接失败
ts@ts-OptiPlex-7070:~/Downloads/test$ objdump -h a.o

a.o:     file format elf64-x86-64

Sections:
Idx Name          Size      VMA               LMA               File off  Algn
  0 .text         0000002c  0000000000000000  0000000000000000  00000040  2**0
                  CONTENTS, ALLOC, LOAD, RELOC, READONLY, CODE
  1 .data         00000000  0000000000000000  0000000000000000  0000006c  2**0
                  CONTENTS, ALLOC, LOAD, DATA
  2 .bss          00000000  0000000000000000  0000000000000000  0000006c  2**0
                  ALLOC
  3 .comment      00000036  0000000000000000  0000000000000000  0000006c  2**0
                  CONTENTS, READONLY
  4 .note.GNU-stack 00000000  0000000000000000  0000000000000000  000000a2  2**0
                  CONTENTS, READONLY
  5 .eh_frame     00000038  0000000000000000  0000000000000000  000000a8  2**3
                  CONTENTS, ALLOC, LOAD, RELOC, READONLY, DATA
ts@ts-OptiPlex-7070:~/Downloads/test$ objdump -h b.o

b.o:     file format elf64-x86-64

Sections:
Idx Name          Size      VMA               LMA               File off  Algn
  0 .text         0000004b  0000000000000000  0000000000000000  00000040  2**0
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
  1 .data         00000004  0000000000000000  0000000000000000  0000008c  2**2
                  CONTENTS, ALLOC, LOAD, DATA
  2 .bss          00000000  0000000000000000  0000000000000000  00000090  2**0
                  ALLOC
  3 .comment      00000036  0000000000000000  0000000000000000  00000090  2**0
                  CONTENTS, READONLY
  4 .note.GNU-stack 00000000  0000000000000000  0000000000000000  000000c6  2**0
                  CONTENTS, READONLY
  5 .eh_frame     00000038  0000000000000000  0000000000000000  000000c8  2**3
                  CONTENTS, ALLOC, LOAD, RELOC, READONLY, DATA
ts@ts-OptiPlex-7070:~/Downloads/test$ objdump -h ab

ab:     file format elf64-x86-64

Sections:
Idx Name          Size      VMA               LMA               File off  Algn
  0 .text         00000077  00000000004000e8  00000000004000e8  000000e8  2**0
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
  1 .eh_frame     00000058  0000000000400160  0000000000400160  00000160  2**3
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  2 .data         00000004  00000000006001b8  00000000006001b8  000001b8  2**2
                  CONTENTS, ALLOC, LOAD, DATA
  3 .comment      00000035  0000000000000000  0000000000000000  000001bc  2**0
                  CONTENTS, READONLY

```

在链接之前，目标文件中的所有段的VMA都是0，因为虚拟空间还没有被分配，所以他们默认都为0。等到链接之后，可执行文件中的各个段都被分配到了相应的虚拟地址。

### 符号地址的确定

各个“段”的虚拟地址在上述第一步过程中就以确定，完成这一步之后链接器开始计算各个符号的虚拟地址——各个符号相对段的偏移量。

## 符号解析与重定位

### 重定位

通过objdump -d可以看到a.o的代码段反汇编结果:

```shell
# objdump -d a.o
ts@ts-OptiPlex-7070:~/Downloads/test$ objdump -d a.o

a.o:     file format elf64-x86-64


Disassembly of section .text:

0000000000000000 <main>:
   0:	55                   	push   %rbp
   1:	48 89 e5             	mov    %rsp,%rbp
   4:	48 83 ec 10          	sub    $0x10,%rsp
   8:	c7 45 fc 64 00 00 00 	movl   $0x64,-0x4(%rbp)
   f:	48 8d 45 fc          	lea    -0x4(%rbp),%rax
  13:	be 00 00 00 00       	mov    $0x0,%esi
  18:	48 89 c7             	mov    %rax,%rdi
  1b:	b8 00 00 00 00       	mov    $0x0,%eax
  20:	e8 00 00 00 00       	callq  25 <main+0x25>
  25:	b8 00 00 00 00       	mov    $0x0,%eax
  2a:	c9                   	leaveq 
  2b:	c3                   	retq 
```

让我们关注在a.c中引用了shared变量和swap函数，

- `be 00 00 00 00`：mov指令总共5个字节，它的作用是将shared变量的地址赋值到ESI寄存器中去，但在编译阶段编译器并不知道shared和swap的地址，这时候shared变量的地址被假定为0x00000000。
- `e8 00 00 00 00`：call指令总共5个字节，0xe8是操作码，后四个字节是被调用函数相对于调用指令的下一条指令的偏移量，在没有重定位之前，相对偏移位置被置为0x00000000。

```shell
# objdump -d ab
ts@ts-OptiPlex-7070:~/Downloads/test$ objdump -d ab

ab:     file format elf64-x86-64


Disassembly of section .text:

00000000004000e8 <main>:
  4000e8:	55                   	push   %rbp
  4000e9:	48 89 e5             	mov    %rsp,%rbp
  4000ec:	48 83 ec 10          	sub    $0x10,%rsp
  4000f0:	c7 45 fc 64 00 00 00 	movl   $0x64,-0x4(%rbp)
  4000f7:	48 8d 45 fc          	lea    -0x4(%rbp),%rax
  4000fb:	be b8 01 60 00       	mov    $0x6001b8,%esi
  400100:	48 89 c7             	mov    %rax,%rdi
  400103:	b8 00 00 00 00       	mov    $0x0,%eax
  400108:	e8 07 00 00 00       	callq  400114 <swap>
  40010d:	b8 00 00 00 00       	mov    $0x0,%eax
  400112:	c9                   	leaveq 
  400113:	c3                   	retq   

0000000000400114 <swap>:
  400114:	55                   	push   %rbp
  400115:	48 89 e5             	mov    %rsp,%rbp
  400118:	48 89 7d f8          	mov    %rdi,-0x8(%rbp)
  40011c:	48 89 75 f0          	mov    %rsi,-0x10(%rbp)
  400120:	48 8b 45 f8          	mov    -0x8(%rbp),%rax
  400124:	8b 10                	mov    (%rax),%edx
  400126:	48 8b 45 f0          	mov    -0x10(%rbp),%rax
  40012a:	8b 00                	mov    (%rax),%eax
  40012c:	31 c2                	xor    %eax,%edx
  40012e:	48 8b 45 f8          	mov    -0x8(%rbp),%rax
  400132:	89 10                	mov    %edx,(%rax)
  400134:	48 8b 45 f8          	mov    -0x8(%rbp),%rax
  400138:	8b 10                	mov    (%rax),%edx
  40013a:	48 8b 45 f0          	mov    -0x10(%rbp),%rax
  40013e:	8b 00                	mov    (%rax),%eax
  400140:	31 c2                	xor    %eax,%edx
  400142:	48 8b 45 f0          	mov    -0x10(%rbp),%rax
  400146:	89 10                	mov    %edx,(%rax)
  400148:	48 8b 45 f0          	mov    -0x10(%rbp),%rax
  40014c:	8b 10                	mov    (%rax),%edx
  40014e:	48 8b 45 f8          	mov    -0x8(%rbp),%rax
  400152:	8b 00                	mov    (%rax),%eax
  400154:	31 c2                	xor    %eax,%edx
  400156:	48 8b 45 f8          	mov    -0x8(%rbp),%rax
  40015a:	89 10                	mov    %edx,(%rax)
  40015c:	90                   	nop
  40015d:	5d                   	pop    %rbp
  40015e:	c3                   	retq  
```

可以看到shared和地址为0x006001b8，swap相对于下一条指令的偏址为0x00000007。

### 重定位表

重定位表专门用来保存与重定位相关的信息

```shell
objdump -r a.o
```

每个要被重定位的地方叫一个重定位入口，重定位入口的偏移表示该入口在要被重定位的段中的位置。`.rela.text`表示这个重定位表是代码段的重定位表，0x1c和0x27代表shared和swap在代码段中的地址。

### 符号解析

重定位的过程中，每个重定位的入口都是对一个符号的引用，当链接器需要对某个符号的引用进行重定位时，链接器会去查找由所有输入目标文件的符号表组成的全局符号表，找到相应的符号后再进行重定位。

```shell
# readelf -s a.o
ts@ts-OptiPlex-7070:~/Downloads/test$ readelf -s a.o

Symbol table '.symtab' contains 11 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
     0: 0000000000000000     0 NOTYPE  LOCAL  DEFAULT  UND 
     1: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS a.c
     2: 0000000000000000     0 SECTION LOCAL  DEFAULT    1 
     3: 0000000000000000     0 SECTION LOCAL  DEFAULT    3 
     4: 0000000000000000     0 SECTION LOCAL  DEFAULT    4 
     5: 0000000000000000     0 SECTION LOCAL  DEFAULT    6 
     6: 0000000000000000     0 SECTION LOCAL  DEFAULT    7 
     7: 0000000000000000     0 SECTION LOCAL  DEFAULT    5 
     8: 0000000000000000    44 FUNC    GLOBAL DEFAULT    1 main
     9: 0000000000000000     0 NOTYPE  GLOBAL DEFAULT  UND shared
    10: 0000000000000000     0 NOTYPE  GLOBAL DEFAULT  UND swap
```

可以看到a.o中存在两个UND类型的符号，即shared和swap，假如链接器扫描完所有的输入目标文件之后，依然无法在全局符号表中找到这些UND类型的符号，就会报符号未定义错误。

### 指令修正方式

| x86基本重定位类型 |      |                   |
| ----------------- | ---- | ----------------- |
| 宏定义            | 值   | 重定位修正方法    |
| R_386_32          | 1    | 绝对寻址修正S+A   |
| R_386_PC32        | 2    | 相对寻址修正S+A-P |

- A=保存在修正位置的值
- P=被修正的位置（相对于段开始的偏移量或者虚拟地址）
- S=符号的实际地址

这里以a.o和ab为例：

```shell
ts@ts-OptiPlex-7070:~/Downloads/test$ readelf -s ab

Symbol table '.symtab' contains 13 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
     0: 0000000000000000     0 NOTYPE  LOCAL  DEFAULT  UND 
     1: 00000000004000e8     0 SECTION LOCAL  DEFAULT    1 
     2: 0000000000400160     0 SECTION LOCAL  DEFAULT    2 
     3: 00000000006001b8     0 SECTION LOCAL  DEFAULT    3 
     4: 0000000000000000     0 SECTION LOCAL  DEFAULT    4 
     5: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS a.c
     6: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS b.c
     7: 0000000000400114    75 FUNC    GLOBAL DEFAULT    1 swap
     8: 00000000006001b8     4 OBJECT  GLOBAL DEFAULT    3 shared
     9: 00000000006001bc     0 NOTYPE  GLOBAL DEFAULT    3 __bss_start
    10: 00000000004000e8    44 FUNC    GLOBAL DEFAULT    1 main
    11: 00000000006001bc     0 NOTYPE  GLOBAL DEFAULT    3 _edata
    12: 00000000006001c0     0 NOTYPE  GLOBAL DEFAULT    3 _end
```

- shared绝对寻址修正：S是shared符号的实际地址，即ab中的值0x006001b8；A是被修正位置的值，即a.o中的值0x00000000。因此S+A=0x006001b8。
- swap相对寻址修正：S为swap函数的实际地址，即0x00400114。从上面readelf的输出可以看出A=0x00000000，P=0x00400109，按照上述计算方式得出的偏移修正值为S+A-P=0x0000000b，而不是0x00000007。这里和书中的计算其实有些出入，由于A不是书中的-4所以导致计算出的偏移值要比实际值大4个字节，这个应该和我的编译环境有关。**根据call指令的原理：call指令是一条近址相对位移调用指令，它后面跟的是call所调用的指令相对call指令的下一条指令的偏移量，在ab中即为call指令之后的mov指令，mov指令的地址为0x0040010d，因此swap的相对寻址修正值的计算方式可以简单理解为0x00400114-0x0040010d=0x00000007。**

绝对寻址修正和相对寻址修正的区别就是：绝对寻址修正后的地址为该符号的实际地址；相对寻址修正后的地址为符号距离被修正位置的地址差。

## COMMON块

由于弱符号机制允许同一个符号的定义存在于多个文件中，而链接器本身并不支持符号的类型，因此当目标文件中多个弱符号定义类型不一致时，链接器便采用COMMON块的机制。

在SimpleSection.c中，编译器将未初始化的全局变量定义作为弱符号处理：

```shell
# readelf -s SimpleSection.o | grep global_uninit_var
12: 0000000000000004     4 OBJECT  GLOBAL DEFAULT  COM global_uninit_var
```

可以看到他是一个全局的数据对象，他的类型为SHN_COMMON类型,占4个字节。假如另外的目标文件中也定义了global_ubinit_var变量且未初始化，这个变量的类型为double，占8个字节，那么按照COMMON类型的连接机制，最终输出文件中的global_uninit_var变量为占8字节的double类型。

注：直接导致需要COMMON机制的原因是编译器和链接器允许不同类型的弱符号存在，但最本质的原因是链接器不支持符号类型。

Q：为什么不直接把未初始化的全局变量也当作未初始化的局部变量一样处理，为它在BSS段分配空间？

弱符号（例如未初始化的全局变量）在编译阶段无法确定它最终所占空间的大小，因为有可能其他编译单元中该符号所占的空间比本编译单元该符号所占的空间要大，因此编译器无法为其在BSS段分配空间，直到链接器在连接过程中最终确定了该符号的大小，再在最终输出文件的BSS段为其分配空间。

> 关于多个文件中出现同一个变量的多个定义的原因，还有一种说法是由于早期C语言程序员粗心大意，经常忘记在声明变量时在前面加上"extern"关键字，使得编译器会在多个目标文件中产生同一个变量的定义。为了解决这个问题，编译器和链接器干脆就把未初始化的变量都当作COMMON类型来处理。

注：GCC的"-fno-common"允许我们把所有未初始化的全局变量不以COMMON快的形式处理或者使用：

```c
int global __attribute__ ((nocommon));
```

经上述处理的未初始化的全局变量相当于强符号。

## C++相关问题

### 重复代码消除

C++编译器在很多时候会产生重复的代码，比如模板、外部内联函数和虚函数表都有可能在不同的编译单元里生成相同的代码。例如模板，当一个模板在多个编译单元同时实例化成相同的类型的时候，必然会生成重复的代码。假如将这些重复的代码都保存下来：

- 空间浪费
- 地址容易出错
- 指令运行效率较低，同一份指令有多份副本，指令Cache的命中率会降低

一个比较有效的而做法就是将每个模板的示例代码都单独地存放在一个段里，每个段只包含一个模板实例，这样当多个编译单元含有同样的模板函实例时，链接器在最终链接时可以将它们并入最后的代码段。

这种方法基本上能够解决代码重复的问题，但假如不同的编译单元使用了不同的编译器版本或者编译优化选项，导致同一个函数编译出来的实际代码有所不同，那么链接器将随意选择其中任意一个副本作为链接的输出，并输出

**函数级别链接**：当链接器需要用到某个函数时，就将它合并到输出文件中，对于那些没有用的函数则将他们抛弃。这么做能够减小输出文件的长度，减少空间浪费，但是会减慢编译和连接的过程，以及由于每个函数都需要存在单独的段中，目标文件会变大，重定位会变得复杂。GCC提供"-ffunction-sections"和"-fdata-sections"编译选项将每个函数或变量分别保持到独立的段中等待链接器处理。

### 全局构造与析构

ELF文件中还定义了两个段：

- .init：该段里面保存的时可执行指令，它构成了进程的初始化代码。因此，当一个程序开始运行时，在main函数被调用之前，Glibc的初始化部分安排执行这个段中的代码。
- .fini：该段保存着进程终止代码指令。因此，但一个程序的main函数正常退出时，Glibc会安排执行这个段中的代码。

C++的全局变量的构造和析构函数就保存在这两个段中。

### C++与ABI

一般的，将符号修饰标准、变量内存布局、函数调用方式等这些跟可执行代码二进制兼容性相关的内容成为ABI。

## 静态库链接

在编译时，一个源文件将被编译成一个目标文件，于是通常人们使用”ar“压缩程序将这些目标文件压缩到一起，并且对其进行行编号和索引，以便于查找和检索，因此一个静态库可以简单地看成一组目标文件的集合。

接下来以Hello，world！程序为例，演示printf()函数静态库的链接过程：

```shell
# objdump -t libc.a | grep printf

# ar -x libc.a

# ld hello.o printf.o

# gcc -static --verbose -fno-builtin hello.c

```

