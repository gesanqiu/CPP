# 目标文件里有什么

编译器编译源代码后生成的文件叫做目标文件，从结构上讲，它是已经编译后的可执行文件格式，只是还没有经过链接的过程，其中可能有些符号或有些地址还没有被调整。其实它本身是按照可执行文件格式存储的，只是跟真正的可执行文件在结构上稍有不同。

## 目标文件的格式

| ELF文件类型                        | 说明                                                         | 实例                          |
| ---------------------------------- | ------------------------------------------------------------ | ----------------------------- |
| 可重定位文件（Relocatable File）   | 这类文件包含了代码和数据，可以被用来链接成可执行文件或共享目标文件，静态链接库也可以归为这一类。 | Linux的.o<br />Windows的.obj  |
| 可执行文件（Executable File）      | 这类文件包含了可以直接执行的程序，它的代表就是ELF可执行文件，它们一般都没有拓展名。 | Linux的ELF<br />Windows的.exe |
| 共享目标文件（Shared Object File） | 这种文件包含了代码和数据，可以在以下两种情况下使用。一种是链接器可以使用这种文件跟其他的可重定位文件和共享目标文件链接，产生新的目标文件。第二种是动态链接器可以将几个这种共享目标文件与可执行文件结合，作为进程映像的一部分来运行。 | Linux的.so<br />Windows的.dll |
| 核心转储文件（Core Dump File）     | 当进程意外终止时，系统可以将该进程的地址空间的内容及终止时的一些其他信息转出到核心转储文件。 | Linux下的core dump            |

##  目标文件是什么样的

```c
// SimpleSection.c
int printf(const char* format, ...);

int global_init_var = 84;
int global_uninit_var;

void func1(int i)
{
    printf("%d\n", i);
}

int main(void)
{
    static int static_var = 85;
    static int static_var2;

    int a = 1;
    int b;

    func(static_var + static_var2 + 2 + b);

    return a;
}
```

使用gcc编译上述代码，通过readelf可以查看可执行文件的内部情况：

