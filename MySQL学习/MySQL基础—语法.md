[TOC]

# MySQL基础语法

## 通用语法

1. 以分号结尾

2. 不区分大小写，关键字建议使用大写

3. 注释：

   * 单行注释：--注释内容 或 #注释内容（MySQL特有）

   * 多行注释：/* 注释内容 */

     

## SQL分类

- DDL：数据定义语言，定义数据库对象（数据库，表，字段）

- DML：数据操作语言，对表中数据进行增删改

- DQL：数据查询语言，查询数据库中表的记录

- DCL：数据控制语言，创建数据库用户、控制数据库的访问权限

  

## DDL

- 操作数据库

  ```mysql
  查询所有数据库：show databases;
  查询当前数据库：select database();
  创建：create database [if not exists] 数据库名 [default charset 字符集] [collate 排序规则];
  删除：drop database [if exists] 数据库名;
  使用：use 数据库名;
  ```

- 操作表

  1. 查询当前数据库所有表：

     ```mysql
     show tables;
     ```

  2. 查询表结构（表的描述信息）：

     ```mysql
     desc 表名;
     ```

  3. 查询指定表的建表语句：

     ```mysql
     show create table 表名；
     ```

  4. 创建表：

     ```mysql
     create table 表名(
     		字段1 字段1类型[comment 字段1注释],
     		字段2 字段2类型[comment 字段2注释]
     	)[comment 表注释];
     ```

  5. 添加字段：

     ```mysql
     alter table 表名 add 字段名 字段类型（长度）[comment 注释][约束];
     ```

  6. 修改数据类型：

     ```mysql
     alter table 表名 modify 字段名 新数据类型（长度）;
     ```

  7. 修改字段名和字段类型：

     ```mysql
     alter table 表名 change 旧字段名 新字段名 新数据类型（长度）;
     ```

  8. 删除字段：

     ```mysql
     alter table 表名 drop 字段名;
     ```

  9. 修改表名：

     ```mysql
     alter table 表名 rename to 新表名;
     ```

  10. 删除表：

      ```mysql
      drop table [if exists] 表名;
      ```

  11. 删除表，并重新创建该表（实际是删除表的所有数据）：

      ```mysql
      truncate table 表名;
      ```

  __课堂练习__:

  ```mysql
  1. 设计一张员工信息表，要求：（1）编号（纯数字）（2）员工工号（字符串类型，长度不超过10位）（3）员工姓名（字符串类型，长度不超过10）（4）性别（男/女，存储一个汉字）
  create table employee(
  	id int comment '编号',
  	workno varchar(10),
  	name varchar(10),
  	gender char(1),
  	age tinyint unsigned comment '年龄，不可能存储负数',
  	idcard char(18) comment '身份证号',
  	entrytime data comment '入职时间，年月日'
  ) comment '员工表';
  2. alter table employee add nickname varchar(10) comment '昵称'; --添加字段
  3. alter table employee change nickname username varchar(10) comment '用户名';--修改字段	名和字段类型
  4. alter table employee drop username; --删除字段
  5. alter table employee rename to emp; --修改表名
  ```

## DML

* 添加数据（insert）

  1. 给指定字段添加数据

     ```mysql
     insert into 表名(字段名1,字段名2，...) values(值1，值2，...);
     ```

  2. 给全部字段添加数据

     ```mysql
     insert into 表名 values(值1，值2，...);
     ```

  3. 批量添加数据

     ```mysql
     insert into 表名(字段名1，字段名2，...) values(值1，值2，...),(值1，值2，...);
     insert into 表名 values(值1，值2，...),(值1，值2，...);
     ```

     <font color='red'>注意：字段名与值一一对应，不可超出字段规定的大小，字符串和日期需用引号。</font>

* 修改数据（update）

  ```mysql
  update 表名 set 字段名1=值1,字段名2=值2,... [where 条件]
  ```

* 删除数据（delete）

  ```mysql
  delete from 表名 [where 条件]
  ```

  <font color='red'>注意：delete 语句不能删除某个字段的值（可以用update）</font>

## DQL

- 基本查询

  ```mysql
  1.查询多个字段
  	select * from 表名;
  2.设置别名
  	select 字段1[as 别名1],字段2[as 别名2]... from 表名;
  3.去除重复记录
  	select distinct 字段1,字段2... from 表名;
  ```

  

