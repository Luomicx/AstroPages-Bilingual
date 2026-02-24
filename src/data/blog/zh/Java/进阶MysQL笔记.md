---
title: Java学习 - MySQL 深度学习
pubDatetime: 2025-08-05T22:20:40Z
description: MySQL 深度学习笔记
featured: false
draft: false
tags:
  - Java
---

> 本笔记记录了本人在学习 MySQL 的过程中的学习笔记，由本人学习之后提取视频内容而做。
>
> 【黑马程序员 MySQL 数据库入门到精通，从 mysql 安装到 mysql 高级、mysql 优化全囊括】 <https://www.bilibili.com/video/BV1Kr4y1i7ru>

# MySQL 基础篇

## 通用语法及分类

- DDL: 数据定义语言，用来定义数据库对象（数据库、表、字段）
- DML: 数据操作语言，用来对数据库表中的数据进行增删改
- DQL: 数据查询语言，用来查询数据库中表的记录
- DCL: 数据控制语言，用来创建数据库用户、控制数据库的控制权限

### DDL（数据定义语言）

数据定义语言

#### 数据库操作

查询所有数据库：
`SHOW DATABASES;`
查询当前数据库：
`SELECT DATABASE();`
创建数据库：
`CREATE DATABASE [ IF NOT EXISTS ] 数据库名 [ DEFAULT CHARSET 字符集] [COLLATE 排序规则 ];`
删除数据库：
`DROP DATABASE [ IF EXISTS ] 数据库名;`
使用数据库：
`USE 数据库名;`

##### 注意事项

- UTF8 字符集长度为 3 字节，有些符号占 4 字节，所以推荐用 utf8mb4 字符集

#### 表操作

查询当前数据库所有表：
`SHOW TABLES;`
查询表结构：
`DESC 表名;`
查询指定表的建表语句：
`SHOW CREATE TABLE 表名;`

创建表：

```
CREATE TABLE 表名(
 字段1 字段1类型 [COMMENT 字段1注释],
 字段2 字段2类型 [COMMENT 字段2注释],
 字段3 字段3类型 [COMMENT 字段3注释],
 ...
 字段n 字段n类型 [COMMENT 字段n注释]
)[ COMMENT 表注释 ];
```

**最后一个字段后面没有逗号**

添加字段：
`ALTER TABLE 表名 ADD 字段名 类型(长度) [COMMENT 注释] [约束];`
例：`ALTER TABLE emp ADD nickname varchar(20) COMMENT '昵称';`

修改数据类型：
`ALTER TABLE 表名 MODIFY 字段名 新数据类型(长度);`
修改字段名和字段类型：
`ALTER TABLE 表名 CHANGE 旧字段名 新字段名 类型(长度) [COMMENT 注释] [约束];`
例：将 emp 表的 nickname 字段修改为 username，类型为 varchar(30)
`ALTER TABLE emp CHANGE nickname username varchar(30) COMMENT '昵称';`

删除字段：
`ALTER TABLE 表名 DROP 字段名;`

修改表名：
`ALTER TABLE 表名 RENAME TO 新表名`

删除表：
`DROP TABLE [IF EXISTS] 表名;`
删除表，并重新创建该表：
`TRUNCATE TABLE 表名;`

### DML（数据操作语言）

#### 添加数据

指定字段：
`INSERT INTO 表名 (字段名1, 字段名2, ...) VALUES (值1, 值2, ...);`
全部字段：
`INSERT INTO 表名 VALUES (值1, 值2, ...);`

批量添加数据：
`INSERT INTO 表名 (字段名1, 字段名2, ...) VALUES (值1, 值2, ...), (值1, 值2, ...), (值1, 值2, ...);`
`INSERT INTO 表名 VALUES (值1, 值2, ...), (值1, 值2, ...), (值1, 值2, ...);`

##### 注意事项

- 字符串和日期类型数据应该包含在引号中
- 插入的数据大小应该在字段的规定范围内

#### 更新和删除数据

修改数据：
`UPDATE 表名 SET 字段名1 = 值1, 字段名2 = 值2, ... [ WHERE 条件 ];`
例：
`UPDATE emp SET name = 'Jack' WHERE id = 1;`

删除数据：
`DELETE FROM 表名 [ WHERE 条件 ];`

### DQL（数据查询语言）

语法：

```
SELECT
 字段列表
FROM
 表名字段
WHERE
 条件列表
GROUP BY
 分组字段列表
HAVING
 分组后的条件列表
ORDER BY
 排序字段列表
LIMIT
 分页参数
```

#### 基础查询

查询多个字段：
`SELECT 字段1, 字段2, 字段3, ... FROM 表名;`
`SELECT * FROM 表名;`

设置别名：
`SELECT 字段1 [ AS 别名1 ], 字段2 [ AS 别名2 ], 字段3 [ AS 别名3 ], ... FROM 表名;`
`SELECT 字段1 [ 别名1 ], 字段2 [ 别名2 ], 字段3 [ 别名3 ], ... FROM 表名;`

去除重复记录：
`SELECT DISTINCT 字段列表 FROM 表名;`

转义：
`SELECT * FROM 表名 WHERE name LIKE '/_张三' ESCAPE '/'`
/ 之后的\_不作为通配符

#### 条件查询

语法：
`SELECT 字段列表 FROM 表名 WHERE 条件列表;`

条件：

| 比较运算符      | 功能                                        |
| --------------- | ------------------------------------------- |
| >               | 大于                                        |
| >=              | 大于等于                                    |
| <               | 小于                                        |
| <=              | 小于等于                                    |
| =               | 等于                                        |
| <> 或 !=        | 不等于                                      |
| BETWEEN … AND … | 在某个范围内（含最小、最大值）              |
| IN(…)           | 在 in 之后的列表中的值，多选一              |
| LIKE 占位符     | 模糊匹配（\_匹配单个字符，%匹配任意个字符） |
| IS NULL         | 是 NULL                                     |

| 逻辑运算符 | 功能                         |
| ---------- | ---------------------------- |
| AND 或 &&  | 并且（多个条件同时成立）     |
| OR 或 \|\| | 或者（多个条件任意一个成立） |
| NOT 或 !   | 非，不是                     |

例子：

```
-- 年龄等于30
select * from employee where age = 30;
-- 年龄小于30
select * from employee where age < 30;
-- 小于等于
select * from employee where age <= 30;
-- 没有身份证
select * from employee where idcard is null or idcard = '';
-- 有身份证
select * from employee where idcard;
select * from employee where idcard is not null;
-- 不等于
select * from employee where age != 30;
-- 年龄在20到30之间
select * from employee where age between 20 and 30;
select * from employee where age >= 20 and age <= 30;
-- 下面语句不报错，但查不到任何信息
select * from employee where age between 30 and 20;
-- 性别为女且年龄小于30
select * from employee where age < 30 and gender = '女';
-- 年龄等于25或30或35
select * from employee where age = 25 or age = 30 or age = 35;
select * from employee where age in (25, 30, 35);
-- 姓名为两个字
select * from employee where name like '__';
-- 身份证最后为X
select * from employee where idcard like '%X';
```

#### 聚合查询（聚合函数）

常见聚合函数：

| 函数  | 功能     |
| ----- | -------- |
| count | 统计数量 |
| max   | 最大值   |
| min   | 最小值   |
| avg   | 平均值   |
| sum   | 求和     |

语法：
`SELECT 聚合函数(字段列表) FROM 表名;`
例：
`SELECT count(id) from employee where workaddress = "广东省";`

#### 分组查询

语法：
`SELECT 字段列表 FROM 表名 [ WHERE 条件 ] GROUP BY 分组字段名 [ HAVING 分组后的过滤条件 ];`

where 和 having 的区别：

- 执行时机不同：where 是分组之前进行过滤，不满足 where 条件不参与分组；having 是分组后对结果进行过滤。
- 判断条件不同：where 不能对聚合函数进行判断，而 having 可以。

例子：

```
-- 根据性别分组，统计男性和女性数量（只显示分组数量，不显示哪个是男哪个是女）
select count(*) from employee group by gender;
-- 根据性别分组，统计男性和女性数量
select gender, count(*) from employee group by gender;
-- 根据性别分组，统计男性和女性的平均年龄
select gender, avg(age) from employee group by gender;
-- 年龄小于45，并根据工作地址分组
select workaddress, count(*) from employee where age < 45 group by workaddress;
-- 年龄小于45，并根据工作地址分组，获取员工数量大于等于3的工作地址
select workaddress, count(*) address_count from employee where age < 45 group by workaddress having address_count >= 3;
```

##### 注意事项

- 执行顺序：where > 聚合函数 > having
- 分组之后，查询的字段一般为聚合函数和分组字段，查询其他字段无任何意义

#### 排序查询

语法：
`SELECT 字段列表 FROM 表名 ORDER BY 字段1 排序方式1, 字段2 排序方式2;`

排序方式：

- ASC: 升序（默认）
- DESC: 降序

例子：

```
-- 根据年龄升序排序
SELECT * FROM employee ORDER BY age ASC;
SELECT * FROM employee ORDER BY age;
-- 两字段排序，根据年龄升序排序，入职时间降序排序
SELECT * FROM employee ORDER BY age ASC, entrydate DESC;
```

##### 注意事项

如果是多字段排序，当第一个字段值相同时，才会根据第二个字段进行排序

#### 分页查询

语法：
`SELECT 字段列表 FROM 表名 LIMIT 起始索引, 查询记录数;`

例子：

```
-- 查询第一页数据，展示10条
SELECT * FROM employee LIMIT 0, 10;
-- 查询第二页
SELECT * FROM employee LIMIT 10, 10;
```

##### 注意事项

- 起始索引从 0 开始，起始索引 = （查询页码 - 1） \* 每页显示记录数
- 分页查询是数据库的方言，不同数据库有不同实现，MySQL 是 LIMIT
- 如果查询的是第一页数据，起始索引可以省略，直接简写 LIMIT 10

#### DQL 执行顺序

FROM -> WHERE -> GROUP BY -> SELECT -> ORDER BY -> LIMIT

### DCL

#### 管理用户

查询用户：

```
USE mysql;
SELECT * FROM user;
```

创建用户:
`CREATE USER '用户名'@'主机名' IDENTIFIED BY '密码';`

修改用户密码：
`ALTER USER '用户名'@'主机名' IDENTIFIED WITH mysql_native_password BY '新密码';`

删除用户：
`DROP USER '用户名'@'主机名';`

例子：

```
-- 创建用户test，只能在当前主机localhost访问
create user 'test'@'localhost' identified by '123456';
-- 创建用户test，能在任意主机访问
create user 'test'@'%' identified by '123456';
create user 'test' identified by '123456';
-- 修改密码
alter user 'test'@'localhost' identified with mysql_native_password by '1234';
-- 删除用户
drop user 'test'@'localhost';
```

##### 注意事项

- 主机名可以使用 % 通配

#### 权限控制

常用权限：

| 权限                | 说明               |
| ------------------- | ------------------ |
| ALL, ALL PRIVILEGES | 所有权限           |
| SELECT              | 查询数据           |
| INSERT              | 插入数据           |
| UPDATE              | 修改数据           |
| DELETE              | 删除数据           |
| ALTER               | 修改表             |
| DROP                | 删除数据库/表/视图 |
| CREATE              | 创建数据库/表      |

