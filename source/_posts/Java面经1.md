---
title: Java面经1
date: 2024-11-27 03:00:00
categories: 面经
---

## 一个接口是可以继承多个接口的

## 抽象类实现一个接口时，可以实现也可以不实现接口中的方法

## interface可用的修饰符只有public

## 构造方法中如果没有super语句，默认在构造方法的第一行会加一个super()语句

## 在同一个构造方法中this(参数)语句和super(参数)语句都必须放在第一行，那么在同一个构造方法中this(参数)语句和super(参数)语句只能有一个

## 在Java语言中 ```int m[]=new int[5]``` 用 ```m.length``` 表示数组的长度，数组被当作对象包含一个变量length

## 解析CQRS架构模式

## 解析TDD架构模式

## 常用于微服务架构中的Sidecar设计模式

## Java迭代器的作用，迭代器能边遍历边更改吗？

1. [【Java基础】 迭代器](https://blog.csdn.net/weixin_73442302/article/details/139480150)
1. [List遍历时删除](https://www.cnblogs.com/wunsiang/p/12765144.html)
1. [ListIterator](https://blog.csdn.net/huaairen/article/details/86687514)
1. [listIterator，可以边遍历边修改](https://blog.csdn.net/gooooa/article/details/77530112)
1. [Java集合—List如何一边遍历，一边删除？](https://blog.csdn.net/sanmi8276/article/details/114756004)
1. [以小见大，ArrayList在遍历时删除会抛异常吗？](https://juejin.cn/post/7197972831794610231)
1. [Java中为什么在迭代器遍历的过程中修改原有集合中的内容会报错？为什么要这样设计？](https://blog.csdn.net/weixin_73922932/article/details/140701810)
1. [Java集合遍历的方法有哪些](https://xiaolincoding.com/interview/collections.html#%E9%9B%86%E5%90%88%E9%81%8D%E5%8E%86%E7%9A%84%E6%96%B9%E6%B3%95%E6%9C%89%E5%93%AA%E4%BA%9B)

## 线程池的作用？阻塞队列有啥？最大线程数量和核心线程数量的区别？什么时候创建新的线程？什么时候回收线程？拒绝策略有哪些？线程池有几种状态？线程池异常怎么处理？

## Linux中查看日志的常用命令

1. [Linux查看log日志的命令与各自适合的场景](https://blog.csdn.net/qq_36245532/article/details/102835184)
1. [Linux查看日志的6种方式](https://blog.csdn.net/qq_41248260/article/details/143240234)
1. [Linux查看日志的常用命令](https://blog.csdn.net/qq_37924396/article/details/124878564)

## Kafka的命令，怎么查看Topic？怎么查看消息余量？怎么在项目中用（注解）？有遇到过消息丢失吗，能怎么解决？有遇到过消息重复消费吗，能怎么解决？有部署过集群吗？Zookeeper在Kafka中的作用？

## Redis常用的命令？Redis能用来干啥？

## 怎么分析JVM的性能问题？用啥方法工具？步骤是啥？

## MyBatis怎么批量执行SQL语句？MySQL怎么批量执行SQL语句？MyBatis的xml怎么对应对象的内容？MyBatis中Dao接口里参数不同但同名的方法能重载吗？

## MySQL更新删除的时候会自动加锁，加锁性能消耗大，怎么能不加锁？MySQL中NULL和空字符串的区别？MySQL中常用的数据类型？MySQL中有没有Boolean类型？

## Java的IO用了什么设计模式，适配器模式？装饰器模式？字节流，字符流怎么装饰？

## SpringBoot的event注解用过吗？事件机制是咋样的？SpringBoot自动装配的过程？

## HashTable和HashMap的区别？HashMap是并发安全的吗？
