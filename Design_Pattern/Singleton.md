# Singleton

## Overview

单例模式是一种创建型设计模式，让你能够保证 一个类只有一个实例，并提供一个访问该实例的全局节点。

单例模式是设计模式中最为简单的一个模式，拥有与全局变量相同的优缺点，因此尽管有时单例模式非常有用，但却会破坏代码的模块化特性，增加模块间的耦合。

实现单例模式的核心在于：

- 私有化默认构造函数，禁用拷贝构造和拷贝赋值；
- 对外只提供一个接口，接口内部构造一个静态成员变量。

```c++
/**
 * The Singleton class defines the `GetInstance` method that serves as an
 * alternative to constructor and lets clients access the same instance of this
 * class over and over.
 */
class Singleton
{

    /**
     * The Singleton's constructor should always be private to prevent direct
     * construction calls with the `new` operator.
     */

protected:
    Singleton(const std::string value): value_(value)
    {
    }

    static Singleton* singleton_;

    std::string value_;

public:

    /**
     * Singletons should not be cloneable.
     */
    Singleton(Singleton &other) = delete;
    /**
     * Singletons should not be assignable.
     */
    void operator=(const Singleton &) = delete;
    /**
     * This is the static method that controls the access to the singleton
     * instance. On the first run, it creates a singleton object and places it
     * into the static field. On subsequent runs, it returns the client existing
     * object stored in the static field.
     */

    static Singleton *GetInstance(const std::string& value);
    /**
     * Finally, any singleton should define some business logic, which can be
     * executed on its instance.
     */
    void SomeBusinessLogic()
    {
        // ...
    }

    std::string value() const{
        return value_;
    } 
};

Singleton* Singleton::singleton_= nullptr;;

/**
 * Static methods should be defined outside the class.
 */
Singleton *Singleton::GetInstance(const std::string& value)
{
    /**
     * This is a safer way to create an instance. instance = new Singleton is
     * dangeruous in case two instance threads wants to access at the same time
     */
    if(singleton_==nullptr){
        singleton_ = new Singleton(value);
    }
    return singleton_;
}
```

##  线程安全

上文展示了一个最简单的懒汉式单例模式的实现，很显然这是线程不安全的，考虑一种极端状态：

- 线程A进入Singleton的构造函数并且通过了非空判断时被挂起，这是线程A的下一步操作就是去new一个Singleton对象；
- 线程B在这时也通过GetInstance函数成功new了一个Singleton对象；
- 线程A被唤醒，继续完成挂起前未完成的new操作；
- 结果就是new了两个对象。

因此大多数关于单例模式的教程中都会通过加锁对单例模式的线程安全进行处理，但是仅仅是加锁，那么每次新建线程都需要去获取锁，并发性能会很差，所以改进为Double check+Lock，但事实上在C++11之后，可以通过局部静态变量来避免这种情况：

```c++
class Singleton
{
public:
	// 注意返回的是引用
	static Singleton& getInstance()
	{
		static Singleton value;  //静态局部变量
		return value;
	}

private:
	Singleton() = default;
	Singleton(const Singleton& other) = delete; //禁止使用拷贝构造函数
	Singleton& operator=(const Singleton&) = delete; //禁止使用拷贝赋值运算符
};
```

侯捷在他的课中提到过另一个饿汉式单例模式：

```c++
class Singleton
{
public:
	// 注意返回的是引用
	static Singleton& getInstance()
	{
		return value;
	}

private:
	Singleton() = default;
	Singleton(const Singleton& other) = delete; //禁止使用拷贝构造函数
	Singleton& operator=(const Singleton&) = delete; //禁止使用拷贝赋值运算符
    
    static Singleton value;  //静态局部变量
};
Singleton Singleton::value;
```

懒汉式：在需要用到对象时才实例化对象，常规的实现方式是：**Double Check + Lock**，解决了并发安全和性能低下问题，C++11之后可以直接使用局部静态变量改进。

饿汉式：在**类加载**时已经创建好该单例对象，在获取单例对象时直接返回对象即可，不会存在并发安全和性能问题。

## FAQ

事实上这也并不算是正真的单例模式，在实际开发过程中，假如我们使用显示调用DLL，由于静态变量是加载时创建，所以DLL加载的时候就会先new对象出来，然后才进dll_main，也就是说你的程序加载了多个DLL，那么每个DLL都会实例化一个它自身认为的单例，具体的解决方法还有待调查。