更多权限请看[权限一览表](https://jimhackking.github.io/运维/MySQL学习笔记/#权限一览表)

查询权限：
`SHOW GRANTS FOR '用户名'@'主机名';`

授予权限：
`GRANT 权限列表 ON 数据库名.表名 TO '用户名'@'主机名';`

撤销权限：
`REVOKE 权限列表 ON 数据库名.表名 FROM '用户名'@'主机名';`

##### 注意事项

- 多个权限用逗号分隔
- 授权时，数据库名和表名可以用 \* 进行通配，代表所有

## 函数

- 字符串函数
- 数值函数
- 日期函数
- 流程函数

### 字符串函数

常用函数：

| 函数                             | 功能                                                            |
| -------------------------------- | --------------------------------------------------------------- |
| CONCAT(s1, s2, …, sn)            | 字符串拼接，将 s1, s2, …, sn 拼接成一个字符串                   |
| LOWER(str)                       | 将字符串全部转为小写                                            |
| UPPER(str)                       | 将字符串全部转为大写                                            |
| LPAD(str, n, pad)                | 左填充，用字符串 pad 对 str 的左边进行填充，达到 n 个字符串长度 |
| RPAD(str, n, pad)                | 右填充，用字符串 pad 对 str 的右边进行填充，达到 n 个字符串长度 |
| TRIM(str)                        | 去掉字符串头部和尾部的空格                                      |
| SUBSTRING(str, start, len)       | 返回从字符串 str 从 start 位置起的 len 个长度的字符串           |
| REPLACE(column, source, replace) | 替换字符串                                                      |

使用示例：

```
-- 拼接
SELECT CONCAT('Hello', 'World');
-- 小写
SELECT LOWER('Hello');
-- 大写
SELECT UPPER('Hello');
-- 左填充
SELECT LPAD('01', 5, '-');
-- 右填充
SELECT RPAD('01', 5, '-');
-- 去除空格
SELECT TRIM(' Hello World ');
-- 切片（起始索引为1）
SELECT SUBSTRING('Hello World', 1, 5);
```

### 数值函数

常见函数：

| 函数        | 功能                                 |
| ----------- | ------------------------------------ |
| CEIL(x)     | 向上取整                             |
| FLOOR(x)    | 向下取整                             |
| MOD(x, y)   | 返回 x/y 的模                        |
| RAND()      | 返回 0~1 内的随机数                  |
| ROUND(x, y) | 求参数 x 的四舍五入值，保留 y 位小数 |

### 日期函数

常用函数：

| 函数                               | 功能                                                |
| ---------------------------------- | --------------------------------------------------- |
| CURDATE()                          | 返回当前日期                                        |
| CURTIME()                          | 返回当前时间                                        |
| NOW()                              | 返回当前日期和时间                                  |
| YEAR(date)                         | 获取指定 date 的年份                                |
| MONTH(date)                        | 获取指定 date 的月份                                |
| DAY(date)                          | 获取指定 date 的日期                                |
| DATE_ADD(date, INTERVAL expr type) | 返回一个日期/时间值加上一个时间间隔 expr 后的时间值 |
| DATEDIFF(date1, date2)             | 返回起始时间 date1 和结束时间 date2 之间的天数      |

例子：

```
-- DATE_ADD
SELECT DATE_ADD(NOW(), INTERVAL 70 YEAR);
```

### 流程函数

常用函数：

| 函数                                                             | 功能                                                          |
| ---------------------------------------------------------------- | ------------------------------------------------------------- |
| IF(value, t, f)                                                  | 如果 value 为 true，则返回 t，否则返回 f                      |
| IFNULL(value1, value2)                                           | 如果 value1 不为空，返回 value1，否则返回 value2              |
| CASE WHEN [ val1 ] THEN [ res1 ] … ELSE [ default ] END          | 如果 val1 为 true，返回 res1，… 否则返回 default 默认值       |
| CASE [ expr ] WHEN [ val1 ] THEN [ res1 ] … ELSE [ default ] END | 如果 expr 的值等于 val1，返回 res1，… 否则返回 default 默认值 |

例子：

```
select
 name,
 (case when age > 30 then '中年' else '青年' end)
from employee;
select
 name,
 (case workaddress when '北京市' then '一线城市' when '上海市' then '一线城市' else '二线城市' end) as '工作地址'
from employee;
```

## 约束

分类：

| 约束                     | 描述                                                     | 关键字      |
| ------------------------ | -------------------------------------------------------- | ----------- |
| 非空约束                 | 限制该字段的数据不能为 null                              | NOT NULL    |
| 唯一约束                 | 保证该字段的所有数据都是唯一、不重复的                   | UNIQUE      |
| 主键约束                 | 主键是一行数据的唯一标识，要求非空且唯一                 | PRIMARY KEY |
| 默认约束                 | 保存数据时，如果未指定该字段的值，则采用默认值           | DEFAULT     |
| 检查约束（8.0.1 版本后） | 保证字段值满足某一个条件                                 | CHECK       |
| 外键约束                 | 用来让两张图的数据之间建立连接，保证数据的一致性和完整性 | FOREIGN KEY |

约束是作用于表中字段上的，可以再创建表/修改表的时候添加约束。

### 常用约束

| 约束条件 | 关键字         |
| -------- | -------------- |
| 主键     | PRIMARY KEY    |
| 自动增长 | AUTO_INCREMENT |
| 不为空   | NOT NULL       |
| 唯一     | UNIQUE         |
| 逻辑条件 | CHECK          |
| 默认值   | DEFAULT        |

例子：

```
create table user(
 id int primary key auto_increment,
 name varchar(10) not null unique,
 age int check(age > 0 and age < 120),
 status char(1) default '1',
 gender char(1)
);
```

### 外键约束

添加外键：

```
CREATE TABLE 表名(
 字段名 字段类型,
 ...
 [CONSTRAINT] [外键名称] FOREIGN KEY(外键字段名) REFERENCES 主表(主表列名)
);
ALTER TABLE 表名 ADD CONSTRAINT 外键名称 FOREIGN KEY (外键字段名) REFERENCES 主表(主表列名);

-- 例子
alter table emp add constraint fk_emp_dept_id foreign key(dept_id) references dept(id);
```

删除外键：
`ALTER TABLE 表名 DROP FOREIGN KEY 外键名;`

#### 删除/更新行为

| 行为        | 说明                                                                                                                    |
| ----------- | ----------------------------------------------------------------------------------------------------------------------- |
| NO ACTION   | 当在父表中删除/更新对应记录时，首先检查该记录是否有对应外键，如果有则不允许删除/更新（与 RESTRICT 一致）                |
| RESTRICT    | 当在父表中删除/更新对应记录时，首先检查该记录是否有对应外键，如果有则不允许删除/更新（与 NO ACTION 一致）               |
| CASCADE     | 当在父表中删除/更新对应记录时，首先检查该记录是否有对应外键，如果有则也删除/更新外键在子表中的记录                      |
| SET NULL    | 当在父表中删除/更新对应记录时，首先检查该记录是否有对应外键，如果有则设置子表中该外键值为 null（要求该外键允许为 null） |
| SET DEFAULT | 父表有变更时，子表将外键设为一个默认值（Innodb 不支持）                                                                 |

更改删除/更新行为：
`ALTER TABLE 表名 ADD CONSTRAINT 外键名称 FOREIGN KEY (外键字段) REFERENCES 主表名(主表字段名) ON UPDATE 行为 ON DELETE 行为;`

## 多表查询

### 多表关系

- 一对多（多对一）
- 多对多
- 一对一

#### 一对多

案例：部门与员工
关系：一个部门对应多个员工，一个员工对应一个部门
实现：在多的一方建立外键，指向一的一方的主键

#### 多对多

案例：学生与课程
关系：一个学生可以选多门课程，一门课程也可以供多个学生选修
实现：建立第三张中间表，中间表至少包含两个外键，分别关联两方主键

#### 一对一

案例：用户与用户详情
关系：一对一关系，多用于单表拆分，将一张表的基础字段放在一张表中，其他详情字段放在另一张表中，以提升操作效率
实现：在任意一方加入外键，关联另外一方的主键，并且设置外键为唯一的（UNIQUE）

### 查询

合并查询（笛卡尔积，会展示所有组合结果）：
`select * from employee, dept;`

> 笛卡尔积：两个集合 A 集合和 B 集合的所有组合情况（在多表查询时，需要消除无效的笛卡尔积）

消除无效笛卡尔积：
`select * from employee, dept where employee.dept = dept.id;`

### 内连接查询

内连接查询的是两张表交集的部分

隐式内连接：
`SELECT 字段列表 FROM 表1, 表2 WHERE 条件 ...;`

显式内连接：
`SELECT 字段列表 FROM 表1 [ INNER ] JOIN 表2 ON 连接条件 ...;`

显式性能比隐式高

例子：

```
-- 查询员工姓名，及关联的部门的名称
-- 隐式
select e.name, d.name from employee as e, dept as d where e.dept = d.id;
-- 显式
select e.name, d.name from employee as e inner join dept as d on e.dept = d.id;
```

### 外连接查询

左外连接：
查询左表所有数据，以及两张表交集部分数据
`SELECT 字段列表 FROM 表1 LEFT [ OUTER ] JOIN 表2 ON 条件 ...;`
相当于查询表 1 的所有数据，包含表 1 和表 2 交集部分数据

右外连接：
查询右表所有数据，以及两张表交集部分数据
`SELECT 字段列表 FROM 表1 RIGHT [ OUTER ] JOIN 表2 ON 条件 ...;`

例子：

```
-- 左
select e.*, d.name from employee as e left outer join dept as d on e.dept = d.id;
select d.name, e.* from dept d left outer join emp e on e.dept = d.id;  -- 这条语句与下面的语句效果一样
-- 右
select d.name, e.* from employee as e right outer join dept as d on e.dept = d.id;
```

左连接可以查询到没有 dept 的 employee，右连接可以查询到没有 employee 的 dept

### 自连接查询

当前表与自身的连接查询，自连接必须使用表别名

语法：
`SELECT 字段列表 FROM 表A 别名A JOIN 表A 别名B ON 条件 ...;`

自连接查询，可以是内连接查询，也可以是外连接查询

例子：

```
-- 查询员工及其所属领导的名字
select a.name, b.name from employee a, employee b where a.manager = b.id;
-- 没有领导的也查询出来
select a.name, b.name from employee a left join employee b on a.manager = b.id;
```

### 联合查询 union, union all

把多次查询的结果合并，形成一个新的查询集

语法：

```
SELECT 字段列表 FROM 表A ...
UNION [ALL]
SELECT 字段列表 FROM 表B ...
```

#### 注意事项

- UNION ALL 会有重复结果，UNION 不会
- 联合查询比使用 or 效率高，不会使索引失效

### 子查询

SQL 语句中嵌套 SELECT 语句，称谓嵌套查询，又称子查询。
`SELECT * FROM t1 WHERE column1 = ( SELECT column1 FROM t2);`
**子查询外部的语句可以是 INSERT / UPDATE / DELETE / SELECT 的任何一个**

根据子查询结果可以分为：

- 标量子查询（子查询结果为单个值）
- 列子查询（子查询结果为一列）
- 行子查询（子查询结果为一行）
- 表子查询（子查询结果为多行多列）

根据子查询位置可分为：

- WHERE 之后
- FROM 之后
- SELECT 之后

#### 标量子查询

子查询返回的结果是单个值（数字、字符串、日期等）。
常用操作符：- < > > >= < <=

例子：

```
-- 查询销售部所有员工
select id from dept where name = '销售部';
-- 根据销售部部门ID，查询员工信息
select * from employee where dept = 4;
-- 合并（子查询）
select * from employee where dept = (select id from dept where name = '销售部');

-- 查询xxx入职之后的员工信息
select * from employee where entrydate > (select entrydate from employee where name = 'xxx');
```

#### 列子查询

返回的结果是一列（可以是多行）。

常用操作符：

| 操作符 | 描述                                        |
| ------ | ------------------------------------------- |
| IN     | 在指定的集合范围内，多选一                  |
| NOT IN | 不在指定的集合范围内                        |
| ANY    | 子查询返回列表中，有任意一个满足即可        |
| SOME   | 与 ANY 等同，使用 SOME 的地方都可以使用 ANY |
| ALL    | 子查询返回列表的所有值都必须满足            |

例子：

```
-- 查询销售部和市场部的所有员工信息
select * from employee where dept in (select id from dept where name = '销售部' or name = '市场部');
-- 查询比财务部所有人工资都高的员工信息
select * from employee where salary > all(select salary from employee where dept = (select id from dept where name = '财务部'));
-- 查询比研发部任意一人工资高的员工信息
select * from employee where salary > any (select salary from employee where dept = (select id from dept where name = '研发部'));
```

#### 行子查询

返回的结果是一行（可以是多列）。
常用操作符：=, <, >, IN, NOT IN

例子：

```
-- 查询与xxx的薪资及直属领导相同的员工信息
select * from employee where (salary, manager) = (12500, 1);
select * from employee where (salary, manager) = (select salary, manager from employee where name = 'xxx');
```

#### 表子查询

返回的结果是多行多列
常用操作符：IN

例子：

```
-- 查询与xxx1，xxx2的职位和薪资相同的员工
select * from employee where (job, salary) in (select job, salary from employee where name = 'xxx1' or name = 'xxx2');
-- 查询入职日期是2006-01-01之后的员工，及其部门信息
select e.*, d.* from (select * from employee where entrydate > '2006-01-01') as e left join dept as d on e.dept = d.id;
```

## 事务

事务是一组操作的集合，事务会把所有操作作为一个整体一起向系统提交或撤销操作请求，即这些操作要么同时成功，要么同时失败。

基本操作：

```
-- 1. 查询张三账户余额
select * from account where name = '张三';
-- 2. 将张三账户余额-1000
update account set money = money - 1000 where name = '张三';
-- 此语句出错后张三钱减少但是李四钱没有增加
模拟sql语句错误
-- 3. 将李四账户余额+1000
update account set money = money + 1000 where name = '李四';

-- 查看事务提交方式
SELECT @@AUTOCOMMIT;
-- 设置事务提交方式，1为自动提交，0为手动提交，该设置只对当前会话有效
SET @@AUTOCOMMIT = 0;
-- 提交事务
COMMIT;
-- 回滚事务
ROLLBACK;

-- 设置手动提交后上面代码改为：
select * from account where name = '张三';
update account set money = money - 1000 where name = '张三';
update account set money = money + 1000 where name = '李四';
commit;
```

操作方式二：

开启事务：
`START TRANSACTION 或 BEGIN TRANSACTION;`
提交事务：
`COMMIT;`
回滚事务：
`ROLLBACK;`

操作实例：

```
start transaction;
select * from account where name = '张三';
update account set money = money - 1000 where name = '张三';
update account set money = money + 1000 where name = '李四';
commit;
```

### 四大特性 ACID

- 原子性(Atomicity)：事务是不可分割的最小操作单元，要么全部成功，要么全部失败
- 一致性(Consistency)：事务完成时，必须使所有数据都保持一致状态
- 隔离性(Isolation)：数据库系统提供的隔离机制，保证事务在不受外部并发操作影响的独立环境下运行
- 持久性(Durability)：事务一旦提交或回滚，它对数据库中的数据的改变就是永久的

### 并发事务

| 问题       | 描述                                                                                   |
| ---------- | -------------------------------------------------------------------------------------- |
| 脏读       | 一个事务读到另一个事务还没提交的数据                                                   |
| 不可重复读 | 一个事务先后读取同一条记录，但两次读取的数据不同                                       |
| 幻读       | 一个事务按照条件查询数据时，没有对应的数据行，但是再插入数据时，又发现这行数据已经存在 |

> 这三个问题的详细演示：<https://www.bilibili.com/video/BV1Kr4y1i7ru?p=55cd>

并发事务隔离级别：

| 隔离级别              | 脏读 | 不可重复读 | 幻读 |
| --------------------- | ---- | ---------- | ---- |
| Read uncommitted      | √    | √          | √    |
| Read committed        | ×    | √          | √    |
| Repeatable Read(默认) | ×    | ×          | √    |
| Serializable          | ×    | ×          | ×    |

- √ 表示在当前隔离级别下该问题会出现
- Serializable 性能最低；Read uncommitted 性能最高，数据安全性最差

查看事务隔离级别：
`SELECT @@TRANSACTION_ISOLATION;`
设置事务隔离级别：
`SET [ SESSION | GLOBAL ] TRANSACTION ISOLATION LEVEL {READ UNCOMMITTED | READ COMMITTED | REPEATABLE READ | SERIALIZABLE };`
SESSION 是会话级别，表示只针对当前会话有效，GLOBAL 表示对所有会话有效

# MySQL 进阶篇

![image-20250720233738136](http://img.sut.qzz.io/img/20250720233738280.webp)

## 存储引擎

- MySQL 默认的存储引擎是 **InnoDB**

### MyISAM

- MySQL 早期的存储引擎
- 不支持事务，不支持外键
- 支持表锁，不支持行锁
- 访问速度快

### Memory

- 存储在内存中，收到硬件问题，或断电问题的影响，只能将这些表作为临时表来进行使用
- 内存存放 **hash** 索引

![image-20250720230330428](http://img.sut.qzz.io/img/20250720230330619.webp)

- **Q1：InnoDB 和 MyISAM 之间的区别？**
  - InnoDB 支持事务，MyISAM 不支持
  - innoDB 支持行锁，而 MyISAM 支持表锁
  - InnoDB 支持外键，而 MyISAM 不支持

### 存储引擎选择

![image-20250720230627951](http://img.sut.qzz.io/img/20250720230628090.webp)

## 索引

### 索引概述

- 演示（索引创建了一个类似的 **排序二叉树索引**，实际上存储的结构并不是二叉树结构）

![image-20250720233429074](http://img.sut.qzz.io/img/20250720233429244.webp)

- 优缺点：
  - 优点：提高数据检索的效率，降低数据库 IO 时间，降低数据排序成本，降低 CPU 的消耗
  - 缺点：占用磁盘空间，索引大大提高查询效率，降低 INSERT，UPDATE，DELETE 的效率

### 索引结构

![image-20250720233807340](http://img.sut.qzz.io/img/20250720233807469.webp)

- InnoDB， MyISAM， Memory 存储引擎都支持 B+索引，一般考察索引都是 **B+树索引**

![image-20250720234205568](http://img.sut.qzz.io/img/20250720234205700.webp)

### B-Tree

- 五阶的 B 树，存储 4 个 Key，5 个指针
- 根据动态网站来进行理解即可

### B+Tree

- 叶子节点是一个单向链表

![image-20250720234835748](http://img.sut.qzz.io/img/20250720234835889.webp)

### B+树索引

![image-20250720234857624](http://img.sut.qzz.io/img/20250720234857778.webp)

### 哈希索引

![image-20250720235102741](http://img.sut.qzz.io/img/20250720235102903.webp)

- **Q2：**

![image-20250720235408595](http://img.sut.qzz.io/img/20250720235408729.webp)

### 索引分类

- 主键索引 (Primary) → Clustered Index，唯一定位整行
- 唯一索引 (Unique) → 非空且唯一，可做逻辑主键
- 普通/二级索引 (Secondary) → 无任何约束，只为得快
- 前缀索引 (Prefix) → Index(col(20))，大字段省空间
- 联合/复合索引 (Composite) → (a, b, c) 遵循最左前缀
- 覆盖/索引覆盖(Covering) → SELECT 列都在索引树上，不回表
  -------- 真正决定“能不能插入重复值”、“回不回表”就看这类索引。

#### 聚集索引 & 二级索引

这两个索引的分类的根据是按照存储方式来说的为 **InnoDB 特有**

- 聚集索引将索引和存储数据放在一起，即一个索引节点存储这一行的数据和索引
- 二级索引的叶子节点，挂的是聚集索引的 ID

![image-20250721000015608](http://img.sut.qzz.io/img/20250721000015777.webp)

- **回表查询：** 先通过二级索引找到聚集索引的 key，然后通过聚集索引来寻找对应的行数据，这称为回表查询

### 索引语法

**创建索引**：`CREATE [UNIQUE FULLTEXT]INDEX index_name ON table_name (index_col_name,...);`

**查看索引**：`SHOW INDEX FROM table_name;`

**删除索引**：`DROP INDEX index_name ON table_name;`

```sql
create index  idx_pro_age_sta on tb_user(profession, age, status) -- 联合索引
```

### SQL 性能优化

- 优化是主要对我们的查询语句进行优化，一般采用索引来进行相关的优化

- **SQL 执行效率：** 通过 `SHOW [session|global] STATUS LIKE 'Com_______'` 来查询四个语句的执行次数

- **慢查询日志：** 记录了执行时间超过指定时间的所有 SQL 语句的日志

  - MySQL 慢查询日志默认未开启，可以通过配置文件开启

  - ```ini
    # 开启MySQL慢日志查询开关
    slow_query_log = 1
    # 设置慢日志的时间为2秒，SQL语句执行时间超过2秒，就会视为慢查询
    long_query_time = 2
    ```

  - 查看 `var/lib/mysql/localhost-slow.log` 文件查询慢日志信息

- **profile 详情：** `show profiles` 可以帮助我们了解时间都耗费在哪里。通过 `select @@have_profiling` 查询是否开启

  - `show profile for query query_id` 可以看到一个详细的查询的耗费详情

- **explain 执行计划：** 通过 explain 执行计划获取 MySQL 是如何执行语句的，也就是语句的执行计划，只要在查询语句前加一个 **explain**, 例如：`explain select * from tb_user;`

### 索引的使用

#### 索引失效

##### 最左索引法则

当使用联合索引时，最左边的索引字段（列）必须存在，否则该联合索引失效

- 例子：idx_user_pro_age_status 这个索引，查询必须从 profession 开始，如果跳跃查询，会使得索引部分失效

```sql
-- 这时会部分失效
select * from tb_user where profession = '软件工程' and status = '0';
-- 全部失效，因为最左索引法则失效
select * from tb_user where age = 19 and status = '0';
```

##### 范围查询

联合索引中，出现范围查询（<，>），**范围查询右侧的列索引失效**。

##### 索引列运算

不要在索引上进行 **计算操作**，索引会失效。

##### 字符单引号

如果查询字符串时，**不加单引号**，也会造成索引失效。

##### 模糊查询

如果仅仅是 **尾部** 模糊匹配，索引不会失效，如果是 **头部** 进行模糊查询，索引会失效。

##### or 连接的条件

用 or 分割开的条件，如果 or 前的条件中的列有索引，而 **后面的列中没有索引**，那么 **涉及的索引都不会被用到**。

```sql
explain select from tb_user where id = 10 or age = 23;
explain select from tb_user where phone ='17799990017'or age = 23;
```

##### 数据分布影响

如果 MySQL 评估使用索引比全表更慢，则不使用索引。

#### SQL 提示

SQL 提示，是优化数据库的一个重要手段，简单来说，就是在 SQL 语句中加入一些人为的提示来达到优化操作的目的。
**use index**: 建议使用
`explain select * from tb user use index(idx user pro) where profession ='软件工程'；`
**ignore index**: 忽略使用
`explain select * from tb_user ignore index(idx_user_pro) where profession = 软件工程'；`
**force index:** 强制使用
`explain select * from tb_user force index(idx_user_pro) where profession = 软件工程'；`

#### 覆盖索引

尽量使用覆盖索引（查询使用了索引，并且需要返回的列，在该索引中已经全部能够找到），减少 select\*。

```sql
explain select id,profession from tb user where profession ='软件工程' and age 31 and status ='0';
explain select id,profession,age,status from tb user where profession = '软件工程' and age =31 and status ='0';
explain select id,profession,age,status,name from tb user where profession = '软件工程' and age 31 and status ='0';
explain select from tb user where profession = '软件工程' and age =31 and status='0';
```

**using index condition**: 查找使用了索引，但是需要回表查询数据
**using where; using index**: 查找使用了索引，但是需要的数据都在索引列中能找到，所以不需要回表查询数据

#### 前缀索引

当字段类型为字符串(varchar, text 等)时，有时候需要索引很长的字符串，这会让索引变得很大，查询时，浪费大量的磁盘 IO, 影响查询效率。此时可以只将字符串的一部分前缀，建立索引，这样可以大大节约索引空间，从而提高索引效率。

- 语法：

```sql
create index idx_xxxx on table_name(column(n));
```

- 前缀长度：可以根据索引的选择性来决定，而选择性是指不重复的索引值（基数）和数据表的记录总数的比值，索引选择- 性越高则查询效率越高
- 唯一索引的选择性是 1，这是最好的索引选择性，性能也是最好的。

```sql
select count(distinct substring(email, 1, 8))/count(*) from tb_user;
create index idx_email on tb_user(email(5));
```

#### 单列索引 & 联合索引

**单列索引**：即一个索引只包含单个列。
**联合索引**：即一个索引包含了多个列。

```sql
create unique index idx_user_phone_name on tb_user(phone, name)
```

在业务场景中，如果存在多个查询条件，考虑针对于查询字段建立索引时，建议建立 **联合索引**，而非 **单列索引**。

联合索引使用得当可以避免回表查询，性能较高。

#### 索引设计原则

- 针对于数据量较大，且查询比较频繁的表建立索引。
- 针对于常作为查询条件(where)、排序(order by)、分组(group by)操作的字段建立索引。
- 尽量选择区分度高的列作为索引，尽量建立唯一索引，区分度越高，使用索引的效率越高。
- 如果是字符串类型的字段，字段的长度较长，可以针对于字段的特点，建立前缀索引。
- 尽量使用联合索引，减少单列索引，查询时，联合索引很多时候可以覆盖索引，节省存储空间，避免回表，提高查询效率。
- 要控制索引的数量，索引并不是多多益善，索引越多，维护索引结构的代价也就越大，会影响增删改的效率。
- 如果索引列不能存储 NULL 值，请在创建表时使用 NOT NULL 约束它。当优化器知道每列是否包含 NULL 值时，它可以更好地确定哪个索引最有效地用于查询。

## SQL 优化

### 插入数据的优化

- insert 优化
  - 批量插入
  - 手动提交事务 `start transaction` `commit`
  - 主键顺序插入：主键顺序插入的效率高于乱序插入
- 大批量插入数据

如果一次性需要插入大批量数据，使用 insert 语句插入性能较低，此时可以使用 MySQL 数据库提供的 load 指令进行插入。操作如下：

```ini
#客户端连接服务端时，加上参数--local-infile
mysql --local-infile -u root-p
#设置全局参数local_infile为1，开启从本地加载文件导入数据的开关
set global local_infile 1;
#执行load指令将准备好的数据，加载到表结构中
load data local infile '/root/sql1.log' into table tb_user fields terminated by ',' lines terminated by '\n';
```

### 主键优化

#### 数据组织方式

在 InnoDB 存储引擎中，表数据都是根据主键顺序组织存放的，这种存储方式的表称为 **索引组织表**(index organized table IOT)。

![image-20250721221617432](http://img.sut.qzz.io/img/20250721221618162.webp)

**页分裂：** 页可以为空，也可以填充一半，也可以填充 100%。每个页包含了 2-N 行数据（如果一行数据多大，会行溢出），根据主键排列

1. 触发条件
   · insert 记录时，目标页已经被占用 ≥ 15/16（内部硬性阈值）。
   · 顺序插入(自增)：几乎不会分裂。
   · 随机高并发：极容易触发现象叫 **mid-page insertion**——插到中间位置，大量后面的记录会后移，导致页内拥挤。
2. 动作过程（16 KB → 8 KB + 8 KB）
   step1 InnoDB 新生成一个空页；
   step2 **原页** 后半部分记录(含索引指针)整体搬到新页；
   step3 在父层的 **非叶子** 节点上增加 **一个额外 key pointer** 指到新页；
   · 如果不能在下级页插入，会导致多次“级联”的分裂一路向上冒。
3. 副作用
   · 更多 IO：父层指针+新页+双页写入；
   · 空间碎片：页 8 KB 数据+8 KB 空洞，SHOW TABLE STATUS 看见 _Data_free_ 上升。

**页合并：** 当删除一行记录时，实际上记录并没有被物理删除，只是记录被标记(flaged)为删除并且它的空间变得允许被其他记录声明使用。当页中删除的记录达到 MERGE THRESHOLD(默认为页的 50%)，InnoDB 会开始寻找最靠近的页（前或后）看看是否可以将两个页合并以优化空间使用。

1. 触发条件
   · **删除** 或 **Purge(undo/旧版本清理)** 后，某叶节点的已用空间 < MERGE_THRESHOLD (50 %，可由 `innodb_page_size` 或 `MERGE_THRESHOLD` 系统变量调整)；
   · 且兄弟页也在同一层级；
   · 这俩页合并后空间仍 ≤ 15/16。
2. 动作过程
   step1 把一个页的剩余记录搬到“左/右”兄弟页；
   step2 把空页放进 **FREE LIST** 供后续重用；
   step3 在父页把指向空页的 key 删掉；
   · 同样可能导致级联合并，一路递归到根节点并被缩减树高。
3. 副作用
   · 合并需要 **加 X 锁** + **分裂锁(排他)**，在并发高(如大量 DELETE + INSERT)时，仍旧考验锁争抢。
   · 合并后页的 pin 会被缓存，对 buffer-pool 也是成本。
4. 避免&治理
   · 批量 DELETE 最好包事务、分批进行；
   · 对经常自增主键的表通过 **TRUNCATE** 而不是 DELETE \*;"> 来重载表；
   · 监控 `information_schema.innodb_metrics` 里的 **merge_failure/split** 两指标判断是否需要调参。

#### 主键设计原则

- 满足业务需求的情况下，尽量降低主键的长度。
- 插入数据时，尽量选择顺序插入，选择使用 AUTO INCREMENT 自增主键。
- 尽量不要使用 UUD 做主键或者是其他自然庄键，如身份证号。
- 业务操作时，避免对主键的修改。

### ORDER BY 优化

①.**Using filesort**: 通过表的索引或全表扫描，读取满足条件的数据行，然后在排序缓冲区 sort buffer 中完成排序操作，所有不是通过索引直接返回排序结果的排序都叫 FileSort 排序。
②.**Using index**: 通过有序索引顺序扫描直接返回有序数据，这种情况即为 using index, 不需要额外排序，操作效率高。

我们在优化时，尽量优化到 Using index 级别。

**具体操作：** 在我们需要排序的列进行创建联合索引的操作，这时我们的 SQL 的执行计划就会使用联合索引来进行 Using index 的级别进行排序。

**优化 2**：如果 order by 需要先升序后降序，可以重新创建一个对应的索引

```sql
#根据age,phone进行降序一个升序，一个降序
explain select id,age,phone from tb user order by age asc,phone desc;
#创建索引
create index idx user age_phone ad on tb user(age asc ,phone desc);
#根据age,phone进行降序一个升序，一个降序
explain select id,age,phone from tb_user order by age asc,phone desc;
```

**总结**

- 根据排序字段建立合适的索引，多字段排序时，也遵循最左前缀法则。
- 尽量使用覆盖索引。
- 多字段排序，一个升序一个降序，此时需要注意联合索引在创建时的规则(ASC/DESC)。
- 如果不可避免的出现 filesort,.大数据量排序时，可以适当增大排序缓冲区大小 sort_buffer_size(默认 256k)。

### GROUP BY 优化

- 在分组操作时，可以通过索引来提高效率。
- 分组操作时，索引的使用也是满足最左前缀法则的。

### LIMIT 优化

一般分页查询时，通过创建覆盖索引能够比较好地提高性能，可以通过覆盖索引加子查询形式进行优化。

### COUNT 优化

- MyISAM 引擎把一个表的总行数存在了磁盘上，因此执行 count(\*)的时候会直接返回这个数，效率很高；
- InnoDB 引擎就麻烦了，它执行 count( \* )的时候，需要把数据一行一行地从引擎里面读出来，然后累积计数。

**优化思路：** 自己计数即可，用 Redis 来优化。在 MySQL8.0 之后 count 会自动根据索引进行优化。

### UPDATE 优化

更新时尽量根据索引进行更新，根据索引进行更新的是行级锁，不影响其他行的更新，否则就是加表锁，会降低并发的性能。

## 视图 & 存储过程 & 触发器

![image-20250722141258301](http://img.sut.qzz.io/img/20250722141258749.webp)

### 视图

视图(View)是一种虚拟存在的表。视图中的数据并不在数据库中实际存在，行和列数据来自定义视图的查询中使用的表，并且是在使用视图时动态生成的。

通俗的讲，视图只保存了查询的 SQL 逻辑，不保存查询结果。所以我们在创建视图的时候，主要的工作就落在创建这条 SQL 查询语句上。

#### 语法

创建视图：
`CREATE [OR REPLACE] VIEW 视图名称(列名列表)】AS SELECT语句[WITH[CASCADED|LOCAL] CHECK OPTION]`

查询视图：
查看创建视图语句：`SHOW CRETE VIEW 视图名称;`
查看视图数据：`查看视图数据:SELECT*FROM 视图名称…;`

修改视图：
方式一：
`CREATE [OR REPLACE]VIEW 视图名称(列名列表)AS SELECT语句[WITH[CASCADEDLLOCAL] CHECK OPTION`
方式二：
`ALTER VEW 视图名称(列名列表)AS SELECT语句[WITH[CASCADED|LOCAL]CHECK OPTION]`

删除视图：
`DROP VIEW [IF EXISTS]视图名称[,视图名称]`

```sql
-- 创建视图
create or replace view stu_v_1 as select id, name from student where id <= 10;

-- 查询视图
show create view stu_v_1;
select * from stu_v_1;

-- 修改视图
create or replace view stu_v_1 as select id, name, no from student where id <= 10;
alter view stu_v_1 as select id, name from student where id <= 10;

-- 删除视图
drop view if exists stu_v_1;


create or replace view stu_v_1 as select id, name from student where id <= 20;

select * from stu_v_1;

insert into stu_v_1 values(6, 'Tom');
```

#### 检查选项

**视图的检查选项：**

当使用 WITH CHECK OPTION 子句创建视图时，MySOL 会通过视图检查正在更改的每个行，例如 插入，更新，删除，以使其符合视图的定义。MVSOL 允许基于另一个视图创建视图，它还会检查依赖视图中的规则以保持一致性。

为了确定检查的范围，mysql 提供了两个选项: CASCADED 和 LOCAL，默认值为 CASCADED。

**cascaded：** 在对创建时含有该字段的视图，插入数据时，该视图依赖的视图都会加上检查，需要 **所有条件** 都满足才能够插入成功。

**local：** 在对创建时含有该字段的视图，插入数据时，对于该视图依赖的视图中 **含有检查语句的条件** 进行检查判断。

#### 更新及作用

**视图的更新：**

要使视图可更新，视图中的行与基础表中的行之间 **必须存在一对一的关系**。

如果视图包含以下任何一项，则该 **视图不可更新**：

1. 聚合函数或窗口函数(SUM()、MIN()、MAX()、COUNT()等
2. DISTINCT
3. GROUP BY
4. HAVINGA
5. UNION 或者 UNION ALL

**作用：**

- **简单**
  视图不仅可以简化用户对数据的理解，也可以简化他们的操作。那些被经常使用的查询可以被定义为视图，从而使得用户不必为以后的操作每次指定全部的条件。
- **安全**
  数据库可以授权，但不能授权到数据库特定行和特定的列上。通过视图用户只能查询和修改他们所能见到的数据。
- **数据独立**
  视图可帮助用户屏蔽真实表结构变化带来的影响。

#### 案例

```sql
-- 1.为了保证数据库表的安全性，开发人员在操作tb_user表时，只能看到的用户的基本字段，屏蔽手机号和邮箱两个字段。
create view tb user view as select id,name,profession, age,gender,status,createtime from tb_user;
select *from tb user view;

-- 2.查询每个学生所选修的课程（三张表联查），这个功能在很多的业务中都有使用到，为了简化操作，定义一个视图。
create view tb_stu_course_view
select s.name student_name, s.no student_no, c.name course_name
from student s, stuent_course sc, course c
where s.id = sc.studentid and sc.courseid = c.id;

-- 以后每次只需要进行查询视图即可
select * from tb_stu_course_view;
```

### 存储过程

存储过程其实就类似 java，c 这种语言，这一部分可以通过文档快速学习，不懂的再回过头看视频。

存储过程是事先经过编译并存储在数据库中的一段 SQL 语句的集合，调用存储过程可以简化应用开发人员的很多工作，减少数据在数据库和应用服务器之间的传输，对于提高数据处理的效率是有好处的。

存储过程思想上很简单，就是数据库 SOL 语言层面的代码封装与重用。

**特点：**

- 封装，复用
- 可以接收参数，也可以返回数据
- 减少网络交互，效率提升

#### 基本语法

```
查看视图数据:SELECT*FROM 视图名称…;
```

**查看：**

```
SELECT* FROM INFORMATION SCHEMA.ROUTINES WHERE ROUTINE_SCHEMA='xx';--查询数据库的存储过程及状态信息`
`SHOW CREATE PROCEDURE 存储过程名称;--查询某个存储过程的定义
```

**删除：**

```
DROP PROCEDURE [IF EXISTS]存储过程名称;
```

案例：

```
-- 存储过程基本语法
-- 创建
create procedure p1()
begin
  select count(*)from student;
end;

-- 调用
call p1();

-- 查看
select * from information_schema.ROUTINES where ROUTINE_SCHEMA = 'itcast';
show create procedure p1;

-- 删除
drop procedure if exists p1;
```

#### 变量

##### 系统变量

系统变量 是 MySQL 服务器提供，不是用户定义的，属于服务器层面。分为全局变量(GLOBAL)、会话变量(SESSION)。

查看系统变量

```
SHOW [SESSION |GLOBAL] VARIABLES ; --查看所有系统变量`
`SHOW[SESSION|GLOBAL] VARIABLES LIKE'; --可以通过LKE模糊匹配方式查找变量`
`SELECT @@[SESSION|GLOBAL]系统变量名; -- 查看指定变量的值
-- 变量：系统变量
-- 查看系统变量
show session variables;
show session variables like 'auto%';
show glabal variables like 'auto%';
select @@global.autocommit;

-- 设置系统变量
set session autocommit = 1;
insert intto course(id, name) values (6, 'ES');
set global auto commit = 0;
```

**注意：**

- 如果没有指定 session / global，默认 session，会话变量
- myesql 服务器重启之后，所设置的全局参数会失效，要想不失效，需要更改/etc/my.cnf 中的配置。

##### 用户定义变量

用户定义变量 是用户根据需要自己定义的变量，用户变量不用提前声明，在用的时候直接用“@变量名”使用就可以。其作用域为当前连接。

**赋值：**

```
SET @var name = expr [, @var_name = expr]...;`
`SET @var name := expr [, @var_name := expr]...;
SELECT @var name := expr , @var name := expr ...;`
`SELECT 字段名 INTO @var_name FROM 表名;
```

**使用：**

```
SELECT @var_name;
```

案例：

```
-- 变量：用户变量
-- 赋值
set @myname = 'itcast';
set @myage := 10;

select @mycolor := 'red';
select count(*) into @mycount from tb_user;

-- 使用
select @myname, @myage, @mycolor, @mycount;

select @abc; -- 输出为NULL
```

**注意：**

用户定义的变量无需对其进行声明或者初始化，只不过获取到的值为 NULL。

##### 局部变量

局部变量 是根据需要定义的在局部生效的变量，访问之前，需要 DECLARE 声明。可用作存储过程内的局部变量和输入参数，局部变量的范围是在其内声明的 BEGIN .. END 块。

**声明：**

```
DECLARE 变量名 变量类型 [DEFAULT..];
```

变量类型就是数据库字段类型: INT、BIGINT、CHAR、VARCHAR、DATE、TIME 等。

**赋值：**

```
SET 变量名=值;
SET 变量名:=值;
SELECT 字段名 INTO 变量名 FROM 表名 ...;
```

案例：

```
-- 变量：局部变量
-- 声明 - declare
-- 赋值 -
create procedure p2()
begin
  declare stu_count int default 0;
  select count(*) into stu_count from student;
  select stu_count;
end;

call p2();
```

#### if 判断

语法：

```
IF 条件1 THEN
        ...
ELSEIF 条件2 THEN -- 可选
        ...
ELSE              -- 可选
        ...
END IF;
```

案例：

```
create procedure p3()
begin
  declare score int default 58;
  declare result varchar(10);
  if score >= 85 then
    set result :='优秀';
  elseif score >= 60 then
    set result :='及格';
  else
    set result :='不及格';
  end if;
  select result;
end;
```

#### 参数（in, out, inout)

| 类型  | 含义                                            | 备注 |
| ----- | ----------------------------------------------- | ---- |
| IN    | 该类参数作为输入，也就是需要调用时传入值        | 默认 |
| OUT   | 该类参数作为输出，也就是该参数可以作为返回值    |      |
| INOUT | 既可以作为输入参数，也可以作为输出参数 \*\*\*\* |      |

**用法：**

```
CREATE PROCEDURE 存储过程名称([IN/OUT/INOUT 参数名 参数类型 ])
BEGIN
    -- SQL语句
END :
```

**案例：**

```
-- 根据传入(in)参数score，判定当前分数对应的分数等级，并返回(out)
-- score >= 85分，等级为优秀。
-- score >= 60分 且 score < 85分，等级为及格
-- score < 60分，等级为不及格。
create procedure p3(in score int, out result varchar(10))
begin
  if score >= 85 then
    set result :='优秀';
  elseif score >= 60 then
    set result :='及格';
  else
    set result :='不及格';
  end if;
  select result;
end;

-- 将传入的200分制的分数，进行换算，换算成百分制，然后返回分数 --> inout
create procedure p5(inout score double)
begin
  set score := score * 0.5;
end;

set @score = 198;
call p5(score);
select @score;
```

#### case

**语法一：**

```
CASE case value
  WHEN when_value1 THEN statement_list1
  [WHEN when_value2 THEN statement_list2]...
  [ELSE statement_list ]
END CASE;
```

**语法二：**

```
CASE
  WHEN search_conditionl THEN statement_list1
  WHEN search_condition2 THEN statement_list2]...
  [ELSE statement_list]
END CASE;
```

**案例：**

```
-- case
-- 根据传入的月份，判定月份所属的季节(要求采用case结构)
-- 1-3月份，为第一季度
-- 4-6月份，为第二季度
-- 7-9月份，为第三季度
-- 10-12月份，为第四季度

create procedure p6(in month int)
begin
  declare result varchar(10);
  case
    when month >= 1 and month <= 3 then
      set result := '第一季度';
    when month >= 4 and month <= 6 then
      set result := '第二季度';
    when month >= 7 and month <= 9 then
      set result := ' 第三季度';
    when month >= 10 and month <= 12 then
      set result := '第四季度';
    else
      set result := '非法参数';
  end case;

  select concat('你输入的月份为：', month, '，所属季度为：', result);
end;
```

#### 循环

##### while

while 循环是有条件的循环控制语句。满足条件后，再执行循环体中的 SQL 语句。

**语法：**

```
#先判定条件，如果条件为true，则执行逻辑，否则，不执行逻辑
WHILE 条件 DO
  SOL逻辑...
END WHILE;
```

**案例：**

```
-- while计算从1累加到 n 的值，n 为传入的参数值。
-- A.定义局部变量，记录累加之后的值;
-- B.每循环一次，就会对 n 进行减1，如果 n 减到0，则退出循环

create procedure p7(in n int)
begin
  declare total int default 0;

  while n>0 do
    set total := total + n
    set n:=n-1;
  end while;

  select total;
end;
call p7( n: 100);
```

##### repeat

repeat 是有条件的循环控制语句, 当满足条件的时候退出循环。

与 while 区别：

1. 先进行循环一次再判断。相当于 c 语言中的 do while();
2. 满足条件则退出

**语法：**

```
#先执行一次逻辑，然后判定逻辑是否满足，如果满足，则退出。如果不满足，则继续下一次循环
REPEAT
  SOL逻辑.
  UNTIL 条件
END REPEAT;
```

**案例：**

```
-- while计算从1累加到 n 的值，n 为传入的参数值。
-- A.定义局部变量，记录累加之后的值;
-- B.每循环一次，就会对 n 进行减1，如果 n 减到0，则退出循环

create procedure p8(innint)
begin
  declare total int default 0;

  repeat
    set total := total + n;
    set n := n - 1;
  until n <= 0
  end repeat;

  select total;
end;

call p8( n: 10);
call p8( n: 100);
```

##### loop

LOOP 实现简单的循环，如果不在 SQL 逻辑中增加退出循环的条件，可以用其来实现简单的死循环。LOOP 可以配合一下两个语句使用。

1. LEAVE: 配合循环使用，退出循环。
2. ITERATE: 必须用在循环中，作用是跳过当前循环剩下的语句，直接进入下一次循环。

```
[begin label:] LOOP
  SQL逻辑..
END LOOP [end label];

LEAVE label;  -- 退出指定标记的循环体
ITERATE label;-- 直接进入下一次循环
```

**案例：**

```
-- loop 计算从1到n之间的偶数累加的值，n为传入的参数值。
-- A.定义局部变量，记录累加之后的值;
-- B.每循环一次，就会劝进行-1，如果n减到0，则退出循环。------> leave xx
-- C.如果当次累加的数据是奇数，则直接进入下一次循坏。-------> iterate xx

create procedure p10(in n int)
begin
  declare total int defatult 0;

  sum: loop
    if n <= 10 then
      leave sum;
    end if;

    if n %2 = 1 then
      set n := n - 1;
      iterate sum;
    end if;

    set total := total + n;
    set n := n - 1;
  end loop sum;

  select total;
end;
```

#### 游标-cursor

游标(CURSOR)是用来存储查询结果集的数据类型, 在存储过程和函数中可以使用游标对结果集进行循环的处理。游标的使用包括游标的声明、OPEN、FETCH 和 CLOSE，其语法分别如下。

通俗点讲：类似于 c 语言中的结构体，java 中的实体类。

**声明游标**

```
DECLARE 游标名称 CURSOR FOR 查询语句;
```

**打开游标：**

```
OPEN 游标名称;
```

**获取游标记录：**

```
FETCH 游标名称 INTO 变量[,变量];
```

**关闭游标：**

```
CLOSE 游标名称;
```

**案例：**

```
-- 游标
-- 根据传入的参数uage，来查询用户表tb_user 中， 所有的用户年龄小于uage的用户姓名（name)和专业（profession），
-- 并将用户的姓名和专业插入到所创建的一张新表(id,name,profession)中。
-- 逻辑:
-- A.声明游标，存储查询结果集-
-- B.准备:创建表结构
-- C.开启游标-
-- D.获取游标中的记录
-- E.插入数据到新表中-
-- F.关闭游标

create procedure p11(in uage int)
begin
  declare uname varchar(100);
  declare upro varchar(100);
  declare u_cursor cursor for select name, profession from tb_user where age <= uage;

  drop table if exists tb_user_pro;
  create table if not exists tb_user_pro(
    id int primary key auto_increment,
    name varchar(100),
    profession varchar(100)
  );

  open u_cursor;
  while true do
    fetch u_cursor into uname,upro;
    insert into tb_user_pro values(null, uname, upro);
  end while;
  close u_cursor;
end;
```

##### 条件处理程序-handler

条件处理程序(Handler)可以用来定义在流程控制结构执行过程中遇到问题时相应的处理步骤。

**语法：**

```
DECLARE handler action HANDLERFOR condition value l, condition value.... statement;
handler action
  CONTINUE: 继续执行当前程序
  EXIT: 终止执行当前程序
condition value
  SOLSTATE sqlstate_value:状态码，如 02000
  SQLWARNING:所有以01开头的SQLSTATE代码的简写
  NOT FOUND:所有以02开头的SOLSTATE代码的简写
  SOLEXCEPTION:所有没有被SOLWARNING 或 NOT FOUND捕获的SOLSTATE代码的简写
```

**案例：**

```sql
create procedure p11(in uage int)
begin
  declare uname varchar(100);
  declare upro varchar(100);
  declare u_cursor cursor for select name, profession from tb_user where age <= uage;

  -- 监控到02000的状态码后，关闭游标后执行exit退出操作。
  declare exit handler for not found close u_cursor;

  drop table if exists tb_user_pro;
  create table if not exists tb_user_pro(
    id int primary key auto_increment,
    name varchar(100),
    profession varchar(100)
  );

  open u_cursor;
  while true do
    fetch u_cursor into uname,upro;
    insert into tb_user_pro values(null, uname, upro);
  end while;
  close u_cursor;
end;
```

### 存储函数

存储函数是有返回值的存储过程，存储函数的参数只能是 IN 类型的。

存储函数用的较少，能够使用存储函数的地方都可以用存储过程替换。

**语法：**

```sql
CREATE FUNCTION 存储函数名称([ 参数列表 ])
RETURNS type [characteristic ...]
BEGIN
  -- SQL语句
  RETURN ...;
END ;
characteristic说明:
· DETERMINISTIC:相同的输入参数总是产生相同的结果
· NO SQL:不包含 SQL语句。
· READS SOL DATA:包含读取数据的语句，但不包含写入数据的语句,
```

**案例：**

```sql
create function fun1(n int)
returns int deterministic
begin
  declare total int default 0;

  while n > 0 do
    set total := total + n;
    set n := n - 1;
  end while;

  return total;
end;
```

### 触发器

触发器是与表有关的数据库对象，指在 insert/update/delete 之前或之后，触发并执行触发器中定义的 SQL 语句集合。触发器的这种特性可以协助应用在数据库端确保数据的完整性，日志记录，数据校验等操作。

使用别名 OLD 和 NEW 来引用触发器中发生变化的记录内容，这与其他的数据库是相似的。现在触发器还只支持行级触发，不支持语句级触发。

| 触发器类型      | NEW 和 OLD                                             |
| --------------- | ------------------------------------------------------ |
| insert 型触发器 | NEW 表示将要或者已经新增的数据                         |
| update 型触发器 | OLD 表示修改之前的数据，NEW 表示将要或已经修改后的数据 |
| delete 型触发器 | OLD 表示将要或者已经删除的数据                         |

**语法：**

**创建：**

```
CREATE TRIGGER trigger name
BEFORE/AFTER INSERT/UPDATE/DELETE
ON tbl name FOR EACH ROW --行级触发器BEGIN
  trigger_stmt;
END;
```

**查看：**

```
SHOW TRIGGERS;
```

**删除：**

```
DROP TRIGGER [schema_name.]trigger_name; --如果没有指定 schema name，默认为当前数据库
```

**案例：**

```sql
-- 插入数据触发器
create trigger tb_user_insert_trigger
  after insert on tb_user for each row
  begin
  insert into user_logs(id, operation, operate_time, operate_id, operate_params)values
  (null, 'insert', now(), new.id, concat('插入的数据内容为：id=', new.id, ',name=', new.name, ', phone=', new.phone, ', email=', new.email, ', profession=', new.profession));
end;

-- 查看
show triggers;

-- 删除
drop trigger tb_user_insert_trigger;

-- 插入数据tb_user
insert into tb_user(id, name, phtone, email, profession, age, gender, status, createtime) values(25, '二皇子', '1880901212', 'erhuangzi@163.com', '软件工程', 23, '1', '1'1, now());

-- 修改数据触发器
create trigger tb_user_update_trigger
  after update on tb_user for each row
  begin
  insert into user_logs(id, operation, operate_time, operate_id, operate_params)values
  (null, 'update', now(), new.id,
   concat('更新之前的数据：id=', old.id, ',name=', old.name, ', phone=', old.phone, ', email=', old.email, ', profession=', old.profession,
    '更新之后的数据：id=', new.id, ',name=', new.name, ', phone=', new.phone, ', email=', new.email, ', profession=', new.profession));
end;

update tb_user set age = 32 where id = 23;
update tb_user set age = 32 where id <= 5; -- 触发器为行级触发器，所以更改几行数据则出发几次，该语句出发5次

-- 删除数据触发器
create trigger tb_user_delete_trigger
  after delete on tb_user for each row
  begin
  insert into user_logs(id, operation, operate_time, operate_id, operate_params)values
  (null, 'insert', now(), old.id,
   concat('删除之前的数据：id=', new.id, ',name=', old.name, ', phone=', old.phone, ', email=', old.email, ', profession=', old.profession));
end;

delete from tb_user where id = 26;
```

## 锁

介绍：

锁是计算机协调多个进程或线程并发访问某一资源的机制。在数据库中，除传统的计算资源(CPU、RAM、I/0)的争用以外，数据也是一种供许多用户共享的资源。如何保证数据并发访问的一致性、有效性是所有数据库必须解决的一个问题，锁冲突也是影响数据库并发访问性能的一个重要因素。从这个角度来说，锁对数据库而言显得尤其重要，也更加复杂。

分类：

MySQL 中的锁，按照锁的粒度分，分为一下三类：

1. 全局锁：锁定数据库中的所有表。
2. 表级锁：每次操作锁住整张表。
3. 行级锁：每次操作锁住对应的行数据。

### 全局锁

介绍：

全局锁就是对整个数据库实例加锁，加锁后整个实例就处于只读状态，后续的 DML 的写语句，DDL 语句，已经更新操作的事务提交语句都将被阻塞。

其典型的使用场景是做全库的逻辑备份，对所有的表进行锁定，从而获取一致性视图，保证数据的完整性。

基本操作：

使用全局锁：`flush tables with read lock`
释放全局锁：`unlock tables`

演示图：

![image-20250722152144921](http://img.sut.qzz.io/img/20250722152145245.webp)

**备份操作：** `mysqldump -u root -p 1234 itcast > itcast_d.sql`

特点：

数据库中加全局锁，是一个比较重的操作，存在以下问题:

1. 如果在主库上备份，那么在备份期间都不能执行更新，业务基本上就得停摆。
2. 如果在从库上备份，那么在备份期间从库不能执行主库同步过来的二进制日志(binlog)，会导致主从延迟。（该结构会在后续主从复制讲解）

解决方法：

在 InnoDB 引擎中，我们可以在备份时加上参数 –single-transaction 参数来完成不加锁的一致性数据备份。

`mysqldump --single-transaction -uroot -p123456 itcast > itcast.sql`（只适用于支持「可重复读隔离级别的事务」的存储引擎）

原理补充：通过加上这个参数，确保了在备份开始时创建一个一致性的快照，通过启动一个新的事务来实现这一点。（该事务的隔离级别是 Repeatable Read 级别），从而实现在该事务读取下一直读取的是创建时的数据，而不影响其他事务的读写操作。

### 表级锁

每次操作锁住整张表。锁定粒度大，发生锁的冲突的概率最高，并发度最低。应用在 MyISAM、InnoDB、BDB 等存储引擎中。

对于表级锁，主要分为一下三类：

1. 表锁
2. 元数据锁（meta data lock，MDL）
3. 意向锁

#### 表锁

对于表锁，分为两类：

1. 表共享读锁（read lock）
2. 表独占写锁（write lock）

**读锁不会阻塞其他客户端的读，但是会阻塞写。写锁既会阻塞其他客户端的读，又会阻塞其他客户端的写。**

语法：

```
//表级别的共享锁，也就是读锁；
//允许当前会话读取被锁定的表，但阻止其他会话对这些表进行写操作。
lock tebles t_student read;

//表级锁的独占锁，也是写锁；
//允许当前会话对表进行读写操作，但阻止其他会话对这些表进行任何操作（读或写）。
lock tables t_stuent write;
```

释放所有锁：

`unlock tables` （会话退出，也会释放所有锁）

#### 元数据锁

MDL 加锁过程是系统自动控制，无需显式使用，在访问一张表的时候会自动加上。MDL 锁主要作用是维护表元数据的数据一致性，在表
上有活动事务的时候，不可以对元数据进行写入操作。**为了避免 DML 与 DDL 冲突，保证读写的正确性。**

- 对一张表进行 CRUD 操作时，加的是 **MDL 读锁**；
- 对一张表做结构变更操作的时候，加的是 **MDL 写锁**；

| 对应 SQL                                    | 锁类型                                | 说明                                                 |
| ------------------------------------------- | ------------------------------------- | ---------------------------------------------------- |
| lock tables xxx read /write                 | SHARED_READ_ONLY/SHARED_NO_READ_WRITE |                                                      |
| select 、 select … lock in share mode       | SHARED_READ                           | 与 SHARED_READ、SHARED_WRITE 兼容，与 EXCLUSIVE 互斥 |
| insert 、update、delete、select …for update | SHARED_WRITE                          | 与 SHARED_READ、SHARED_WRITE 兼容，与 EXCLUSIVE 互斥 |
| alter table …                               | EXCLYSIVE                             | 与其他的 MDL 都互斥                                  |

查看元数据锁：

```
select object_type,object_schema,object_name,lock_type,lock_duration from performance_schema.metadata_locks;
```

#### 意向锁

为了避免 DML 在执行时，加的行锁与表锁的冲突，在 InnoDB 中引入了意向锁，使得表锁不用检查每行数据是否加锁，使用意向锁来减
少表锁的检查。

**意向共享锁和意向独占锁是表级锁，不会和行级的共享锁和独占锁发生冲突，而且意向锁之间也不会发生冲突，只会和共享表锁（lock tables … read)和独占表锁（lock tables … write）发生冲突**

如果没有「意向锁」，那么加「独占表锁」时，就需要遍历表里所有记录，查看是否有记录存在独占锁，这样效率会很慢。

那么有了「意向锁」，由于在对记录加独占锁前，先会加上表级别的意向独占锁，那么在加「独占表锁」时，直接查该表是否有意向独占锁，如果有就意味着表里已经有记录被加了独占锁，这样就不用去遍历表里的记录。

**意向锁的目的是为了快速判断表里是否有记录被加锁**。

加锁方式：

意向共享锁：（先在表上加上意向共享锁，然后对读取的记录加共享锁）
由 `select ... lock in share mode` 添加

意向独占锁：（先表上加上意向独占锁，然后对读取的记录加独占锁）
由 `insert、update、delete、select ... for update` 添加

### 行级锁

行级锁，每次操作锁住对应的行数据。锁定粒度最小，发生锁冲突的概率最低，并发度最高。应用在 InnoDB 存储引擎中。

1. 行锁(Record Lock): 锁定单个行记录的锁，防止其他事务对此行进行 update 和 delete。在 RC、RR 隔离级别下都支持。
2. 间隙锁(GapLock): 锁定索引记录间隙(不含该记录)，确保索引记录间隙不变，防止其他事务在这个间隙进行 insert，产生幻读。在 RR 隔离级别下都支持。
3. 临键锁(Next-Key Lock): 行锁和间隙锁组合，同时锁住数据，并锁住数据前面的间隙 Gap。在 RR 隔离级别下支持。

![image-20250722155529114](http://img.sut.qzz.io/img/20250722155529427.webp)

#### Record Lock（行锁）

Record Lock 称为记录锁，锁住的是一条记录。而且记录锁是有 S 锁和 X 锁之分。

InnoDB 实现了以下两种类型的行锁：

1. 共享锁(S)：允许一个事务去读一行，阻止其他事务获得相同数据集的排它锁。
2. 排他锁(X)：允许获取排他锁的事务更新数据，阻止其他事务获得相同数据集的共享锁和排他锁。

|           | S(共享锁) | X(排他锁) |
| --------- | --------- | --------- |
| S(共享锁) | 兼容      | 冲突      |
| X(排他锁) | 冲突      | 冲突      |

行锁类型：

| SQL                         | 行锁类型   | 说明                                        |
| --------------------------- | ---------- | ------------------------------------------- |
| insert，update，delete …    | 排他锁     | 自动加锁                                    |
| select                      | 不加任何锁 |                                             |
| select … lock in share mode | 共享锁     | 需要手动 select 之后加上 lock in share mode |
| select … for update         | 排他锁     | 需要手动在 select 之后 for update           |

默认情况下，InnoDB 在 REPEATABLE READ 事务隔离级别运行，InnoDB 使用 next-key 锁进行搜索和索引扫描，以防止幻读。

1. 针对唯一索引进行检索时，对已存在的记录进行等值匹配时，将会自动优化为行锁。
2. InnoDB 的行锁是针对于索引加的锁，不通过索引条件检索数据，那么! nnoDB 将对表中的所有记录加锁，此时 **就会升级为表锁**。

查看意向锁及行锁的加锁情况：

```
select object_schema,object_name,index_name,lock_type,lock_mode,lock_data from peformance_schema.data_locks;
```

#### Gap Lock（间隙锁）

![image-20250722155159455](http://img.sut.qzz.io/img/20250722155159768.webp)

#### Next-Key Lock（临键锁）

![image-20250722155217292](http://img.sut.qzz.io/img/20250722155217592.webp)

默认情况下，InnoDB 在 REPEATABLE READ 事务隔离级别运行，InnoDB 使用 next-key 锁进行搜索和索引扫描，以防止幻读。

1. 索引上的等值查询(唯一索引)，给不存在的记录加锁时, 优化为间隙锁 。
2. 索引上的等值查询(普通索引)，向右遍历时最后一个值不满足查询需求时，next-keylock 退化为间隙锁。
3. 索引上的范围查询(唯一索引)–会访问到不满足条件的第一个值为止。

**总结：**

**加锁的对象是索引，加锁的基本单位是 next-key lock**，它是由记录锁和间隙锁组成的， next-key lock 是前开后闭区间，而间隙锁是前开后开区间。

但是在 MySQL 中，临键锁在一些场景下会退化成间隙锁或记录锁

那么是在什么场景下呢？ 总结一下就是，**在能使用记录锁或者间隙锁就能避免幻读现象的场景下，临键锁就会退化成记录锁或者间隙锁。**

在 MySQL 8.0 以上的版本，默认的隔离级别是 RR 可重复读，所以加锁的规则适用下述例子。

##### 唯一索引（主键索引）等值查询

当我们使用唯一索引进行等值查询时，查询的记录是否存在，加锁的规则也会不同。

- 当查询的记录是「存在」的，在索引 l 树上定位到这一条记录后，将该记录的索引 l 中的 next-keylock 会

  **退化成「记录锁 J**。

- 当查询的记录是「不存在」的，在索引树找到第一条大于该查询记录的记录后，将该记录的索引中的

  next-keylock 会 **退化成「间隙锁 J**。

  1、记录存在的情况

  假设事务 A 执行了这条等值查询语句，查询的记录是 **存在** 于表中的。

  ```sql
  mysql> begin;
  Query OK, 0 rows affected (0.00 sec)
  mysql> select * from user where id = 1 for update;
  +----+--------+-----+
  | id | name   | age |
  +----+--------+-----+
  | 1  | 路飞    | 19  |
  +----+--------+-----+
  1 row in set (0.02 sec)

  ```

  这时事务 A 就为我们 id = 1 的记录加上了 X 型的记录锁，所以在事务 B 或者事务 C 进行修改和删除操作的时候会被堵塞。

  分析一下事务 A 上了什么锁：

  - 行锁：X 类型的记录锁
  - 表锁：X 类型的意向锁

  这里我们重点关注行级锁，**此时事务 A 在 id = 1 记录的主键索引上加的是记录锁，锁住的范围是 id 为 1 的这条记录。** 这
  样其他事务就无法对 id 为 1 的这条记录进行更新和删除操作了。

  幻读的定义就是，当一个事务前后两次查询的结果集，不相同时，就认为发生幻读。所以，要避免幻读就
  是避免结果集某一条记录被其他事务删除，或者有其他事务插入了一条新记录，这样前后两次查询的结果
  集就不会出现不相同的情况。

##### 唯一索引（主键索引)范围查询

范围查询和等值查询的加锁规则是不同的。
当唯一索引进行范围查询时，**会对每一个扫描到的索引 l 加 next-key 锁，然后如果遇到下面这些情况，会**
**退化成记录锁或者间隙锁：**

- 情况一：针对「大于等于」的范围查询，因为存在等值查询的条件，那么如果等值查询的记录是存在于
  表中，那么该记录的索引中的 next-key 锁会 **退化成记录锁**
- 情况二：针对「小于或者小于等于」的范围查询，要看条件值的记录是否存在于表中：
  - 当条件值的记录不在表中，那么不管是「小于」还是「小于等于」条件的范围查询，**扫描到终止范**
    **围查询的记录时，该记录的索引的 next-key 锁会退化成间隙锁**，其他扫描到的记录，都是在这些记
    录的索引 l 上加 next-key 锁。
  - 当条件值的记录在表中，如果是「小于」条件的范围查询，**扫描到终止范围查询的记录时，该记录**
    **的索引的 next-key 锁会退化成间隙锁**，其他扫描到的记录，都是在这些记录的索引 l 上加 next-key
    锁；如果「小于等于」条件的范围查询，扫描到终止范围查询的记录时，该记录的索引 next-key 锁
    不会退化成间隙锁。其他扫描到的记录，都是在这些记录的索引 l 上加 next-key 锁。

##### 非唯一索引等值查询

当我们用非唯一索引进行等值查询的时候，**因为存在两个索引，一个是主键索引，一个是非唯一索引（二**
**级索引），所以在加锁时，同时会对这两个索引都加锁，但是对主键索引加锁的时候，只有满足查询条件**
**的记录才会对它们的主键索引加锁**

针对非唯一索引等值查询时，查询的记录存不存在，加锁的规则也会不同：

- 当查询的记录「存在」时，由于不是唯一索引，所以肯定存在索引值相同的记录，于是 **非唯一索引等值**
  **查询的过程是一个扫描的过程，直到扫描到第一个不符合条件的二级索引记录就停止扫描，然后在扫描**
  **的过程中，对扫描到的二级索引记录加的是 next-key 锁，而对于第一个不符合条件的二级索引记录，**
  **该二级索引的 next-key 锁会退化成间隙锁。同时，在符合查询条件的记录的主键索引上加记录锁**
- 当查询的记录「不存在」时，**扫描到第一条不符合条件的二级索引记录，该二级索引的 next-key 锁会退化成间隙锁。因为不存在满足查询条件的记录，所以不会对主键索引加锁**

## InnoDB 引擎

![image-20250724224108984](http://img.sut.qzz.io/img/20250724224109287.webp)

### 逻辑存储架构

![image-20250722230522899](http://img.sut.qzz.io/img/20250722230523313.webp)

- 表空间(ibd 文件)，一个 mysql 实例可以对应多个表空间，用于存储记录、索引等数据。
- 段，分为数据段（Leafnodesegment）、索引 l 段（Non-leafnodesegment）、回滚段（Rollbacksegment），InnoDB 是索引组织表，数据段就是 B+树的叶子节点，索引 l 段即为 B+树的非叶子节点。段用来管理多个 Extent（区）。
- 区，表空间的单元结构，每个区的大小为 1M。默认情况下，InnoDB 存储引擎页大小为 16K, 即一个区中一共有 64 个连续的页
- 页，是 InnoDB 存储引擎磁盘管理的最小单元，每个页的大小默认为 16KB。为了保证页的连续性，InnoDB 存储引擎每次从磁盘申请 4-5 个区。
- 行，InnoDB 存储引擎数据是按行进行存放的。
  - Trx_id：每次对某条记录进行改动时，都会把对应的事务 id 赋值给 trx_id 隐藏列。
  - Roll_pointer：每次对某条引记录进行改动时，都会把旧的版本写入到 undo 日志中，然后这个隐藏列就相当于一个指针，可以通过它来找到该记录修改前的信息。

### 架构

MySQL5.5 版本开始，默认使用 InnoDB 存储引擎，它擅长事务处理，具有崩溃恢复特性，在日常开发中使用非常广泛。下面是 InnoDB 架构图，左侧为内存结构，右侧为磁盘结构。

![image-20250722231007169](http://img.sut.qzz.io/img/20250722231007445.webp)

#### 内存架构

##### buffer pool

Buffer Pool: 缓冲池是主内存中的一个区域，里面可以缓存磁盘上经常操作的真实数据，在执行增删改查操作时，先作缓冲池中的数据（若缓冲池没有数据，则从磁盘加载并缓存），然后再以一定频率刷新到磁盘，从而减少磁盘 O, 加快处理速度。

缓冲池以 Page 页为单位，底层采用链表数据结构管理 Page。根据状态，将 Page 分为三种类型：

- ·freepage：空闲 page，未被使用。
- ·cleanpage：被使用 page，数据没有被修改过。
- ·dirtypage：脏页，被使用 page，数据被修改过，也中数据与磁盘的数据产生了不一致。

##### Change Buffer

ChangeBuffer：更改缓冲区（针对于非唯一二级索引 l 页），在执行 DML 语句时，如果这些数据 Page 没有在 BufferPool 中，不会直接操作磁盘，而会将数据变更存在更改缓冲区 ChangeBuffer 中，在未来数据被读取时，再将数据合并恢复到 BufferPool 中，再将合并后的数据刷新到磁盘中。

- ChangeBuffer 的意义是什么？

  与聚集索引不同，二级索引通常是非唯一的，并且以相对随机的顺序插入二级索引。同样，删除和更
  新可能会影响索引树中不相邻的二级索引页，如果每一次都操作磁盘，会造成大量的磁盘 IO。有了
  ChangeBuffer 之后，我们可以在缓冲池中进行合并处理，减少磁盘 lO。

##### Adaptive Hash Index

AdaptiveHashIndex：自适应 hash 索引 l，用于优化对 BufferPool 数据的查询。InnoDB 存储引 l 擎会监
控对表上各索引 l 页的查询，如果观察到 hash 索引可以提升速度，则建立 hash 索引 l，称之为自适应 hash
索引。
**自适应哈希索引，无需人工干预，是系统根据情况自动完成。**
参数：adaptive_hash_index

##### Log Buffer

Log Buffer: 日志缓冲区，用来保存要写入到磁盘中的 log 日志数据(redo log、undo log), 默认大小为 16MB, 日志缓冲区的日志会定期刷新到磁盘中。如果需要更新、插入或删除许多行的事务，增加日志缓冲区的大小可以节省磁盘 I0。
参数：
innodb_log_buffer_size: 缓冲区大小
innodb_flush_log_at_trx_commit: 日志刷新到磁盘时机

#### 磁盘结构

##### System Tablespace

System Tablespace: 系统表空间是更改缓冲区的存储区域。如果表是在系统表空间而不是每个表文件或通用表空间中创建的，它也可能包含表和索引数据。（在 MySQL5.x 版本中还包含 nnoDB 数据字典、undolog 等。
参数：innodb_data_file_path

### 事务原理

事务：

事务 是一组操作的集合，它是一个不可分割的工作单位，事务会把所有的操作作为一个整体一起向系统提交或撤销操作请求，即这些操作要么同时成功，要么同时失败。

特征：

- 原子性(Atomicity)：事务是不可分割的最小操作单元，要么全部成功，要么全部失败。
- 一致性(Consistency) ：事务完成时，必须使所有的数据都保持一致状态。
- 隔离性(lsolation)：数据库系统提供的隔离机制，保证事务在不受外部并发操作影响的独立环境下运行。
- 持久性(Durability)：事务一旦提交或回滚，它对数据库中的数据的改变就是永久的。

特性原理分类图：

#### redo log

重做日志，记录的是事务提交时数据页的物理修改，是用来实现事务的 **持久性**。

该日志文件由两部分组成: 重做日志缓冲(redo log buffer)以及重做日志文件(redo log file), 前者是在内存中，后者在磁盘中。当事务提交之后会把所有修改信息都存到该日志文件中, 用于在刷新脏页到磁盘, 发生错误时, 进行数据恢复使用。

Buffer Pool 在产生脏页数据的时候，会先将数据存储到 redo log buffer 再存储到 redo log 中进行磁盘持久化存储，在内存出现异常（比如突然断电）时，通过 redo log 中持久化的数据进行回滚。过程如下图：

redo log 要写到磁盘，数据也要写磁盘，为什么要多此一举?

写入 redo log 的方式使用了追加操作，所以磁盘操作是 **顺序写**，而写入数据需要先找到写入位置，然后才写到磁盘，所以磁盘操作是 **随机写**。

#### undo log

回滚日志，用于记录数据被修改前的信息，作用包含两个: 提供回滚 和 MVCC(多版本并发控制)。

undo log 和 redo log 记录物理日志不一样，它是逻辑日志。可以认为当 delete 一条记录时，undo log 中会记录一条对应的 insert 记录，反之亦然，当 update 一条记录时，它记录一条对应相反的 update 记录。当执行 rollback 时，就可以从 undo log 中的逻辑记录读取到相应的内容并进行回滚。

Undo log 销毁：undo log 在事务执行时产生，事务提交时，并不会立即删除 undol0g，因为这些日志可能还用于 MVCC。

Undo log 存储：undo log 采用段的方式进行管理和记录，存放在前面介绍的 rollback segment 回滚段中，内部包含 1024 个 undo log segment.

### MVCC

![image-20250724223819539](http://img.sut.qzz.io/img/20250724223819819.webp)

**当前读：**

读取的是记录的最新版本，读取时还要保证其他并发事务不能修改当前记录，会对读取的记录进行加锁。对于我们日常的操作，如: select…lock in share mode(共享锁)，select… for update、update、insert、delete(排他锁)都是一种当前读。

**快照读：**

简单的 select(不加锁)就是快照读，快照读，读取的是记录数据的可见版本，有可能是历史数据，不加锁，是非阻塞读。

- Read committed: 每次 select，都生成一个快照读。
- Repeatable Read: 开启事务后第一个 select 语句才是快照读的地方。
- Serializable: 快照读会退化为当前读。

**MVCC：**

全称 Multi-Version Concurrency Control，多版本并发控制。指维护一个数据的多个版本，使得读写操作没有冲突，快照读为 MVSOL 实现 MVCC 提供了一个非阻塞读功能。MVCC 的具体实现，还需要依赖于数据库记录中的三个隐式字段、undo log 日志、read View。

#### 三个隐藏字段

#### undo log

回滚日志，在 insert、update、delete 的时候产生的便于数据回滚的日志。

当 insert 的时候，产生的 undoloq 日志只在回滚时需要，在事务提交后，可被立即删除。

而 update、delete 的时候，产生的 undo log 日志不仅在回滚时需要，在快照读时也需要，不会立即被删除。

那么何时删除？

- 事务提交后：
  - 对于 `INSERT` 操作，事务提交后，undo log 可以被立即删除，因为不再需要用于回滚。
  - 对于 `UPDATE` 和 `DELETE` 操作，undo log 不会立即被删除，因为它们可能在后续的快照读取中被使用。
- 快照读取结束：
  - 当所有依赖于该 undo log 的快照读取操作结束后，undo log 才会被删除。这意味着如果有一个事务正在进行快照读

#### readview

ReadView(读视图)是 快照读 SOL 执行时 MVCC 提取数据的依据，记录并维护系统当前活跃的事务(未提交的)id。

ReadView 中包含了四个核心字段:

| 字段           | 含义                                                     |
| -------------- | -------------------------------------------------------- |
| m_ids          | 当前活跃的事务 ID 集合                                   |
| min_trx_id     | 最小活跃事务 ID                                          |
| max_trx_id     | 预分配事务 ID，当前最大事务 ID+1（因为事务 ID 是自增的） |
| creator_trx_id | ReadView 创建者的事务 ID                                 |

![image-20250724222535859](http://img.sut.qzz.io/img/20250724222536630.webp)

依次比较 undo log 日志中版本数据链，找到可以进行访问的版本数据。

**不同的隔离级别，生成 ReadView 的时机不同：**

- **READCOMMITTED**：在事务中每一次执行快照读时生成 ReadVieW。
- **REPEATABLEREAD**：仅在事务中第一次执行快照读时生成 ReadVieW，后续复用该 ReadVieW。

![image-20250724223532164](http://img.sut.qzz.io/img/20250724223532484.webp)

## MySQL 管理

![image-20250724225231577](http://img.sut.qzz.io/img/20250724225231885.webp)

### 系统数据库介绍

Mysql 数据库安装完成后，自带了一下四个数据库，具体作用如下：

| 数据库             | 含义                                                                                      |
| ------------------ | ----------------------------------------------------------------------------------------- |
| mysql              | 存储 MVSQL 服务器正常运行所需要的各种信息(时区、主从、用户、权限等)                       |
| information_schema | 提供了访问数据库元数据的各种表和视图，包含数据库、表、字段类型及访问权限等                |
| performance_schema | 为 MySQL 服务器运行时状态提供了一个底层监控功能，主要用于收集数据库服务器性能参数         |
| sys                | 包含了一系列方便 DBA 和开发人员利用 performance_schema 性能数据库进行性能调优和诊断的视图 |

### 常用工具

#### mysql

![image-20250724224528744](http://img.sut.qzz.io/img/20250724224529028.webp)

#### mysqladmin

![image-20250724224638523](http://img.sut.qzz.io/img/20250724224638844.webp)

#### mysqlbinlog

![image-20250724224855039](http://img.sut.qzz.io/img/20250724224855309.webp)

#### mysqlshow

![image-20250724225040480](http://img.sut.qzz.io/img/20250724225040780.webp)

#### mysqldump

![image-20250724225106737](http://img.sut.qzz.io/img/20250724225107152.webp)

#### mysqlimport/source

![image-20250724225142593](http://img.sut.qzz.io/img/20250724225142942.webp)

## 面试题总结

### Q1：InnoDB 和 MyISAM 之间的区别？

- InnoDB 支持事务，MyISAM 不支持
- innoDB 支持行锁，而 MyISAM 支持表锁
- InnoDB 支持外键，而 MyISAM 不支持

### Q2：为什么 InnoDB 引擎选择 B+树索引结构？

![image-20250720235408595](http://img.sut.qzz.io/img/20250720235408729.webp)

### Q3：以下 SQL 语句，那个执行效率高？为什么？

![image-20250721000331286](http://img.sut.qzz.io/img/20250721000331378.webp)

- Answer: 第一条的执行效率高，因为 id 是主键，为聚集索引，因为这时不需要回表查询，速度更快，效率更高

### Q4: InnoDB 主键索引的 B+Tree 高度为多高呢？

Answer：

- 假设：一行数据大小为 1k，一页中可以存储 16 行这样的数据。InnoDB 指针占用 6 个字节的空间，即使为 bigint，占用字节为 8

- 高度为 2： n _8 + (n + 1)_ 6 = 16 \* 1024, 算出 n 约为 1170 这个节点下边最多会有 1171 个指针

- 存储的数据量为：1171 \* 16 = 18736 行的数据

- 一页为 16KB， 一个数据为 1K 的话就是可以存储 16 个，每个树上的节点是一页/块

- 三层一般可存储：2193 万数据。

### Q5: 一张表，有四个字段(id, username, password, status)

由于数据量大，需要对以下 SQL 语句进行优化，该如何进
行才是最优方案：

```sql
select id,username,password from tb_user where username ='itcast';
```

Answer: 针对 id, username, password 三个字段创建联合索引，从而实现覆盖索引，减少回表查询，

SQL 语句：`create index idx_user_id_name_pwd on tb_user(id, username, password);`

# MySQL 运维篇

## 日志

### 错误日志

错误日志是 MySQL 中最重要的日志之一，它记录了当 mysqld 启动和停止时，以及服务器在运行过程中发生任何严重错误时的相关信息当数据库出现任何故障导致无法正常使用时，建议首先查看此日志。

该日志是默认开启的，默认存放目录 /var/log/，默认的日志文件名为 mysqld.log 。查看日志位置：

```
show variables like '%log_error%'
```

### 二进制日志

#### 介绍

二进制日志(BINLOG)记录了所有的 DDL(数据定义语言)语句和 DML(数据操纵语言)语句，但不包括数据查询(SELECT、SHOW)语句。

作用：

1. 灾难时的数据恢复；
2. MySQL 的主从复制。

在 MVSOL8 版本中，默认二进制日志是开启着的，涉及到的参数如下：

```
show variables like '%log_bin%'
```

#### 日志格式

MySQL 服务器中提供了多种格式来记录二进制记录，具体格式及特点如下：

| 日志格式  | 含义                                                                                              |
| --------- | ------------------------------------------------------------------------------------------------- |
| statement | 基于 SQL 语句的日志记录，记录的是 SQL 语句，对数据进行修改的 SQL 都会记录在日志文件中。           |
| row       | 基于行的日志记录，记录的是每一行的数据变更。(默认)                                                |
| mined     | 混合了 STATEMENT 和 ROW 两种格式，默认采用 STATEMENT，在某些特殊情况下会自动切换为 ROW 进行记录。 |

查看参数方式：`show variables like '%binlog_format%';`

#### 日志查看

由于日志是以二进制方式存储的，不能直接读取，需要通过二进制日志查询工具 mysqlbinlog 来查看，具体语法：

```
mysqlbinlog[参数选项]logfilename

参数选项：
    -d    指定数据库名称，只列出指定的数据库相关的操作。
    -o    忽略掉日志中的前n行命令。
    -v    将行事件(数据变更)重构为SOL语句。
    -w    将行事件(数据变更)重构为SQL语句，并输出注释信息
```

#### 日志删除

对于比较繁忙的业务系统，每天生成的 binlog 数据巨大，如果长时间不清除，将会占用大量磁盘空间。可以通过以下几种方式清理日志：

| 指令                                             | 含义                                                                  |
| ------------------------------------------------ | --------------------------------------------------------------------- |
| reset master                                     | 删除全部 binlog 日志，删除之后，日志编号，将从 binlog.000001 重新开始 |
| purge master logs to ‘binlog.\*\*\*’             | 删除 \*\*\* 编号之前的所有日志                                        |
| purge master logs before ‘yyyy-mm-dd hh24:mi:ss’ | 删除日志为”yyyy-mm-dd hh24:mi:ss”之前产生的所有日志                   |

也可以在 mysql 的配置文件中配置二进制日志的过期时间，设置了之后，二进制日志过期会自动删除.

```
show variables like '%binlog_expire_logs_seconds%'
```

### 查询日志

查询日志中记录了客户端的所有操作语句，而二进制日志不包含查询数据的 SQL 语句。默认情况下，查询日志是未开启的。如果需要开启查询日志，可以设置一下配置：

修改 MySQL 的配置文件 /etc/my.cnf 文件，添加如下内容：

```
#该选项用来开启查询日志，可选值：0或者1；0代表关闭，1代表开启
general_log=1
#设置日志的文件名 ， 如果没有指定，默认的文件名为 host_name.log
general_log_file=mysql_query.log
```

### 慢查询日志

慢查询日志记录了所有执行时间超过参数 long_query_time 设置值并且扫描记录数不小于 min_examined_row_limit 的所有的 SQL 语句的日志，默认未开启。long_query_time 默认为 10 秒，最小为 0，精度可以到微秒。

```
#慢查询日志
slow_query_log=1
#执行时间参数
long_query_time=2
```

默认情况下，不会记录管理语句，也不会记录不使用索引进行查找的查询。可以使用 log_slow_admin_statements 和更改此行为 log_queries_not_using_indexes，如下所述。

```
#记录执行较慢的管理语句
log_slow_admin_statements = 1
#记录执行较慢的未使用索引的语句
log_queries_not_using_indexes = 1
```

## 主从复制

### 概述

![image-20250724230751332](http://img.sut.qzz.io/img/20250724230751637.webp)

### 原理

在 MySQL 中主要使用了 binlog 日志文件来实现主从复制。

![image-20250724231047085](http://img.sut.qzz.io/img/20250724231047460.webp)

### 搭建实现

![image-20250724231223800](http://img.sut.qzz.io/img/20250724231224118.webp)

#### 主库配置

![image-20250724231353770](http://img.sut.qzz.io/img/20250724231354020.webp)

![image-20250724231851608](http://img.sut.qzz.io/img/20250724231851906.webp)

#### 从库配置

![image-20250724232029816](http://img.sut.qzz.io/img/20250724232030163.webp)

![image-20250724232059351](http://img.sut.qzz.io/img/20250724232059745.webp)

![image-20250724232212972](http://img.sut.qzz.io/img/20250724232213438.webp)

#### 测试

1、在主库上创建数据库、表，并插入数据

```sql
create database db01;
use db01;
create table tb_use(
 id int(11) primary key not null auto_increment,
 name varchar(50) not null,
 sex varchar(1)
)engine=innodb default charset=utf8mb4;
insert into tb_user(id, name, sex) valurs (null, 'Tom', '1'), (null, 'Trigger', '0'), (null, 'Dawn', '1');
```

2、在从库中查询数据，验证主从是否同步。

## 分库分表

### 介绍

随着互联网及移动互联网的发展，应用系统的数据量也是成指数式增长，若采用单数据库进行数据存储，存在以下性能瓶颈：

- 1.IO 瓶颈：热点数据太多，数据库缓存不足，产生大量磁盘 IO，效率较低。请求数据太多，带宽不够，网络 IO 瓶颈。
- 2.CPU 瓶颈：排序、分组、连接查询、聚合统计等 SQL 会耗费大量的 CPU 资源，请求数太多，CPU 出现瓶颈。

分库分表的中心思想都是将数据分散存储，使得单一数据库/表的数据量变小来缓解单一数据库的性能问题，从而达到提升数据库性能的目的。

#### 拆分策略

![image-20250725134939634](http://img.sut.qzz.io/img/20250725134939980.webp)

#### 垂直拆分

![image-20250725135124374](http://img.sut.qzz.io/img/20250725135124607.webp)

**垂直分库**：以表为依据，根据业务将不同表拆分到不同库中。
特点： 1.每个库的表结构都不一样。 2.每个库的数据也不一样。 3.所有库的并集是全量数据。

![image-20250725135203002](http://img.sut.qzz.io/img/20250725135203228.webp)

**垂直分表**：以字段为依据，根据字段属性将不同字段拆分到不同表中。
特点： 1.每个表的结构都不一样。 2.每个表的数据也不一样，一般通过一列（主键/外键）关联。 3.所有表的并集是全量数据。

#### 水平拆分

![image-20250725135424288](http://img.sut.qzz.io/img/20250725135424533.webp)

**水平分库**：以字段为依据，按照一定策略，将一个库的数据拆分到多个库中。
特点： 1.每个库的表结构都一样。 2.每个库的数据都不一样。 3.所有库的并集是全量数据。

![image-20250725135431231](http://img.sut.qzz.io/img/20250725135431460.webp)

**水平分表**：以字段为依据，按照一定策略，将一个表的数据拆分到多个表中。
特点： 1.每个表的表结构都一样。 2.每个表的数据都不一样。 3.所有的并集是全量数据。

#### 实现技术

● shardingJDBC:基于 AOP 原理，在应用程序中对本地执行的 SQL 进行拦截，解析、改写、路由处理。需要自行编码配置实现，只支持
java 语言，性能较高。
● MyCat:数据库分库分表中间件，不用调整代码即可实现分库分表，支持多种语言，性能不及前者。

### Mycat

#### Mycat 概述

Mycat 是开源的、活跃的、基与 Java 语言编写的 MySOL 数据库中间件。可以像使用 mysql 一样来使用 mycat,对于开发人员来说根本感觉
不到 mycat 的存在。

优势：
性能可靠稳定
强大的技术团队
体系完善
社区活跃

![image-20250725140412506](http://img.sut.qzz.io/img/20250725140412847.webp)

#### 入门

![image-20250725141150594](http://img.sut.qzz.io/img/20250725141150843.webp)

![image-20250725141807814](http://img.sut.qzz.io/img/20250725141808068.webp)

![image-20250725141825257](http://img.sut.qzz.io/img/20250725141825561.webp)

#### Mycat 配置

![image-20250725142610816](http://img.sut.qzz.io/img/20250725142611056.webp)

![image-20250725142754900](http://img.sut.qzz.io/img/20250725142755180.webp)

![image-20250725142833662](http://img.sut.qzz.io/img/20250725142833939.webp)

![image-20250725142939182](http://img.sut.qzz.io/img/20250725142939451.webp)

![image-20250725142949826](http://img.sut.qzz.io/img/20250725142950107.webp)

**Mycat 可以用到再详细理解，这里就烂尾咯~~**

## 读写分离

> 这里引用 Javaguide 中的文章内容 [读写分离](https://javaguide.cn/high-performance/read-and-write-separation-and-library-subtable.html)

# 完结

这里我们全部的内容就学习完成了，关于后边的运维篇，涉及到我们面试需要的可以留到看八股的时候再看，所以这里的笔记有些水，需要更详细的关于 MySQL 的讲解可以参考以下网站：

- [小林 Coding](https://xiaolincoding.com/mysql/)
- [JavaGuide 面试指导](https://javaguide.cn/)
