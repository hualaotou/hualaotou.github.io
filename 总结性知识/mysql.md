# mysql

### 基础概念

* 数据库存储引擎

  * innodb存储引擎
    * InnoDB是事务型数据库的首选引擎，支持事务安全表（ACID），支持行锁定和外键;InnoDB是默认的MySQL引擎
    * 特点：支持事务处理、支持外键、支持崩溃修复能力和并发控制
  * MYISAM存储引擎
    * MyISAM基于ISAM存储引擎，并对其进行扩展。它是在Web、数据仓储和其他应用环境下最常使用的存储引擎之一。MyISAM拥有较高的插入、查询速度，但不支持事务
    * 特点：插入数据快，空间和内存使用比较低
  * MEMORY存储引擎
    * MEMORY存储引擎将表中的数据存储到内存中，未查询和引用其他表数据提供快速访问
    * 特点：数据处理快，但是安全性不高

* 引擎功能对比

  ![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211026103342.png)

* 存储引擎查看

  ```mysql
  MariaDB [(none)]> show engines;
  
  mysql> alter table service engine=innodb;
  ```

  ![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211026105114.png)

* mysql事务
  *  一般来说，事务是必须满足4个条件（ACID）：：原子性（Atomicity，或称不可分割性）、一致性（Consistency）、隔离性（Isolation，又称独立性）、持久性（Durability）。
    * 原子性：一个事务（transaction）中的所有操作，要么全部完成，要么全部不完成，不会结束在中间某个环节。事务在执行过程中发生错误，会被回滚（Rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。
    * 一致性：在事务开始之前和事务结束以后，数据库的完整性没有被破坏。这表示写入的资料必须完全符合所有的预设规则，这包含资料的精确度、串联性以及后续数据库可以自发性地完成预定的工作。
    * 隔离性：数据库允许多个并发事务同时对其数据进行读写和修改的能力，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致。事务隔离分为不同级别，包括读未提交（Read uncommitted）、读提交（read committed）、可重复读（repeatable read）和串行化（Serializable）。
    * 持久性：事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失。
  *  在 MySQL 中只有使用了 Innodb 数据库引擎的数据库或表才支持事务
  *  事务处理可以用来维护数据库的完整性，保证成批的 SQL 语句要么全部执行，要么全部不执行。
  * 事务用来管理 insert,update,delete 语句

* MySQL事务处理主要有两种方法

  * 用 BEGIN, ROLLBACK, COMMIT来实现
    * BEGIN 开始一个事务
    * ROLLBACK 事务回滚
    * COMMIT 事务确认
  * 直接用 SET 来改变 MySQL 的自动提交模式:
    * SET AUTOCOMMIT=0 禁止自动提交
    * SET AUTOCOMMIT=1 开启自动提交

  ```mysql
  >show variables like 'autocommit';     //查看是否修改成功
  
  # 注意：在编写应用程序时，最好事务的控制权限交给开发人员
  ```

### mysql库表操作

###  mysql备份机制