- 条件查询

  ```mysql
  1. 条件运算符
  >,>=,<,<=,=,<>不等于 或!=
  between...and...在某个范围之内，包含两端
  in(...) 在in之后的列表中的值
  like 占位符   模糊匹配(_匹配单个字符，%匹配任意字符)
  is null 是null
  is not null 不是null
  and或&& 
  or或||
  not或!
  2. like使用
  select * from emp where name like '___';--三个下划线代表三个字
  select * from emp where idcard like '%X';--查询身份证号最后一位为X的员工
  ```

- 聚合函数

  | 函数  | 功能     |
  | ----- | -------- |
  | count | 统计数量 |
  | max   | 最大值   |
  | min   | 最小值   |
  | avg   | 平均值   |
  | sum   | 求和     |

  ```mysql
  select 聚合函数(字段名列表) from 表名;
  ```

  <font color='red'>null值不参与所有聚合函数运算</font>

- 分组查询

  ```mysql
  select 字段列表 from 表名 [where 条件]group by 分组字段名[having 分组后过滤条件];
  ```

  > where 和having的区别
  >
  > · 执行时机不同：where是分组之前进行过滤，不满足where条件，不参与分组；而having是分组之后对结果进行过滤。
  >
  > · 判断条件不同：where不能对聚合函数进行判断，而having可以。

  课堂练习

  ```mysql
  1. 查询年龄小于45的员工，并根据工作地址分组，获取员工数量大于等于3的工作地址
  select workaddress,count(*) address_count from emp where age<45 group by workaddress having address_count>=3;
  ```

- 排序查询

  - asc：升序（默认值）
  - desc：降序

  ```mysql
  select 字段列表 from 表名 order by 字段1 排序方式1，字段2 排序方式2;
  ```

  <font color='red'>多字段排序时，当第一个字段值相同时，才会根据第二个字段进行排序</font>

- 分页查询(limit)

  ```mysql
  select 字段列表 from 表名 limit 起始索引,查询记录数;
  ```

  - 起始索引从0开始，起始索引=（查询页面-1）*每页显示记录数
  - 不同数据库不一样，mysql是limit
  - 查询第一页时，起始索引可以省略
  - limit放句末

  课堂练习

  ```mysql
  1. 查询第一页员工数据，每页显示10条记录
  select * from emp limit 0,10;
  或者：select * from emp limit 10;
  2. 查询第3页员工数据，每页显示10条记录
  select * from emp limit 20,10;
  ```

- 课后练习

  ```mysql
  1. 
  select * from emp where gender='女' and age in(20,21,22,23);
  select * from emp where gender='女' and age between 20 and 23;
  2. 
  select * from emp where gender='男' and age between 20 and 40 and name like '___';
  3.
  select gender,count(*) from emp where age<60 group by gender;
  4.
  select name,age from emp where age<=35 order by age asc,entrytime desc;
  5.
  select * from emp where gender='男' and age between 20 and 40 order by age asc, entrytime asc limit 5;
  ```

- 执行顺序

  > from->where->group by->having->select->order by->limit

## DCL

- 管理用户

  1. 查询用户

     ```mysql
     use mysql; --mysql数据库的所有用户信息存在于mysql中
     select * from user;
     ```

  2. 创建用户

     ```mysql
     create user '用户名' @ '主机号' identified by '密码';
     ```

  3. 修改用户密码

     ```mysql
     alter user '用户名' @ '主机号' identified with mysql_native_password by '新密码';
     ```

  4. 删除用户

     ```mysql
     drop user '用户名' @ '主机号';
     ```

     <font color='red' >主机名可以使用通配符%替代</font>

- 权限控制

  | 权限               | 说明               |
  | ------------------ | ------------------ |
  | all,all privileges | 所有权限           |
  | select             | 查询数据           |
  | insert             | 插入数据           |
  | update             | 修改数据           |
  | delete             | 删除数据           |
  | alter              | 修改表             |
  | drop               | 删除数据库/表/视图 |
  | create             | 创建数据库/表      |

  1. 查询权限

     ```mysql
     show grants for '用户名' @ '主机号'
     ```

  2. 授予权限

     ```mysql
     grant 权限列表 on 数据库名.表名 to '用户名' @ '主机号'
     ```

  3. 撤销权限

     ```mysql
     revoke 权限列表 on 数据库名.表名 from '用户名' @ '主机号'
     ```

  <font color='red'>数据库名和表名都可以用通配符*替代，代表所有</font>

  