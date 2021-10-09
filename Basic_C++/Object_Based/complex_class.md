# Complex

## Introduction

- Object Based：基于对象，面对的是单一的Class设计；
- Object Oriented：面向对象，面对的是多重Classes的设计，处理Classes之间的关系。

### 代码基本组成形式

- .h(Class Declaration)
- .cpp(Program implement + main function)
- .h(Link Library)

### 头文件中的防卫式声明

```c++
// complex.h
#ifndef __COMPLEX_H__
#define __COMPLEX_H__

#pragma once
/* forward declaration */
class complex;
complex& __doapl (complex* ths, const complex& r);
complex& __doami (complex* ths, const complex& r);
complex& __doaml (complex* ths, const complex& r);

/* class declaration */
class complex
{
public:
    complex (double r = 0, double i = 0): re (r), im (i) { }
    complex& operator += (const complex&);
    complex& operator -= (const complex&);
    complex& operator *= (const complex&);
    complex& operator /= (const complex&);
    double real () const { return re; }
    double imag () const { return im; }
private:
    double re, im;

    friend complex& __doapl (complex *, const complex&);
    friend complex& __doami (complex *, const complex&);
    friend complex& __doaml (complex *, const complex&);
};
/* class definition */
inline complex complex::funtion ...
#endif // __COMPLEX_H__
```

## 内联函数

- 函数若在class body内定义完成，便自动成为inline候选人

## 构造函数

```c++
complex (double r = 0, double i = 0): re (r), im (i) { }
```

构造函数带默认实参，使用初始列完成成员变量的初始化，性能上优于赋值，因为对于非内嵌类型(int, char, float)来说赋值实际要调用拷贝赋值，先拷贝再赋值。

## 重载

C++与C的一个显著不同就是C++支持重载，一个函数可以有多种不同的声明，提高程序的可拓展性，重载的底层逻辑是函数经过mangling之后变成一个全局的包含参数列等信息在内的新的函数名，调用时，编译器会修改为新函数（对象指针）。

假设complex还有一个构造函数：

```c++
complex (): re(0), im(0) {}
```

那么这两个构造函数在编译后的名称可能为：

```shell
?real@Complex@@QBENXZ
?real@Complex@@QAENABN@Z
```

## 常量成员函数

```c++
double real () const { return re; }
double imag () const { return im; }
```

const修饰函数代表函数不会修改数据的内容。

假设real和imag没有限定为常量成员函数，考虑如下情况：

```c++
const complex c1 (2,1);
cout << c1.real();
cout << c1.imag();
```

c1是一个常量对象，c1中的成员变量是常量，不能修改，但是假如real和imag不是常量成员函数，那么就意味着c1.real将返回一个可修改的非常量变量，这显然是矛盾的，编译器将抛出错误：将const成员变量赋给非const变量。

### 友元

```c++
inline complex& __doapl (complex* ths, const complex& r)
{
    ths->re += r.re;
    ths->im += r.im;
    return *ths;
}
```

- 相同class的各个object互为友元，可以直接访问对方的私有内容。

## 操作符重载

```c++
inline complex& complex::operator += (const complex& r)
{
    return __doapl (this, r);
}
```

```c++
inline complex operator + (const complex& x, const complex& y)
{
    return complex (real (x) + real (y), imag (x) + imag (y));
}

inline complex operator + (const complex& x, double y)
{
    return complex (real (x) + y, imag (x));
}

inline complex operator + (double x, const complex& y)
{
    return complex (x + real (y), imag (y));
}
```

- 设计为成员函数将带有一个this指针指向调用的对象。
- 返回引用的前提是返回值为非local对象。

```c++
ostream& operator << (ostream& os, const complex& x)
{
  return os << '(' << real (x) << ',' << imag (x) << ')';
}
```

- 返回引用以处理多重输出，例如`cout << c1 << endl;`将鲜输出c1，假设返回值不是ostreamer类型，那么就无法继续输出`endl`。

