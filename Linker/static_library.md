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