```shell
ts@ts-OptiPlex-7070:~/Downloads/test$ gcc SimpleSection.c -o SimpleSection
ts@ts-OptiPlex-7070:~/Downloads/test$ readelf -S SimpleSection
There are 31 section headers, starting at offset 0x1ab0:

Section Headers:
  [Nr] Name              Type             Address           Offset
       Size              EntSize          Flags  Link  Info  Align
  [ 0]                   NULL             0000000000000000  00000000
       0000000000000000  0000000000000000           0     0     0
  [ 1] .interp           PROGBITS         0000000000400238  00000238
       000000000000001c  0000000000000000   A       0     0     1
  [ 2] .note.ABI-tag     NOTE             0000000000400254  00000254
       0000000000000020  0000000000000000   A       0     0     4
  [ 3] .note.gnu.build-i NOTE             0000000000400274  00000274
       0000000000000024  0000000000000000   A       0     0     4
  [ 4] .gnu.hash         GNU_HASH         0000000000400298  00000298
       000000000000001c  0000000000000000   A       5     0     8
  [ 5] .dynsym           DYNSYM           00000000004002b8  000002b8
       0000000000000060  0000000000000018   A       6     1     8
  [ 6] .dynstr           STRTAB           0000000000400318  00000318
       000000000000003f  0000000000000000   A       0     0     1
  [ 7] .gnu.version      VERSYM           0000000000400358  00000358
       0000000000000008  0000000000000002   A       5     0     2
  [ 8] .gnu.version_r    VERNEED          0000000000400360  00000360
       0000000000000020  0000000000000000   A       6     1     8
  [ 9] .rela.dyn         RELA             0000000000400380  00000380
       0000000000000018  0000000000000018   A       5     0     8
  [10] .rela.plt         RELA             0000000000400398  00000398
       0000000000000030  0000000000000018  AI       5    24     8
  [11] .init             PROGBITS         00000000004003c8  000003c8
       000000000000001a  0000000000000000  AX       0     0     4
  [12] .plt              PROGBITS         00000000004003f0  000003f0
       0000000000000030  0000000000000010  AX       0     0     16
  [13] .plt.got          PROGBITS         0000000000400420  00000420
       0000000000000008  0000000000000000  AX       0     0     8
  [14] .text             PROGBITS         0000000000400430  00000430
       00000000000001c2  0000000000000000  AX       0     0     16
  [15] .fini             PROGBITS         00000000004005f4  000005f4
       0000000000000009  0000000000000000  AX       0     0     4
  [16] .rodata           PROGBITS         0000000000400600  00000600
       0000000000000008  0000000000000000   A       0     0     4
  [17] .eh_frame_hdr     PROGBITS         0000000000400608  00000608
       000000000000003c  0000000000000000   A       0     0     4
  [18] .eh_frame         PROGBITS         0000000000400648  00000648
       0000000000000114  0000000000000000   A       0     0     8
  [19] .init_array       INIT_ARRAY       0000000000600e10  00000e10
       0000000000000008  0000000000000000  WA       0     0     8
  [20] .fini_array       FINI_ARRAY       0000000000600e18  00000e18
       0000000000000008  0000000000000000  WA       0     0     8
  [21] .jcr              PROGBITS         0000000000600e20  00000e20
       0000000000000008  0000000000000000  WA       0     0     8
  [22] .dynamic          DYNAMIC          0000000000600e28  00000e28
       00000000000001d0  0000000000000010  WA       6     0     8
  [23] .got              PROGBITS         0000000000600ff8  00000ff8
       0000000000000008  0000000000000008  WA       0     0     8
  [24] .got.plt          PROGBITS         0000000000601000  00001000
       0000000000000028  0000000000000008  WA       0     0     8
  [25] .data             PROGBITS         0000000000601028  00001028
       0000000000000018  0000000000000000  WA       0     0     8
  [26] .bss              NOBITS           0000000000601040  00001040
       0000000000000010  0000000000000000  WA       0     0     4
  [27] .comment          PROGBITS         0000000000000000  00001040
       0000000000000035  0000000000000001  MS       0     0     1
  [28] .shstrtab         STRTAB           0000000000000000  0000199f
       000000000000010c  0000000000000000           0     0     1
  [29] .symtab           SYMTAB           0000000000000000  00001078
       00000000000006c0  0000000000000018          30    49     8
  [30] .strtab           STRTAB           0000000000000000  00001738
       0000000000000267  0000000000000000           0     0     1
Key to Flags:
  W (write), A (alloc), X (execute), M (merge), S (strings), l (large)
  I (info), L (link order), G (group), T (TLS), E (exclude), x (unknown)
  O (extra OS processing required) o (OS specific), p (processor specific)
```

从上面的输出信息可以看到这个可执行文件有31个段，但通常我们只需要关注其中的部分段。一般C语言的编译后执行语句都编译成机器代码保存在.text段；已初始化的全局变量和局部静态变量都保存在.data段；未初始化的全局变量和局部静态变量一般放在.bss段中。

> 未初始化的全局变量和局部静态变量默认值都为0，本来它们也可以被放在.data段的，但是因为它们都是0，所以为它们在.data段分配空间并存放数据0是没有必要的。程序运行的时候它们的确是要占内存空间的，并且可执行文件必须记录所有未初始化的全局变量和局部静态变量的大小总和，记为.bss段，即.bss段只是为未初始化的全局变量和局部静态变量预留位置而已，.bss段并没有内容，在文件中不占据空间。

注：事实上初始化值为0的全局变量和局部静态变量也保存在.bss段中，这是因为由于初始化为0的全局变量和局部静态变量恰好也符合了.bss的设计，于是编译器就把它们当做未初始化的全局变量和局部静态变量一同处理了。

总体来说，程序源代码被编译以后主要分为两种段：程序指令和程序数据。代码段属于程序指令，而数据段和.bss段属于程序数据。这种分离式的设计是出于以下几种原因：

- 通常来说数据段和代码段的读写权限不同，代码段通常是只读的，这样可以防止程序的指令被修改。
- CPU缓存被设计为数据缓存和指令缓存分离，有利于提高程序的局部性。
- 当系统中运行着多个程序的副本时，指令是可以共用的，而数据是私有的，分离式的设计能够节约指令部分的空间。

## 挖掘SimpleSection.o

