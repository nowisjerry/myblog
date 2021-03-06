# spring支持的数据库事务传播属性和事务隔离级别

2020-12-03 11:20:10

------

title: SSM - spring支持的常用数据库事务传播属性和事务隔离级别
date: 2020年11月11日 17:05:40
tag:
\- ssm
\- 数据库
\- 事务
\- 面试题

------

收获：ssm学习笔记。

## 前言

事务是恢复和[并发控制](https://baike.baidu.com/item/并发控制)的基本单位。

事务应该具有4个属性：原子性、一致性、隔离性、持久性。这四个属性通常称为**ACID特性**。

| 属性                  | 释义                                                         |
| --------------------- | ------------------------------------------------------------ |
| 原子性（atomicity）   | 一个事务是一个不可分割的工作单位，事务中包括的操作要么都做，要么都不做。 |
| 一致性（consistency） | 事务必须是使数据库从一个一致性状态变到另一个一致性状态。一致性与原子性是密切相关的。 |
| 隔离性（isolation）   | 一个事务的执行不能被其他事务干扰。即一个事务内部的操作及使用的数据对并发的其他事务是隔离的，并发执行的各个事务之间不能互相干扰。 |
| 持久性（durability）  | 持久性也称永久性（permanence），指一个事务一旦提交，它对数据库中数据的改变就应该是永久性的。接下来的其他操作或故障不应该对其有任何影响。 |

## 1. Spring支持的常用数据库事务传播属性和事务隔离级别

```java
//请简单介绍Spring支持的常用数据库事务传播属性和事务隔离级别？

/**
 * 事务的属性：
 *     1.★propagation：用来设置事务的传播行为
 *        事务的传播行为：一个方法运行在了一个开启了事务的方法中时，当前方法是使用原来的事务还是开启一个新的事务
 *        -Propagation.REQUIRED：默认值，使用原来的事务
 *        -Propagation.REQUIRES_NEW：将原来的事务挂起，开启一个新的事务
 *     2.★isolation：用来设置事务的隔离级别
 *        -Isolation.REPEATABLE_READ：可重复读，MySQL默认的隔离级别
 *        -Isolation.READ_COMMITTED：读已提交，Oracle默认的隔离级别，开发时通常使用的隔离级别
 */
```

## 2. 事务的7个传播属性：

当事务方法被另一个事务方法调用时，必须指定事务应该如何传播。例如：方法可能继承在现有事务中运行，也可能开启一个新事务，并在自己的事务中运行。 事务的传播行为可以由传播属性指定。Spring定义了7种类型的传播行为。

| 传播属性          | 描述                                                         |
| ----------------- | ------------------------------------------------------------ |
| REQUIRED （默认） | 如果有事务在运行，当前的方法就在这个事务内运行，否则，就启动一个新的事务，并在自己的事务内运行 |
| REQUIRES_NEW      | 当前的方法必须启动新的事务，并在它自己的事务内运行，如果有事务正在运行，应该将它挂起 |
| SUPPORTS          | 如果有事务正在运行，当前的方法就在这个事务内运行。否则它可以不运行在事务中 |
| NOT_SUPPORTED     | 当前的方法不应该运行在事务中。如果有运行的事务，将它挂起     |
| MANDATORY         | 当前的方法必须运行在事务内部。如果没有正在运行的事务，就抛出异常 |
| NEVER             | 当前的方法不应该运行在事务中。如果有运行的事务,就抛出异常    |
| NESTED            | 如果有事务在运行，当前的方法就应该在这个事务的嵌套事务内运行。否则，就启动一个新的事务，并在它自己的事务内运行 |

## 3. 事务的隔离级别

### (1) 数据库事务并发问题

```
假设现在有两个事务：Transaction01和Transaction02并发执行
```

- 脏读
  - Transaction01 将某条记录的AGE值从20修改为30
  - Transaction02 读取了 Transaction01 更新后的值：30
  - Transaction01 回滚，AGE值恢复到了20
  - Transaction02 读到的30就是一个无效的值
- 不可重复读
  - Transaction01读取了AGE值为20
  - Transaction02将AGE值修改为30
  - Transaction01再次读取AGE值为30，和第一次读取的不一致
- 幻读
  - Transaction01读取了stu表中的一部分数据
  - Transaction02向stu表中插入了新的行
  - Transaction01读取stu表时，多出了一些行

### (2) 事务的隔离级别

数据库系统必须具有隔离并发运行各个事务的能力，使他们不会相互影响，避免各种并发问题。一个事务与其他事务隔离的程度称为隔离级别。SQL标准中规定了好多事务隔离级别，不同隔离级别对应不同的干扰程度，隔离级别越高，数据一致性就越好，性能开销越大

- 读未提交：**READ UNCOMMITTED**
  允许Transaction01读取Transaction02未提交的修改
  (上面三种情况都不可避免）

- 读已提交：**READ COMMITTED**（常用的，其他事务已经提交的数据都认为是真的）

  要求Transaction01只能读取Transaction02已提交的修改
  （可避免脏读）

- 可重复读：**REPEATABLE READ**
  确保Transaction01可以多次从一个字段中读取到相同的值，即Transaction01执行期间禁止其他事务对这个字段进行更新。
  （可避免脏读，不可重复读）

- 串行化：**SERIALIZABLE**
  确保Transaction01可以从一个表中读取到相同的行，在Transaction01执行期间，禁止其他事务对这个表进行增删改操作。可以避免任何并发问题，但性能十分低下。

**各个隔离级别解决并发问题的能力见下表**

| 隔离级别         | 脏读 | 不可重复读 | 幻读 |
| ---------------- | ---- | ---------- | ---- |
| READ UNCOMMITTED | 有   | 有         | 有   |
| READ COMMITTED   | 无   | 有         | 有   |
| REPEATABLE READ  | 无   | 无         | 有   |
| SERIALIZABLE     | 无   | 无         | 无   |

**各种数据库产品对事务隔离级别的支持程度**

| 隔离级别         | Oracle  | MySQL   |
| ---------------- | ------- | ------- |
| READ UNCOMMITTED | x       | √       |
| READ COMMITTED   | √(默认) | √       |
| REPEATABLE READ  | x       | √(默认) |
| SERIALIZABLE     | √       | √       |





[(17条消息) spring支持的数据库事务传播属性和事务隔离级别_Harbour_zhang的博客-CSDN博客](https://blog.csdn.net/Harbour_zhang/article/details/110524005)