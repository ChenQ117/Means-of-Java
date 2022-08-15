# MySQL基础—约束、多表查询、事务

## 约束

### 分类

| 约束                       | 描述                                                 | 关键字      |
| -------------------------- | ---------------------------------------------------- | ----------- |
| 非空约束                   | 限制该字段的数据不能为null                           | not null    |
| 唯一约束                   | 保证该字段的所有数据都是唯一、不重复的               | unique      |
| 主键约束                   | 主键是一行数据的唯一标识，要求非空且唯一             | primary key |
| 默认约束                   | 若未指定约束，则为默认约束                           | default     |
| 检查约束（8.0.16版本之后） | 保证字段值满足某一个条件                             | check       |
| 外键约束                   | 让两张表的数据之间建立联系，保证数据的一致性和完整性 | foreign key |

<font color='red'>作用于表的字段上，可以在创建表/修改表的时候添加约束</font>

```mysql
create table user(
	id int primary key auto_increment,
	name varchar(10) not null unique,
	age int check(age>0 and age<=120),
	status char(1) default '1',
	gender char(1)
)
```

### 外键

- 添加外键

  ```mysql
  create table 表名(
  	字段名 数据类型,
  	...
  	[constraint][外键名称]foreign key (外键字段名) references 主表(主表列名)
  );
  
  alter table 表名 add constraint 外键名称 foreign key(外键字段名) references 主表(主表列名);
  ```

  ```mysql
  alter table emp add constraint fk_emp_dept_id foreign key (dept_id) references dept(id);
  ```

- 删除外键

  ```mysql
  alter table 表名 drop foreign key 外键名;
  ```

- 外键删除、更新行为

  | 行为        | 说明                                                         |
  | ----------- | ------------------------------------------------------------ |
  | no action   | 在父表中删除更新时，如果有对应外键，则不允许删除更新         |
  | restrict    | 在父表中删除更新时，如果有对应外键，则不允许删除更新（与no action一致） |
  | cascade     | 在父表中删除更新时，如果有对应外键，则也删除更新外键在子表中的记录 |
  | set null    | 在父表中删除是，如果有对应外键，则设置子表中该外键值为null（如果允许为null的话） |
  | set default | 父表有变更时，子表将外键列设置成一个默认的值（innodb不支持） |

  ```mysql
  alter table 表名 add constraint 外键名称 foregin key (外键字段) references 主表名(主表字段名) on update cascade on delete cascade;
  ```

  

## 多表查询

### 多表关系

- 一对多（多对一）

  **关系：**一个部门对应多个员工，一个员工对应一个部门

  **实现：**在多的一方建立外键，指向一的一方的主键（对每个员工设置一个外键）

- 多对多

  **关系：** 一个学生可以有多门课程，一门课程可以有多个学生

  **实现：** 建立中间表，至少包含两个外键，分别关联两方主键

- 一对一

  **关系：** 多用于单表的拆分，将一张表的基础字段放在一张表中，其他详情字段放在另一张表中，以提升效率

  **实现：** 在任意一方加入一个外键，关联另一方的主键，并且设置外键为唯一的（unique）

### 多表查询概述

#### 笛卡尔积

表A和表B的组合

#### 内连接

查询A、B交集部分数据

- 隐式内连接

  ```mysql
  select 字段列表 from 表1，表2 where 条件...;
  ```

- 显示内连接

  ```mysql
  select 字段列表 from 表1 [inner]join 表2 on 连接条件...;
  ```

#### 外连接

- 左外连接：查询左表数据以及两张表交集部分数据

  ```mysql
  select 字段列表 from 表1 left [outer] join 表2 on 条件...;
  ```

- 右外连接：查询右表数据以及两张表交集部分数据

  ```mysql
  select 字段列表 from 表1 right [outer] join 表2 on 条件...;
  ```

- 自连接：当前表与自身的连接查询，自连接必须使用表别名

  ```mysql
  select 字段列表 from 表1 别名A join 表1 别名B on 条件...;
  ```

#### 联合查询

把多次查询的结果上下拼接合并起来，形成一个新的查询结果集

```mysql
select 字符列表 from 表A ...
union [all]
select 字符列表 from 表B ...;
```

- 不加all时可以自动去重
- 多张表的列数必须相同，字段类型也需要保持一致

#### 子连接（嵌套查询）

```mysql
select * from t1 where column1 = (select column1 from t2);
```

- 标量子查询

  返回结果是单个值

- 列子查询

  返回结果为一列

  常用操作符：in, not in, any, some, all

- 行子查询

  返回结果为一行

  常用操作符：=, <>, in, not in

  ```mysql
  --查询与'张无忌'的薪资及直属领导相同的员工信息
  select * from emp where (salary,managerid)=(select salary,managerid from emp where name = '张无忌');
  ```

- 表子查询

  返回结果为多行多列

  常用操作符：in

  ```mysql
  --查询与张三，李四职位和薪资相同的员工信息
  select * from emp where (job,salary) in (select job,salary from emp where name='张三' or name = '李四');
  ```

#### 练习

```mysql
--1.
select e.name,e.age,e.job,dept.* from emp e,dept where e.dept_id=dept.id;
--2. 
select name,age,job,dept.* from emp,dept where emp.dept_id=dept.id and age<30;
   --修正
select e.name,e.age,e.job,d.name from emp e join dept d on e.dept_id = d.id where e.age<30;
--3
select distinct id,name from emp,dept where emp.dept_id = dept.id;
--4
select emp.*,dept.name from emp left join dept on dept.id = emp.dept_id where emp.age>40;
--5
select e.*,s.grade from emp e,salgrade s where e.salary between s.losal and s.hisal;
--6
select e.*,s.grade from emp e,salgrade s,dept d where d.name='研发部' and d.id = e.dept_id and e.salary (between s.losal and s.hisal);
--7
select avg(e.salary) from emp e,dept d where d.name='研发部' and d.id = e.dept_id;
--8 
select e1.* from emp e1,emp e2 where e1.salary > e2.salary and e2.name = '灭绝';
select * from emp where salary > (select salary from emp where name = '灭绝');
--9
select * from emp where salary>(select avg(salary) from emp);
--10
select * from emp e1 where e1.salary<(select avg(e2.salary) from emp e2 where e2.dept_id = e1.dept_id);
--11 
select dept.*,count(emp.id) from dept,emp group by emp.dept_id where emp.dept_id = dept.id;
select d.*,(select count(e.id) from emp e where e.dept = d.id) from dept d;
--12
select s.name,s.id,c.name from stu s,class c,stu_cla cs where cs.sid=s.id and cs.cid=c.id; 
```



## 事务

###  ACID四特性

#### 原子性

要么同时成功，要么同时失败，有undo log日志来保证

#### 一致性

由业务代码正确逻辑保证，比如负数、除数为零等操作时报错回滚

#### 隔离性

```mysql
-- 查看事务隔离级别
select @@transaction_isolation;
-- 设置事务隔离级别
set [session|global] transaction isolation level {read uncommitted | read committed | repeatable read | serializable}
```

- read uncommitted 读未提交：有脏读问题，未提交的数据可能是脏数据，随时可能被撤回
- read committed 读已提交：（oracle默认）有不可重复读问题，每一次读得的数据可能会变
- repeatable read 可重复读：（MySQL默认）有幻读问题，每一次读得的数据都一样，在commit之前不实时更新（查报表时用的，即查同一数据纬度数据）
- serializable 串行：解决以上所有问题，同一个数据的读写不能在多个事务中同时执行

#### 持久性

一旦提交了数据，对数据库的改变应该是永久的