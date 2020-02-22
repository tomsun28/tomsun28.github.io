---
layout: post
title:  mysql learn
date: 2020-02-21
tag: database
---
<br>

## mysql learn  

将之前在幕布整理的mysql知识点再学习加深下  

### sql语言  

* 数据查询语言DQL - select show
* 数据操纵语言DML - insert update create
* 数据定义语言DDL - create drop alter
* 数据控制语言DCL - grant commit

----

### 存储引擎  

* **innodb**
  1. 支持事务
  2. 支持行级锁 - where主键时有效，其他情况锁全表
  3. 支持外键  

* **myisam**  
  1. 不支持事务，查询具有原子性  
  2. 锁全表
  3. 不支持外键  

----

### 索引  

#### 索引相关sql  

````
alter table school add index stu_name_indx(stu_name);
alter table school add unique stu_id_index(stu_id);
alter table school add primary key stu_name_key(stu_name);
````
#### 索引类型  

* b+树索引 - 范围和单条数据查询都不错
* 哈希索引 - 单条查询快，范围慢

#### b+树索引  

todo

#### 索引(b+树)设置原则  

todo

----

### 事务  

#### 事务特性ACID  

* atomicity - 原子性    
> undo log实现，保证在一个事务里的操作要么全部成功，要么全部失败，没有中间态

* consistency - 一致性
> 通过原子性，隔离性，持久性保证数据的完整性和一致性，eg:银行转账
> 数据从一个一致性的状态转移到另一个一致性的状态  

* isolation - 隔离性
> 并发控制下多个线程操作数据对数据的隔离性要求，不同隔离级别进而影响到数据库操作性能和一致性  

* durability - 持久性  
> redo log实现，保证已经提交的事务的数据的持久化，不会因为奔溃重启而丢失  

#### 可靠性  

> 事务可靠性靠redo log，undo log保证  
> redo log保证已提交事务的持久化  
> undo log保证事务撤销的原子性

* redo log 重做日志 - 已提交事务的持久性  
> mysql修改数据不是直接永久刷到磁盘，会先修改纪录到buffer_pool，后台线程再同步buffer_pool里面的修改到磁盘  
> 也会将修改纪录到redo_buffer，当事务成功提交会将redo_buffer纪录同步到redo_log  
> 这样事务提交系统奔溃重启会从持久化的redo_log恢复  

* undo log 回滚日志 - 未提交事务的原子性(回滚)
> 同上，mysql每次修改数据之前会将修改纪录到undo log里
> 这样事务未提交系统奔溃重启，会根据undo log信息回滚数据  


#### 并发性  

读写锁
mvcc - 多版本控制 + 乐观锁 + undo log  
mvcc - 版本链(trx_id,roll_pointer)  ReadView - m_ids  
[MySQL事务隔离级别和MVCC](https://juejin.im/post/5c9b1b7df265da60e21c0b57)  

#### 事务的隔离级别  

* Read Uncommitted 读未提交
> 脏读 - 读到一个未提交事务的数据，此事务又回滚  

*  Read Committed 读提交  
> mvcc - 读一次生成一个标记版本，多次读取生成多个ReadView版本  
> 不可重复读 - 一个事务内多次读同一数据，中间段被另一事务修改，这两次读取的数据就可能不一致  

* Repeatable Read 可重复读(mysql默认级别)  
> mvcc - 乐观读，多次读取生成只一个ReadView版本
> 幻读 - 事务在插入的时候已经检查过不存在的纪录时，被另一事务插入此纪录，后事务发现这些数据已经存在导致插入失败  

* Serializable 串行化
> 即对读和写每次都加锁，串行化处理  
> 无脏读，无不可重复读，无幻读  







**相关参考**  
[Mysql事务实现原理](https://juejin.im/post/5cb2e3b46fb9a0686e40c5cb)  
[MySQL事务隔离级别和MVCC](https://juejin.im/post/5c9b1b7df265da60e21c0b57)  


<br>
<br>

*转载请注明* [from tomsun28](http://usthe.com)
