# MySQL进阶—锁

## 概述

### 分类

- 全局锁：锁定数据库中的所有表
- 表级锁：每次操作锁住整张表
- 行级锁：每次操作锁住对应的行数据

### 全局锁

- 加锁后整个实例就处于只读状态，后续的DML、DDL语句、已经更新操作的事务提交语句都将被阻塞

- 应用场景：做全库的逻辑备份，获取一致性视图

- 加全局锁：

```mysql
flush tables with read lock;
```

- 备份数据库：在Windows命令行下执行

```windows
mysqldump -h 主机地址 -u用户名 -p密码 数据库名>存放位置.sql
```

- 解锁：

```mysql
unlock tables;
```

- 特点

  1. 在主库上备份时，备份期间不能执行更新操作，业务基本停摆
  2. 在从库上备份时，备份期间从库不能执行主库同步过来的二进制日志（binlog），会导致主从延迟

- <font color='red'>替代情况</font>

  在InnoDB引擎中，可以在备份时加上参数 --single-transaction 参数来完成不加锁的一致性数据备份

  ```windows
  mysqldump --single-transaction -h 主机地址 -u用户名 -p密码 数据库名>存放位置.sql
  ```

### 表级锁

- 简介：锁定粒度大，发生锁冲突的概率最高，并发度最低。应用在MySIAM、InnoDB、BDB等存储引擎中。

- 分类：

  - 表锁

    - 读锁：只允许所有用户读（DQL），不允许任何用户写（DDL、DML）

    - 写锁：当前用户允许读和写（DDL、DML、DQL），但其他客户不允许读和写（DDL、DML、DQL）

    - 加锁：

      ```mysql
      lock tables 表名 read/write;
      ```

    - 解锁：

      ```mysql
      unlock tables; 或者客户端断开连接
      ```

  - 元数据锁（MDL）

    - 系统自动控制加锁，无需显式使用，在访问一张表的时候会自动加上。

    - 作用：维护表元数据的数据一致性，在表上有活动事务时，不允许对元数据进行写入操作。<font color='red'>避免DML与DDL冲突。</font>

    - 查看元数据锁：

      ```mysql
      select object_type,object_schema,object_name,lock_type,lock_duration from performance_schema.metadata_locks;
      ```

  - 意向锁

    - InnoDB用来提升锁的效率的，当添加表锁时，需要检查表中是否有行锁与之不兼容，若一行一行检查则效率极低，此时便可以使用意向锁，在添加表锁时，直接检查与意向锁是否冲突，不必一行一行检查，提升了效率。<font color='red'>解决行锁与表锁冲突问题。</font>

    - 意向共享锁（IS）：由语句select ...lock in share mode添加，与表锁中读锁兼容，写锁互斥。

    - 意向排他锁（IX）：有Insert、update、delete、select ... for update添加，与表锁中的读锁和写锁都互斥。

    - 意向锁之间不会互斥。

    - 查看意向锁及行锁：

      ```mysql
      select object_type,object_schema,object_name,lock_type,lock_duration from performance_schema.data_locks;
      ```

### 行级锁

- 简介：锁定粒度最小，发生锁冲突的概率最低，并发度最高，应用在InnoDB引擎中。

- InnoDB默认使用<font color='red'>可重复读事务隔离级别</font>，使用<font color='red'>临键锁</font>进行搜索和索引扫描，以防止幻读。

- InnoDB的行锁是<font color='red'>针对索引加的锁</font>，不通过索引条件索引数据，那么InnoDB将对表中的所有记录加锁，<font color='red'>升级为表锁。</font>

- 自动加锁：

  1. 索引上的等值查询（唯一索引），给<font color='red'>不存在的记录</font>加锁时，优化为<font color='red'>间隙锁</font>，对<font color='red'>已存在</font>的记录进行等值匹配时，将会<font color='red'>自动优化为行锁</font>。
  2. 索引上的等值查询（普通索引），向右遍历时最后一个值不满足查询需求时，next-key lock退化为<font color='red'>间隙锁</font>。<font color='red'>在查询到的记录前后加上间隙锁，该记录加上行锁</font>。
  3. 索引上的范围查询（唯一索引），会访问到不满足条件的第一个值为止，<font color='red'>该记录加行锁，该记录及范围内的记录加上临键锁</font>。

- 分类：

  - 行锁（Record Lock）：<font color='red'>对索引上的索引项加锁实现的，不是对记录加的锁。</font>锁定单个行记录的锁，防止对其他事务进行update和delete。在读可提交（RC）、可重复读（RR）隔离级别下都支持。

    - 共享锁（S）：允许读一行，不允许写，阻止其他事务获取相同数据集的排他锁。<font color='red'>S与S兼容，S与X互斥。</font>
    - 排他锁（X）：允许获取排他锁的事务更新数据，阻止其他事务获取相同数据集的共享锁和排他锁。<font color='red'>X与X和S都互斥。</font>

    | SQL                           | 行锁类型   | 说明                                     |
    | ----------------------------- | ---------- | ---------------------------------------- |
    | insert                        | X          | 自动加锁                                 |
    | update                        | X          | 自动加锁                                 |
    | delete                        | X          | 自动加锁                                 |
    | select                        | 不加任何锁 |                                          |
    | select ... lock in share mode | S          | 需要手动在select之后加lock in share mode |
    | select ... for update         | S          | 需要手动在select之后加for update         |

  - 间隙锁（Gap Lock）：锁定索引记录间隙（不含该记录），确保索引记录间隙不变，<font color='red'>防止其他事务在这个间隙进行insert，避免产生幻读</font>。在RR隔离级别下支持。<font color='red'>间隙锁之间兼容。</font>
  - 临键锁（Next-Key Lock）：行锁和间隙锁组合，同时锁住数据，并锁住数据前面的间隙Gap。在RR隔离级别下支持。