```shell
ts@ts-OptiPlex-7070:~/Downloads/test$ readelf -h SimpleSection.o
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00 
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              REL (Relocatable file)
  Machine:                           Advanced Micro Devices X86-64
  Version:                           0x1
  Entry point address:               0x0
  Start of program headers:          0 (bytes into file)
  Start of section headers:          1064 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           0 (bytes)
  Number of program headers:         0
  Size of section headers:           64 (bytes)
  Number of section headers:         13
  Section header string table index: 10
ts@ts-OptiPlex-7070:~/Downloads/test$ readelf -S SimpleSection.o
There are 13 section headers, starting at offset 0x428:

Section Headers:
  [Nr] Name              Type             Address           Offset
       Size              EntSize          Flags  Link  Info  Align
  [ 0]                   NULL             0000000000000000  00000000
       0000000000000000  0000000000000000           0     0     0
  [ 1] .text             PROGBITS         0000000000000000  00000040
       0000000000000053  0000000000000000  AX       0     0     1
  [ 2] .rela.text        RELA             0000000000000000  00000318
       0000000000000078  0000000000000018   I      11     1     8
  [ 3] .data             PROGBITS         0000000000000000  00000094
       0000000000000008  0000000000000000  WA       0     0     4
  [ 4] .bss              NOBITS           0000000000000000  0000009c
       0000000000000004  0000000000000000  WA       0     0     4
  [ 5] .rodata           PROGBITS         0000000000000000  0000009c
       0000000000000004  0000000000000000   A       0     0     1
  [ 6] .comment          PROGBITS         0000000000000000  000000a0
       0000000000000036  0000000000000001  MS       0     0     1
  [ 7] .note.GNU-stack   PROGBITS         0000000000000000  000000d6
       0000000000000000  0000000000000000           0     0     1
  [ 8] .eh_frame         PROGBITS         0000000000000000  000000d8
       0000000000000058  0000000000000000   A       0     0     8
  [ 9] .rela.eh_frame    RELA             0000000000000000  00000390
       0000000000000030  0000000000000018   I      11     8     8
  [10] .shstrtab         STRTAB           0000000000000000  000003c0
       0000000000000061  0000000000000000           0     0     1
  [11] .symtab           SYMTAB           0000000000000000  00000130
       0000000000000180  0000000000000018          12    11     8
  [12] .strtab           STRTAB           0000000000000000  000002b0
       0000000000000066  0000000000000000           0     0     1
Key to Flags:
  W (write), A (alloc), X (execute), M (merge), S (strings), l (large)
  I (info), L (link order), G (group), T (TLS), E (exclude), x (unknown)
  O (extra OS processing required) o (OS specific), p (processor specific)

```

### 文件头

`readelf -h`的输出为ELF Header信息，定义了ELF魔术、文件机器字节长度、数据存储方式、版本、运行平台、ABI版本、ELF重定位类型、硬件平台、硬件平台版本、入口地址、程序头入口和长度、段表的位置和长度及段的数量等。ELF Header结构被定义在`/usr/include/elf.h`中，其中的变量分别对应了上述输出的一条信息，具体可以自行研究，假如不深究细节，readelf输出的信息已经足够用于分析ELF文件结构。

### 段表

根据ELF Header的信息，SimpleSection.o有13个段，段表就是记录这些段的基本属性的结构，例如段名、段的长度、在文件中的偏移以及其他属性。

从ELF Header可以看出段表的在SImpleSection.o中的偏移为`0x428`，段表中每个段的长度为64 bytes，总共有13个段，也即Section Table的大小为832bytes，加上offset等于1896 bytes，也即SimpleSection.o的大小。

段表的结构比较简单，是一个`Elf_64Shdr`结构体数组，数组个数等于段的个数，每个数组元素对应一个段。

注：ELF段表的第一个段永远是无效的段描述符，类型为`NULL`。

根据readelf -s的输出可以确认SimpleSection.o的文件结构如下图所示：

![SimpleSection.o](C:\Users\RicardoLu\Desktop\SimpleSection.o.png)

### 重定位表

链接器在处理目标文件时，需要对目标文件中某些部位进行重定位，即代码段和数据段中那些绝对地址的引用为位置。这些重定位信息记录在ELF文件的重定位表中，只要是需要进行重定位的段都会有一个相应的重定位表。在SimpleSection.o中可以看到`.rel.text`（引用了printf）和`.rel.eh_frame`两个重定位表，一个重定位表同时也是ELF文件中的一个段。

### 字符串表

