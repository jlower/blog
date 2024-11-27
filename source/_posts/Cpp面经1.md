---
title: Cpp面经1
date: 2024-11-26 03:00:00
categories: 面经
---

## C++ lambda怎么实现

## C++ shared_ptr 实现

## proactor\reactor模式怎么使用

## 介绍 move 与 完美转发foward

## 编译器怎么优化模板

```cpp
template <typename T>
class Base
{
public:
    void func()
    {
        static_cast<T *>(this)->func1();
    }
};
class D : public Base<D>
{
public:
    void func1()
    {
        cout << "s" << endl;
    }
};
Base<D> *d = new D();
d->func();
```

```cpp
class Base
{
public:
    virtual void func()
    {
    }
};
class D : public Base
{
public:
    virtual void func() override
    {
        cout << "s";
    }
};

Base *d = new D();
d->func();
```

> 上面两段1（模板）比2（虚函数）快，模板编译时就确定了直接调用即可（静态），虚函数需要先查虚函数表再调用函数（动态）

## 介绍NAT

## 树，数组，链表分别什么情况下用

## 基于udp实现像tcp一样的稳定协议（QUDP）

## tcp粘包加什么分隔符解决

> Netty解决TCP粘包/拆包相关类以及功能

1. LineBasedFrameDecoder：以```\r```或```\r\n```为分隔符
1. StringDecoder：将接收到的消息转换成字符串
1. DelimiterBasedFrameDecoder：自定义分隔符
1. FixedLengthFrameDecoder：定长解析

## c++栈空间多少，爆栈后编译器会有什么操作

## 最大公约数gcd

## sql设计：orders表 和 user表 ，找出买东西数量top10的user输出他们的信息

```sql
select
    customer_number
from
    Orders
group by
    customer_number
order by
    count(*) desc
limit 10
```

## 数学：社会上1%的概率得病，事实上ill检测为ill的概率95%检测为healthy的概率5%；事实上healthy检测为ill的概率5%检测为healthy的概率95%

1. 一个人检测一次结果是ill并且此人事实上为ill的概率```(1%*95%)/(1%*95%+99%*5%)=16.1%```
1. 一个人检测两次结果都是ill并且此人事实上为ill的概率```(1%*95%*95%）/(1%*95%*95%+99%*5%*5%)=78.478%```