ELF文件中用到了很多字符串，比如段名、变量名等，因为字符串的长度往往不是固定的，假如要使用固定的结构来表示字符串，那么将造成很多的空间浪费，因此一种常见的作法时把字符串集中起来存放到一个表，然后使用字符串在表中的偏移来引用字符串（从offset起点到字符串结束符`\0`为一个字符串），在这种情况下只需要一个offset数字即可表示一个字符串而无需考虑字符串长度。

ELF文件中的字符串表包含`.strtab`（String Table）和`.shstrtab`（Section Header String Table）两个段，`.strtab`为普通的字符串表，用于保存符号的名字；`shstrtab`为段表字符串表，用于保存段表中的字符串（段名）。

在ELF Header中可以看到SimpleSection.o的段表字符串表的下标为10，对照readelf -S的输出，恰好是10。

### 数据段和只读数据段

```shell
ts@ts-OptiPlex-7070:~/Downloads/test$ objdump -s -d SimpleSection.o 
...
# 16进制输出
Contents of section .data:
 0000 54000000 55000000                    T...U...        
Contents of section .rodata:
 0000 25640a00                             %d..            
...
```

`.data`段保存的是已经初始化了的全局静态变量和局部静态变量，即`global_init_varabal`和`static_var`，共8个字节，`.data`中的输出为`0x0054`和`0x0055`正好对应了这两个变量的值84和85。

`.rodata`段存放的是只读数据，一般是程序里的只读变量和字符串常量，在SimpleSection.c的func1中调用printf是用到了一个字符串常量`%d\n`，对应的16进制ascii码为`0x25`，`0x64`，`0x0a`，最后以`0x00`即`\0`结尾。

### BSS段

`.bss`段存放的是未初始化的全局变量和局部静态变量，即`global_uninit_var`和`static_var2`，但是我们可以看到`.bss`段的size为4字节而不是8字节，这是因为某些编译器只会为未定义的全局变量预留一个符号，等待最终链接成可执行文件的时候再在`.bss`段分配空间，因此更准确的说法是`.bss`段为这两个变量预留了空间。

```c
static int x1 = 0;
static int x2 = 1;
```

这是一个特殊情况，x1将被放入`.bss`段而x2则将如设计的一样被放入`.data`段，这是因为x1虽然被初始化了，但是由于初始值为0，但是由于随着编译器的发展，大多数未初始化的变量都会被编译器自动初始化为0，因此对于编译器来说0也可以被认为是未初始化的，因此在编译器的优化过程中x1同样会被放入`.bss`段中。

### 代码段

```shell
ts@ts-OptiPlex-7070:~/Downloads/test$ objdump -s -d SimpleSection.o 
# 忽略各段的16进制输出
# 只关注代码段的汇编代码
Disassembly of section .text:

0000000000000000 <func1>:
   0:	55                   	push   %rbp
   1:	48 89 e5             	mov    %rsp,%rbp
   4:	48 83 ec 10          	sub    $0x10,%rsp
   8:	89 7d fc             	mov    %edi,-0x4(%rbp)
   b:	8b 45 fc             	mov    -0x4(%rbp),%eax
   e:	89 c6                	mov    %eax,%esi
  10:	bf 00 00 00 00       	mov    $0x0,%edi
  15:	b8 00 00 00 00       	mov    $0x0,%eax
  1a:	e8 00 00 00 00       	callq  1f <func1+0x1f>
  1f:	90                   	nop
  20:	c9                   	leaveq 
  21:	c3                   	retq   

0000000000000022 <main>:
  22:	55                   	push   %rbp
  23:	48 89 e5             	mov    %rsp,%rbp
  26:	48 83 ec 10          	sub    $0x10,%rsp
  2a:	c7 45 f8 01 00 00 00 	movl   $0x1,-0x8(%rbp)
  31:	8b 15 00 00 00 00    	mov    0x0(%rip),%edx        # 37 <main+0x15>
  37:	8b 05 00 00 00 00    	mov    0x0(%rip),%eax        # 3d <main+0x1b>
  3d:	01 d0                	add    %edx,%eax
  3f:	8d 50 02             	lea    0x2(%rax),%edx
  42:	8b 45 fc             	mov    -0x4(%rbp),%eax
  45:	01 d0                	add    %edx,%eax
  47:	89 c7                	mov    %eax,%edi
  49:	e8 00 00 00 00       	callq  4e <main+0x2c>
  4e:	8b 45 f8             	mov    -0x8(%rbp),%eax
  51:	c9                   	leaveq 
  52:	c3                   	retq 
```

