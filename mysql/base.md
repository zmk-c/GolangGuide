# MySQL基础

![mysql](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/mysql.jpg)

## 1.初识MySQL

```sql
mysql -uroot -p123456	-- 连接数据库

update mysql.user set authentication_string=password('123456') where user='root' and host='localhost';	-- 修改用户名密码
flush privileges;	-- 刷新权限

show databases;	-- 查看所有的数据结构
show tables;	-- 查看数据库中所有表信息
describe student;	-- 显示数据库中所有表的信息
MySQL
-- 单行注释
/*
多行注释
*/
```

DDL数据库定义语言（alter）	

DML数据库操作语言（update，insert，delete）

DQL数据库查询语言（query）

DCL数据库控制语言（grant）

## 2.操作数据库(了解)

操作数据库->操作数据库中的表->操作数据库中表的数据

==关键字不区分大小写==

### 2.1操作数据库

1.创建数据库

```sql
create database [if not exists] student;
```

2.删除数据库

```sql
drop database [if exists] student;
```

3.使用数据库

```sql
use `student`;	-- 如果你的表名或字段名是一个特殊字符，就需要带``
```

4.查看数据库

```sql
show databases;
```

### 2.2数据库的类型

> 数值

- tinyint  十分小的数据  1个字节
- smallint  较小的数据  2个字节
- mediumint 中等大小的数据  3个字节
- **int  标准的整数  4个字节  int32**
- bigint  较大的数据  8个字节
- float  浮点数  4个字节
- double  浮点数  8个字节  精度问题
- decimal 字符串形式的浮点数  金融计算的时候，一般使用decimal 

> 字符串

- char  字符串固定大小 0-255
- **varchar  可变字符串 0-65535  string类型** 
- tinytext  微型文本  2<sup>8</sup>-1
- text  文本串  2<sup>16</sup>-1  保存大文本

> 时间日期

- date  YYYY-MM-DD  日期
- time  HH:mm:ss  时间格式
- **datetime  YYYY-MM-DD HH:mm:ss 最常用的时间格式**
- timestamp  时间戳
- year  年份

> null

- 不要使用null进行运算，结果为null

### 2.3数据库的字段属性(重点)

`Unsigned`

- 无符号整数
- 声明该列不能声明为负数

`zerofill`

- 零填充的
- 不足的位数使用0来填充   int(3)    5  ->  005

`auto_increment`

- 通常理解为自增，自动在上一条记录的基础上+1（默认）
- 通常用来设置唯一的主键
- 可以设置自增的起始值和步长

`not null`

- 假设设置为not null，如果不给它赋值，就会报错
- null，如果不填写值，默认为空

`default`

- sex，默认为男，如果不指定该值，则设置为默认值

每一个表都必须存在以下五个字段！未来做项目用的，表示一个记录存在的意义

`id 主键`

`version 乐观锁`

`is_delete  伪删除`

`gmt_create  创建时间`

`gmt_update  修改时间`

### 2.4创建数据库表(重点)

格式

```sql
create table [if not exists] `表名`(
	`字段名` 列属性 [字段属性] [注释],	
    `字段名` 列属性 [字段属性] [注释],	
    ...
    `字段名` 列属性 [字段属性] [注释],	
)[表类型][字符集设置]
```

```sql
-- 创建学生表(列名，字段) ：学号	登录密码	姓名	性别	出生日期	家庭住址	email
-- 注意点，使用英文()，表的 名称 和 字段 尽量使用``括起来防止特殊字符	
-- auto_increment 自增
-- comment '注释'
-- primary key 主键，一般一个表只有一个
create table if not exists `student`(
    `id`	int(4)	not null auto_increment	comment '学号',
    `name`	varchar(30)	not null default '匿名' comment '姓名',
    `passwd`	varchar(20)	not null default '123456' comment '密码',
    `sex`	varchar(2) not null default '男' comment '性别',
    `birthday` datetime default null comment '出生日期',
    `address` varchar(100) default null comment '家庭住址',
    `email` varchar(30) default null comment '邮箱',
    primary key (`id`)
)engine=innodb default charset=utf8
```

常用命令(创建语句偷懒)

```sql
SHOW CREATE DATABASE kuangshen;-- 查看创建数据库的语句 
-- CREATE DATABASE `kuangshen` /*!40100 DEFAULT CHARACTER SET utf8 */
SHOW CREATE TABLE student; -- 查看创建student表的语句
/*
CREATE TABLE `student` (
  `id` int(4) NOT NULL AUTO_INCREMENT COMMENT '学号',
  `name` varchar(30) NOT NULL DEFAULT '匿名' COMMENT '姓名',
  `passwd` varchar(20) NOT NULL DEFAULT '123456' COMMENT '密码',
  `sex` varchar(2) NOT NULL DEFAULT '男' COMMENT '性别',
  `birthday` datetime DEFAULT NULL COMMENT '出生日期',
  `address` varchar(100) DEFAULT NULL COMMENT '家庭住址',
  `email` varchar(30) DEFAULT NULL COMMENT '邮箱',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8
*/
DESC student; -- 显示表结构
```

### 2.5数据表类型

- INNODB 5.5版本后默认使用 	安全性高，事务的处理，多表多用户操作

- MyISAM    节约空间，速度较快 

|            | MyISAM | INNODB               |
| ---------- | ------ | -------------------- |
| 事务支持   | 不支持 | 支持                 |
| 行锁       | 不支持 | 支持                 |
| 外键约束   | 不支持 | 支持                 |
| 全文索引   | 支持   | 不支持               |
| 表空间大小 | 较小   | 较大，约为MyISAM两倍 |

> 在物理空间存储的位置

所有的数据库文件都存在data目录下，一个文件夹就对应一个数据库

本质还是文件的存储

MySQL引擎在物理文件上的区别

- INNODB	在数据库表中只要一个*.frm文件，以及上级目录下的ibdata1文件
- MyISAM    
  -  *.frm	表结构的定义文件
  - *.MYD   数据文件(data)
  - *.MYI    索引文件(index)

> 设置数据库表的字符串集编码

```sql
charset=utf8
```

不设置的话，会是mysql默认的字符集编码（不支持中文）

MySQL的默认编码是Latin1，不支持中文

可以在my.ini文件中配置默认编码（window系统下）

```sql
character-set-server=utf8
```

### 2.6修改删除表

```sql
-- 修改表名	alter table 旧表名 rename as 新表名;
alter table stu rename as student;
```

```sql
-- 增加表的字段	alter table 表名 add 字段名 列属性;
alter table student add age int(11);
```

```sql
-- 修改表的字段	alter table 表名 modify 字段名 列属性;
alter table student modify age varchar(11); --modify修改约束
-- 修改表的字段	alter table 表名 change 旧字段名 新字段名 列属性;
alter table student change age age1 int(1); --change字段重命名
```

```sql
-- 删除表的字段	alter table 表名 drop 字段名;
alter table student drop age1;
```

```sql
-- 删除表	drop table [if exists] 表名;
drop table if exists student;
```

## 3.MySQL数据管理

### 3.1外键(了解)

> 方式一：建表的同时添加外键(一般不用)

```sql
-- 外键(了解)
CREATE TABLE IF NOT EXISTS `grade`(
	`gradeid` INT(4) NOT NULL AUTO_INCREMENT COMMENT '年级id',
	`gradename` VARCHAR(20) NOT NULL COMMENT '年级名称',
	PRIMARY KEY (`gradeid`)
)ENGINE=INNODB DEFAULT CHARSET=utf8

-- 学生表的 gradeid 字段要求引用年纪表的 gradeid 字段
-- 定义外键key
-- 给这个外键添加约束(执行引用) references 引用

CREATE TABLE IF NOT EXISTS `student`(
    `id`	INT(4)	NOT NULL AUTO_INCREMENT	COMMENT '学号',
    `name`	VARCHAR(30)	NOT NULL DEFAULT '匿名' COMMENT '姓名',
    `passwd`	VARCHAR(20)	NOT NULL DEFAULT '123456' COMMENT '密码',
    `sex`	VARCHAR(2) NOT NULL DEFAULT '男' COMMENT '性别s',
    `birthday` DATETIME DEFAULT NULL COMMENT '出生日期',
    `address` VARCHAR(100) DEFAULT NULL COMMENT '家庭住址',
    `email` VARCHAR(30) DEFAULT NULL COMMENT '邮箱',
    `gradeid` INT(4) NOT NULL COMMENT '年级id',
    PRIMARY KEY (`id`), 
    KEY `FK_gradeid` (`gradeid`), --主要就这两行
    CONSTRAINT `FK_gradeid` FOREIGN KEY (`gradeid`) REFERENCES `grade`(`gradeid`)
)ENGINE=INNODB DEFAULT CHARSET=utf8
```

> 方式二：创建表成功后，添加外键约束

```sql
-- 创建表的时候没有外键关系 之后添加外键(用这种)
CREATE TABLE IF NOT EXISTS `grade`(
	`gradeid` INT(4) NOT NULL AUTO_INCREMENT COMMENT '年级id',
	`gradename` VARCHAR(20) NOT NULL COMMENT '年级名称',
	PRIMARY KEY (`gradeid`)
)ENGINE=INNODB DEFAULT CHARSET=utf8

CREATE TABLE IF NOT EXISTS `student`(
    `id`	INT(4)	NOT NULL AUTO_INCREMENT	COMMENT '学号',
    `name`	VARCHAR(30)	NOT NULL DEFAULT '匿名' COMMENT '姓名',
    `passwd`	VARCHAR(20)	NOT NULL DEFAULT '123456' COMMENT '密码',
    `sex`	VARCHAR(2) NOT NULL DEFAULT '男' COMMENT '性别s',
    `birthday` DATETIME DEFAULT NULL COMMENT '出生日期',
    `address` VARCHAR(100) DEFAULT NULL COMMENT '家庭住址',
    `email` VARCHAR(30) DEFAULT NULL COMMENT '邮箱',
    `gradeid` INT(4) NOT NULL COMMENT '年级id',
    PRIMARY KEY (`id`), 
)ENGINE=INNODB DEFAULT CHARSET=utf8

-- alter table 表名 add constraint 约束名 foreign key (作为外键的列) references 哪个表(哪个字段)
ALTER TABLE `student` ADD CONSTRAINT `FK_gradeid` FOREIGN KEY (`gradeid`) REFERENCES `grade`(`gradeid`);
```

以上的操作都是**物理外键**，数据库级别的外键，删除外键关系的表的时候，必须要先删除引用别人的表（从表），再删除被引用的表（主表），不建议使用！（避免数据库过多造成困扰）

==最佳实践==

- 数据库就是单纯的表，只用来存储数据，只要行（数据）和列（字段）
- 我们想使用多张表的数据，想使用外键（程序去实现）

### 3.2DML(全部记住)

**数据库意义：**数据存储，数据管理

DML语言：数据操作语言  

- insert
- update
- delete

### 3.3添加

```sql
-- 插入数据	insert into 表名(字段名1,字段名2...) values(值1,值2...);
insert into `grade`(`gradename`) values('大四');
-- 由于主键自增我们可以省略（如果不写表字段，就会一一匹配）
-- 插入多个字段
insert into `grade`(gradename`) values('大二'),('大一');
```

### 3.4修改

```sql
-- 修改学员名称  update 表名 set column_name=value,[column_name=value,...] where 条件;
update `student` set `name`='小白' where id=1;
-- 不指定条件的情况下，会改动所有的表
update `student` set `name`='长江7号';
-- 修改多个属性,逗号隔开
update `student` set `name`='小黑',`email`='2000112@qq.com' where id=1;
```

- **column_name是数据库的列，尽量带上``**
- 筛选的条件，如果没有指定，则会修改所有的列
- value是一个具体的值，也可以是一个**变量**

```sql
UPDATE `student` SET `birthday`=CURRENT_TIME WHERE `name`='小白' AND `email` IS NOT NULL;
```

### 3.5删除

> delete命令

```sql
-- 删除指定数据	delete from 表名 where 条件
delete from `student` where id=1;
```

> truncate命令

作用：完全情况一个数据库表，表的结构和索引都不变

```sql
-- 清空student表	truncate table 表名
truncate table `student`;
```

> delete 和 truncate的区别

- 相同点：都能删除数据，都不会删除表结构
- 不同：
  - truncate 重新设置 自增列 计数器会归零
  - truncate 不会影响 事务

```sql
-- 测试delete和truncate的区别
CREATE TABLE IF NOT EXISTS `test` (
	`id` INT(4) NOT NULL AUTO_INCREMENT,
	`coll` VARCHAR(20) NOT NULL,
	PRIMARY KEY (`id`)
)ENGINE=INNODB DEFAULT CHARSET=utf8;

INSERT INTO `test`(`coll`) VALUES('123'),('adad'),('asdasd');

DELETE FROM `test`; -- 不会影响自增

INSERT INTO `test`(`coll`) VALUES('1'),('2'),('3');

TRUNCATE TABLE `test`; -- 自增会清零

INSERT INTO `test`(`coll`) VALUES('1'),('2'),('3');
```

了解即可：delete删除的问题，重启数据库，现象：

- InnoDB	自增列会从1开始（存在内存中，断电即失）
- MyISAM   继续从上一个自增量开始（存在文件中，不会丢失）

## 4.DQL查询数据(最重点)

### 4.1DQL

(Data Query Language：数据查询语言)

- 所有的查询操作都使用它	`select`
- 简单的查询，复杂的查询它都能做
- **数据库中最核心的语言，最重要的语句**
- 使用频率最高的语句

> select语法

```sql
SELECT [ALL | DISTINCT]
{* | table.* | [table.field1[AS alias1][,table.field2[AS alias2]][,...]]}
FROM table_name [AS table_alias]
	[LEFT | RIGHT | INNER JOIN table_name2] -- 联合查询
	[WHERE ...] -- 指定结果满足条件
	[GROUP BY ...] -- 指定结果按照哪几个字段来分组
	[HAVING] -- 过滤分组的记录必须满足的次要条件
	[ORDER BY ...] -- 指定查询记录按一个或多个条件排序
	[LIMIT offset,row_count] -- 指定查询的记录从哪条到哪条
```

准备阶段：

```sql
-- 练习查询所建的表
CREATE DATABASE IF NOT EXISTS `school`;

-- 创建一个school数据库
USE `school`;-- 创建学生表
DROP TABLE IF EXISTS `student`;
CREATE TABLE `student`(
	`studentno` INT(4) NOT NULL COMMENT '学号',
    `loginpwd` VARCHAR(20) DEFAULT NULL,
    `studentname` VARCHAR(20) DEFAULT NULL COMMENT '学生姓名',
    `sex` TINYINT(1) DEFAULT NULL COMMENT '性别，0或1',
    `gradeid` INT(11) DEFAULT NULL COMMENT '年级编号',
    `phone` VARCHAR(50) NOT NULL COMMENT '联系电话，允许为空',
    `address` VARCHAR(255) NOT NULL COMMENT '地址，允许为空',
    `borndate` DATETIME DEFAULT NULL COMMENT '出生时间',
    `email` VARCHAR (50) NOT NULL COMMENT '邮箱账号允许为空',
    `identitycard` VARCHAR(18) DEFAULT NULL COMMENT '身份证号',
    PRIMARY KEY (`studentno`),
    UNIQUE KEY `identitycard`(`identitycard`),
    KEY `email` (`email`)
)ENGINE=MYISAM DEFAULT CHARSET=utf8;

-- 创建年级表
DROP TABLE IF EXISTS `grade`;
CREATE TABLE `grade`(
	`gradeid` INT(11) NOT NULL AUTO_INCREMENT COMMENT '年级编号',
  `gradename` VARCHAR(50) NOT NULL COMMENT '年级名称',
    PRIMARY KEY (`gradeid`)
) ENGINE=INNODB AUTO_INCREMENT = 6 DEFAULT CHARSET = utf8;

-- 创建科目表
DROP TABLE IF EXISTS `subject`;
CREATE TABLE `subject`(
	`subjectno`INT(11) NOT NULL AUTO_INCREMENT COMMENT '课程编号',
    `subjectname` VARCHAR(50) DEFAULT NULL COMMENT '课程名称',
    `classhour` INT(4) DEFAULT NULL COMMENT '学时',
    `gradeid` INT(4) DEFAULT NULL COMMENT '年级编号',
    PRIMARY KEY (`subjectno`)
)ENGINE = INNODB AUTO_INCREMENT = 19 DEFAULT CHARSET = utf8;

-- 创建成绩表
DROP TABLE IF EXISTS `result`;
CREATE TABLE `result`(
	`studentno` INT(4) NOT NULL COMMENT '学号',
    `subjectno` INT(4) NOT NULL COMMENT '课程编号',
    `examdate` DATETIME NOT NULL COMMENT '考试日期',
    `studentresult` INT (4) NOT NULL COMMENT '考试成绩',
    KEY `subjectno` (`subjectno`)
)ENGINE = INNODB DEFAULT CHARSET = utf8;

-- 插入学生数据 其余自行添加 这里只添加了2行
INSERT INTO `student` (`studentno`,`loginpwd`,`studentname`,`sex`,`gradeid`,`phone`,`address`,`borndate`,`email`,`identitycard`)
VALUES
(1000,'123456','张伟',0,2,'13800001234','北京朝阳','1980-1-1','text123@qq.com','123456198001011234'),
(1001,'123456','赵强',1,3,'13800002222','广东深圳','1990-1-1','text111@qq.com','123456199001011233'),
(1002,'123456','周丹',0,2,'13800002345','上海浦东','1988-12-1','text234@qq.com','123456198001012345'),
(1003,'123456','杨文锐',1,3,'13800003456','上海静安','1997-6-1','text345@qq.com','123456199001013456'),
(1004,'123456','李贺',0,2,'13800004567','安徽合肥','1990-12-11','text456@qq.com','123456198001014567'),
(1005,'123456','韩江',1,3,'13800005678','广东广州','1993-7-9','text567@qq.com','123456199001015678'),
(1006,'123456','刘恒',0,2,'13800006789','北京朝阳','1987-10-21','text678@qq.com','123456198001016789'),
(1007,'123456','赵成',1,3,'13800009012','广东深圳','1996-2-4','text789@qq.com','123456199001017890'),
(1008,'123456','徐凤年',0,2,'13800000123','上海虹口','1989-10-1','text222@qq.com','123456198001011111'),
(1009,'123456','王仙之',1,3,'13800001111','江苏南京','1993-12-21','text111@qq.com','123456199001012222'),
(1010,'123456','徐荣祥',0,2,'13800003333','北京大兴','1997-4-1','text333@qq.com','123456198001013333'),
(1011,'123456','拓跋菩萨',1,3,'13800004444','浙江杭州','1996-5-1','text444@qq.com','123456199001014444'),
(1012,'123456','姜泥',0,2,'13800005555','江苏苏州','1987-9-1','text555@qq.com','123456198001015555'),
(1013,'123456','李存刚',1,3,'13800006666','江苏无锡','1995-10-21','text666@qq.com','123456199001016666');

-- 插入成绩数据  这里仅插入了一组，其余自行添加
INSERT INTO `result`(`studentno`,`subjectno`,`examdate`,`studentresult`)
VALUES
(1000,1,'2013-11-11 16:00:00',85),
(1000,2,'2013-11-12 16:00:00',70),
(1000,3,'2013-11-11 09:00:00',68),
(1000,4,'2013-11-13 16:00:00',98),
(1000,5,'2013-11-14 16:00:00',58),
(1001,1,'2013-11-11 16:00:00',85),
(1001,2,'2013-11-12 16:00:00',99),
(1001,3,'2013-11-11 09:00:00',79),
(1001,4,'2013-11-13 16:00:00',100),
(1002,5,'2013-11-14 16:00:00',58),
(1002,1,'2013-11-11 16:00:00',85),
(1002,2,'2013-11-12 16:00:00',70),
(1003,3,'2013-11-11 09:00:00',69),
(1003,4,'2013-11-13 16:00:00',98),
(1004,5,'2013-11-14 16:00:00',58),
(1004,1,'2013-11-11 16:00:00',86),
(1005,2,'2013-11-12 16:00:00',70),
(1005,3,'2013-11-11 09:00:00',68),
(1006,4,'2013-11-13 16:00:00',98),
(1007,5,'2013-11-14 16:00:00',58),
(1008,1,'2013-11-11 16:00:00',78),
(1008,2,'2013-11-12 16:00:00',70),
(1008,3,'2013-11-11 09:00:00',68),
(1008,4,'2013-11-13 16:00:00',90),
(1009,5,'2013-11-14 16:00:00',58),
(1009,1,'2013-11-11 16:00:00',85),
(1009,2,'2013-11-12 16:00:00',70),
(1009,3,'2013-11-11 09:00:00',66),
(1010,4,'2013-11-13 16:00:00',98),
(1010,5,'2013-11-14 16:00:00',58),
(1010,1,'2013-11-11 16:00:00',85),
(1010,2,'2013-11-12 16:00:00',77),
(1011,3,'2013-11-11 09:00:00',68),
(1012,4,'2013-11-13 16:00:00',98),
(1013,5,'2013-11-14 16:00:00',58);

-- 插入年级数据
INSERT INTO `grade` (`gradeid`,`gradename`) VALUES(1,'大一'),(2,'大二'),(3,'大三'),(4,'大四'),(5,'预科班');

-- 插入科目数据
INSERT INTO `subject`(`subjectno`,`subjectname`,`classhour`,`gradeid`)VALUES
(1,'高等数学-1',110,1),
(2,'高等数学-2',110,2),
(3,'高等数学-3',100,3),
(4,'高等数学-4',130,4),
(5,'C语言-1',110,1),
(6,'C语言-2',110,2),
(7,'C语言-3',100,3),
(8,'C语言-4',130,4),
(9,'Java程序设计-1',110,1),
(10,'Java程序设计-2',110,2),
(11,'Java程序设计-3',100,3),
(12,'Java程序设计-4',130,4),
(13,'数据库结构-1',110,1),
(14,'数据库结构-2',110,2),
(15,'数据库结构-3',100,3),
(16,'数据库结构-4',130,4),
(17,'C#基础',130,1);
```

### 4.2查询指定字段

```sql
-- 查询全部的学生
SELECT * FROM student;

-- 查询指定字段	select 字段,.. from 表名
SELECT `studentno`,`studentname` FROM student;

-- 别名，给结果起一个名字 as  可以给字段起别名，也可以给表起别名
SELECT `studentno` AS 学号,`studentname` AS 学生姓名 FROM student AS s;

-- 函数 Concat(a,b)
SELECT CONCAT('姓名：',`studentname`) AS 学生姓名 FROM student;
```

> 去重 distinct

```sql
-- 查询一下有哪些同学参加了考试,成绩
SELECT * FROM result; -- 查询全部的考试成绩表
SELECT `studentno` FROM result; -- 查询有哪些同学参加了考试
SELECT DISTINCT `studentno` FROM result; -- 发现重复数据，去重
```

> 数据库的列（表达式）
>
> 数据库中的表达式：文本值，列，null，函数，计算表达式，系统变量...

```sql
-- select 表达式 from 表
SELECT VERSION()-- 查询系统版本号（函数）
SELECT 100*3-1 AS 计算结果 -- 用来计算（计算表达式）
SELECT @@auto_increment_increment -- 查询自增的步长（变量）

-- 学院考试成绩+1分查看
SELECT `studentno`,`studentresult` FROM result; -- 原成绩
SELECT `studentno`,`studentresult`+1 AS 更改后的成绩 FROM result; -- +1分后的数据
```

### 4.3where条件字句

作用：检索数据中**符合条件**的值

搜索的条件由一个或者多个表达式组成，结果为 **布尔值**

> 逻辑运算符

| 运算符  | 语法                   | 描述                       |
| ------- | ---------------------- | -------------------------- |
| and &&  | a and b       a && b   | 逻辑与，两个都为真，才为真 |
| or \|\| | a or b          a\|\|b | 逻辑或，一个为真即为真     |
| not !   | not a           !a     | 逻辑非                     |

**尽量使用英文字符**

```sql
-- ================  where  =================
-- 查询考试成绩在 95~100 分之间
SELECT `studentno`,`studentresult` FROM result 
WHERE `studentresult`>=95 AND `studentresult`<=100; -- and

SELECT `studentno`,`studentresult` FROM result 
WHERE `studentresult`>=95 && `studentresult`<=100; -- &&

SELECT `studentno`,`studentresult` FROM result
WHERE `studentresult` BETWEEN 95 AND 100; -- between... and ... 区间查询

-- 除了1000学号以外的成绩
SELECT `studentno`,`studentresult` FROM result 
WHERE `studentno`!=1000; -- !

SELECT `studentno`,`studentresult` FROM result 
WHERE NOT `studentno`=1000; -- not
```

> 模糊查询：比较运算符

| 运算符      | 语法                | 描述                                |
| ----------- | ------------------- | ----------------------------------- |
| is null     | a is null           | 如果操作符为null，结果为真          |
| is not null | a is not null       | 如果操作符不为null，结果为真        |
| between     | a between b and c   | 若a在b和c之间，则结果为真           |
| **like**    | a like b            | SQL匹配，如果a匹配b，则结果为真     |
| **in**      | a in （a1,a2,a3...) | 假设a在a1,或者a2...某个中，结果为真 |

```sql
-- =================  模糊查询  ===============
-- 查询姓徐的同学  like结合 %(代表0到任意个字符)  _(表示一个字符)
SELECT `studentno`,`studentname` FROM student 
WHERE `studentname` LIKE '徐%';

-- 查询姓刘的同学 名字后面只有一个字的
SELECT `studentno`,`studentname` FROM student
WHERE `studentname` LIKE '刘_';
```

```sql
-- ========= in（具体的一个或多个值） ===========
-- 查询 1001,1002,1003学号成员
SELECT `studentno`,`studentname` FROM student
WHERE `studentno` IN (1001,1002,1003);

-- 查询 在北京的学生
SELECT `studentno`,`studentname`,`address` FROM student
WHERE `address` IN ('北京朝阳','上海浦东');
```

```sql
-- ============= null     not null =============
-- 查询地址为空的学生 null ''
SELECT `studentno`,`studentname` FROM student 
WHERE `address`='' OR `address` IS NULL; 
```

### 4.4联表查询

![1.jpg](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/%E8%BF%9E%E8%A1%A8%E6%9F%A5%E8%AF%A2.jpeg)

```sql
-- ===============  联表查询 join  ===================
-- 查询参加了考试的同学（学号，姓名，科目编号，分数）
/*思路：
1. 分析需求，分析查询的字段来自哪些表（连接查询）
2. 确定使用哪种查询（7种）
确定交叉点（这两个表中哪些数据是相同的）
判断的条件：学生表中的studentno = 成绩表中的studentno
*/
-- inner join
SELECT s.`studentno`,`studentname`,`subjectno`,`studentresult` 
FROM student AS s
INNER JOIN result AS r -- 两张表联接起来了
ON s.`studentno` = r.`studentno`; -- 确定了交叉点

-- rigth join
SELECT s.`studentno`,`studentname`,`subjectno`,`studentresult`
FROM student AS s
RIGHT JOIN result AS r
ON s.`studentno` = r.`studentno`;

-- left join
SELECT s.`studentno`,`studentname`,`subjectno`,`studentresult`
FROM student AS s
LEFT JOIN result AS r
ON s.`studentno` = r.`studentno`;

-- 查询缺考的同学
SELECT s.`studentno`,`studentname`,`subjectno`,`studentresult`
FROM student AS s
LEFT JOIN result AS r
ON s.`studentno` = r.`studentno`
WHERE `studentresult` IS NULL;
```

| 操作                     | 描述                                           |
| ------------------------ | ---------------------------------------------- |
| inner join（内连接）     | 如果表中至少有一个匹配，就返回行               |
| left join（左[外]连接）  | **会从左表中返回所有的值，即使右表中没有匹配** |
| right join（右[外]连接） | **会从右表中返回所有的值，即使左表中没有匹配** |

注意：

- `等值连接`和`内连接`的效果一样，但是开发中**建议使用内连接**，因为**等值连接在查询的时候会将2个表会先进行笛卡尔乘积运算，生成一个新表格**，占据在电脑内存里，当表的数据量很大时，很耗内存，这种方法效率比较低；**内连接查询时会将2个表根据共同ID进行逐条匹配，不会出现笛卡尔乘积的现象**，效率比较高。

> 自连接

准备数据:

```sql
-- ===================  自连接  ======================
USE school;
CREATE TABLE  IF NOT EXISTS `category`( 
	`categoryid` INT(3) NOT NULL COMMENT '主题id', 
	`pid` INT(3) NOT NULL COMMENT '父id', 
	`categoryname` VARCHAR(10) NOT NULL COMMENT '主题名字', 
	PRIMARY KEY (`categoryid`)
) ENGINE=INNODB DEFAULT CHARSET=utf8; 

INSERT INTO `category` (`categoryid`, `pid`, `categoryname`) 
VALUES (2, 1, '信息技术'),
(3, 1, '软件开发'),
(5, 1, '美术设计'),
(4, 3, '数据库'),
(8, 2, '办公信息'),
(6, 3, 'web开发'),
(7, 5, 'ps技术');
```

自己的表和自己的表连接，核心：**一张表拆为两张一样的表即可**

```sql
-- 查询父子信息 父类下面包含的子类
SELECT f.`categoryname` AS 父栏目,s.`categoryname` AS 子栏目
FROM `category` AS f,`category` AS s
WHERE f.`categoryid` = s.`pid`;
```

<img src="https://raw.githubusercontent.com/zmk-c/blogImages/master/img/%E8%87%AA%E8%BF%9E%E6%8E%A5.png" alt="image-20210508084657256" style="zoom:67%;" />

### 4.5分页和排序

```sql
-- ===============  分页limit和排序order by ====================
-- 排序：升序 asc ，降序 desc

-- 根据查询的结果 按照成绩降序排序
-- 语法：order by 字段 desc
SELECT s.`studentno`,`studentname`,`subjectname`,`studentresult`
FROM student AS s
INNER JOIN result AS r
ON s.`studentno` = r.`studentno`
INNER JOIN SUBJECT AS sub
ON r.`subjectno` = sub.`subjectno`
WHERE `subjectname`='高等数学-1'
ORDER BY `studentresult` DESC;

-- 分页，每页只显示五条数据
-- 语法：limit 起始值，页面的大小
-- 网页应用：当前，总的页数，页面的大小

-- 第一页  limit 0,5	(1-1)*5
-- 第二页  limit 5,5	(2-1)*5
-- 第三页  limit 10,5   (3-1)*5
-- [pageSize：页面大小]
-- [(n-1)*pageSize：为起始值，n为当前页]	
-- [数据总数/页面大小=总页数]
SELECT s.`studentno`,`studentname`,`subjectname`,`studentresult`
FROM student AS s
INNER JOIN result AS r
ON s.`studentno` = r.`studentno`
INNER JOIN SUBJECT AS sub
ON r.`subjectno` = sub.`subjectno`
WHERE `subjectname`='高等数学-1'
ORDER BY `studentresult` DESC
LIMIT 0,5;
```

```sql
-- 思考题：查询 JAVA第一学年 课程成绩排名前十的同学，并且分数要大于80 学生的信息（学号，姓名，课程名称，分数）
SELECT s.`studentno`,`studentname`,`subjectname`,`studentresult`
FROM student AS s
INNER JOIN result AS r
ON s.`studentno` = r.`studentno`
INNER JOIN `subject` AS sub
ON r.`subjectno` = sub.`subjectno`
WHERE `subjectname`='Java程序设计-1' AND `studentresult`>80
ORDER BY `studentresult` DESC
LIMIT 0,10;
```

### 4.6子查询

```sql
-- 查询 数据库结构-1 的所有考试结果（学号，科目编号，成绩） 降序排序
-- 第一种方式：使用连接查询
SELECT `studentno`,r.`subjectno`,`studentresult`
FROM `result` AS r
INNER JOIN `subject` AS sub
ON r.`subjectno`=sub.`subjectno`
WHERE sub.`subjectname`='数据库结构-1'
ORDER BY `studentresult` DESC;

-- 第二种方式：使用子查询(由里及外)
SELECT `studentno`,`subjectno`,`studentresult`
FROM `result`
WHERE `subjectno` = (SELECT `subjectno` FROM `subject` WHERE `subjectname`='数据库结构-1')
ORDER BY `studentresult` DESC;
```

## 5.MySQL函数

> 官网：[MySQL :: MySQL 5.6参考手册:: 12.1 SQL函数和运算符参考](https://dev.mysql.com/doc/refman/5.6/en/sql-function-reference.html)

### 5.1常用函数

```sql
-- ================   常用函数  ====================
-- 数学运算
SELECT ABS(-8); -- 绝对值
SELECT CEILING(9.5); -- 向上取整
SELECT FLOOR(9.5); -- 向下取整
SELECT RAND(); -- 返回一个 0~1 之间的小数
SELECT SIGN(10); -- 判断一个数的符号	负数返回-1 正数返回1 0返回0

-- 字符串函数
SELECT CHAR_LENGTH('即使再小的帆也能远航'); -- 计算字符串长度
SELECT CONCAT('我','爱中国'); -- 拼接字符串
SELECT INSERT('我爱编程hello world',1,2,'超级爱'); -- 在指定位置插入长度的字符串，mysql从1开始
SELECT LOWER('GOLANG'); -- 小写
SELECT UPPER('golang'); -- 大写
SELECT INSTR('中国万岁，中国人民万岁','中国'); -- 返回第一次出现的子串的索引
SELECT REPLACE('我爱编程hello world','hello','world');-- 替换出现的指定字符串
SELECT SUBSTR('我爱编程hello world',2,5); -- 返回指定的子串(源字符串，截取的位置，截取的长度)
SELECT REVERSE('我爱编程'); -- 翻转字符串

-- 查询姓徐的同学，将徐姓 改为周姓
SELECT REPLACE(studentname,'徐','周') FROM student 
WHERE `studentname` LIKE '徐%';


-- 时间和日期函数
SELECT CURRENT_DATE(); -- 获取当前日期
SELECT NOW(); -- 获取当前的时间
SELECT LOCALTIME(); -- 获取本地时间
SELECT SYSDATE(); -- 获取系统时间 
SELECT UNIX_TIMESTAMP(); -- 获取当前时间戳
SELECT YEAR(NOW()); -- 年
SELECT MONTH(NOW()); -- 月
SELECT DAY(NOW()); -- 日
SELECT HOUR(NOW()); -- 小时
SELECT MINUTE(NOW()); -- 分钟
SELECT SECOND(NOW()); -- 秒

-- 系统
SELECT SYSTEM_USER(); -- 系统用户
SELECT VERSION(); -- 系统版本
```

### 5.2聚合函数(常用)

```sql
-- ===============  聚合函数  ==================
-- 都能够统计 表中的数据
SELECT COUNT(`studentname`) FROM student; -- count(指定列)  会忽略所有的null值   列名为主键使用count(列名)比count(1)快 
 
SELECT COUNT(*) FROM student; -- count(*)  不会忽略null值  包括了所有的列，相当于行数   如果表只有一个字段，则count(*)最优
SELECT COUNT(1) FROM student; -- count(1)  不会忽略null值  忽略所有列，用1代表代码行   列名不为主键，count(1)会比count(列名)快


SELECT SUM(`studentresult`) AS 总和 FROM result; -- 获取学生分数总和

SELECT AVG(`studentresult`) AS 平均分 FROM result; -- 获取平均分

SELECT MIN(`studentresult`) AS 最低分 FROM result; -- 获取最低分

SELECT MAX(`studentresult`) AS 最高分 FROM result; -- 获取最高分


-- 查询不同课程的平均分，最高分，最低分，且平均分>=80
-- 核心：根据不同课程分组
SELECT `subjectname`,AVG(`studentresult`) AS 平均分,MAX(`studentresult`) AS 最高分,MIN(`studentresult`) AS 最低分
FROM `subject` AS sub
INNER JOIN `result` AS r
ON r.`subjectno` = sub.`subjectno`
GROUP BY r.`subjectno`
HAVING 平均分>=80;
```

## 6.事务

### 6.1什么是事务

==要么都成功，要么都失败==，将一组SQL放在同一个批次中去执行

> 事务原则：ACID原则

`A：原子性(atomicity)`

一个事务要么全部提交成功，要么全部失败回滚，不能只执行其中的一部分操作，这就是事务的原子性。

`C：一致性(consistency)`

一个事务在执行之前和执行之后，数据的完整性要保持一致。

`I：隔离性(isolation)`

事务的隔离性是多个用户并发访问数据库时，数据库为每一个用户开启的事务不能被其他事务的操作所干扰，事务之间要隔离。

**隔离所导致的问题：**

1. **脏读**：读到其他事务未提交的数据 
2. **不可重复读**：前后读取的记录内容不一致 
3. **幻读**：前后读取的记录数不一致

**隔离的级别：**

1. **读未提交(Read Uncommitted)**：可以读取其他事务修改并且未提交的数据。但是会造成“脏读”，“幻读”，“不可重复读取”。 
2. **读提交(Read Committed)**：可以读取其他事务修改并提交的数据。避免了“脏读”，“但不能避免""幻读“和”不可重复读取”。(是大多主流数据库默认的事务等级) 
3. **可重复读(Repeatable Read)**：锁定了已经读取的数据，当前事务未提交前其他事物不予许修改。避免了“脏读“和”不可重复读取“的情况，但不能避免“幻读”。（带来了更多的性能损失） 
4. **串行化(Serializable)**：读取前锁定所有要读取的数据，当前事务提交前其他事务不允许修改。（最严格级别，事务串行执行，资源消耗最大）

`D：持久性(durability)`

事务没有提交，则恢复到原状；事务已经提交，则持久化到数据库。

### 6.2事务操作

```sql
-- =================  事务  ====================
-- mysql 是默认开启事务自动提交的
SET autocommit = 0 -- 关闭
SET autocommit = 1 -- 开启(默认)


-- 手动处理事务
SET autocommit = 0 -- 关闭自动提交

-- 事务开启
START TRANSACTION -- 标记一个事务的开启，从这个之后的sql都在同一个事务中

INSERT xxx
INSERT xxx

-- 提交：持久化（成功！）
COMMIT

-- 回滚：回到原来的样子（失败）
ROLLBACK

-- 事务结束
SET autocommit = 1 -- 开启自动提交

-- 了解 
SAVEPOINT 保存点名 -- 设置一个保存点
ROLLBACK TO SAVEPOINT 保存点名 -- 回滚到保存点
RELEASE SAVEPOINT 保存点名 -- 删除保存点 
```

**转账案例：**

```sql
-- ==============  转账案例 =================
-- 创建数据库
CREATE DATABASE shop CHARACTER SET utf8 COLLATE utf8_general_ci;
USE shop;

-- 创建表
CREATE TABLE IF NOT EXISTS `account`(
	`id` INT(4) NOT NULL AUTO_INCREMENT,
	`name` VARCHAR(20) NOT NULL,
	`money` DECIMAL(9,2) NOT NULL,
	PRIMARY KEY (`id`)
)ENGINE=INNODB DEFAULT CHARSET=utf8;

-- 插入数据
INSERT INTO `account`(`name`,`money`) VALUES('A',2000.00),('B',1000.00);
```

查看表中数据如下：

<img src="https://raw.githubusercontent.com/zmk-c/blogImages/master/img/transaction1.png" alt="image-20210512213350098" style="zoom: 67%;" />

下面开始模拟事务，语句一条一条执行

```sql
-- 模拟事务
SET autocommit = 0; -- 关闭自动提交
START TRANSACTION; -- 开启一个事务

UPDATE `account` SET `money`=`money`-500 WHERE `name`='A'; -- A减500
UPDATE `account` SET `money`=`money`+500 WHERE `name`='B'; -- B加500
```

执行完更新操作后：

<img src="https://raw.githubusercontent.com/zmk-c/blogImages/master/img/transaction2.png" alt="image-20210512213418523" style="zoom: 67%;" />

```sql
ROLLBACK; -- 失败回滚
```

倘若失败，执行回滚后，数据恢复原来的样子：

<img src="https://raw.githubusercontent.com/zmk-c/blogImages/master/img/transaction3.png" alt="image-20210512213451198" style="zoom: 67%;" />

```sql
COMMIT; -- 成功提交事务
SET autocommit = 1; -- 恢复默认提交 
```

若执行成功，点击commit提交，则被持久化保存了。再回滚也没用了

<img src="https://raw.githubusercontent.com/zmk-c/blogImages/master/img/transaction4.png" alt="image-20210512213548029" style="zoom: 67%;" />

## 7.索引

> MySQL官方对索引的定义为：索引（Index）是帮助MySQL高效获取数据的数据结构。提取句子主干，就可以得到索引的本质：索引是数据结构。

### 7.1索引的分类

> 索引是在存储引擎中实现的，也就是说不同的存储引擎，会使用不同的索引
>
> - MyISAM和InnoDB存储引擎：只支持BTREE索引， 也就是说默认使用BTREE，不能够更换
>
> - MEMORY/HEAP存储引擎：支持HASH和BTREE索引

索引分成四类来讲

- 单列索引(普通索引，唯一索引，主键索引)
  - **普通索引**：MySQL中基本索引类型，没有什么限制，允许在定义索引的列中插入重复值和空值，纯粹为了查询数据更快一点。
  - **唯一索引**：索引列中的值必须是唯一的，但是允许为空值。
  - **主键索引**：是一种特殊的唯一索引，不允许有空值，主键不可重复。
- 组合索引：在表中的多个字段组合上创建的索引，只有在查询条件中使用了这些字段的左边字段时，索引才会被使用，使用组合索引时遵循`最左前缀集合`。
- 全文索引：MySQL及以后的版本，MyISAM和InnoDB存储引擎军均支持全文索引，全文索引就是在一堆文字中，通过其中的某个关键字等，就能找到该字段所属的记录行，只能在`CHAR，VARCHAR，TEXT`类型字段上使用全文索引。

### 7.2索引的使用

```sql
-- 索引的使用
-- 1.在创建表的时候给字段增加索引
-- 2.创建完毕后，增加索引


-- 显示所有的索引信息 
SHOW INDEX FROM student; -- 可以通过添加 \G 来格式化输出信息。

-- 增加一个全文索引 索引名(列名)
ALTER TABLE `student` ADD FULLTEXT INDEX `studentname` (`studentname`);
```

![image-20210512213710777](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/index1.png)

**在一百万条数据下测试索引的巨大作用：**

```sql
-- ===============  在一百万条数据下测试索引的巨大作用  =================
-- 创建一个新表  这里除了主键没有其他索引
CREATE TABLE `app_user`(
	`id` BIGINT(20) UNSIGNED NOT NULL AUTO_INCREMENT,
	`name` VARCHAR(50) DEFAULT '' COMMENT '用户昵称',
	`email` VARCHAR(50) NOT	NULL COMMENT '用户邮箱',
	`phone` VARCHAR(20) DEFAULT '' COMMENT '手机号',
	`gender` TINYINT(4) UNSIGNED DEFAULT '0' COMMENT '性别(0:男生,1:女生)',
	`password` VARCHAR(100) NOT NULL COMMENT '密码',
	`age` TINYINT(4) DEFAULT '0' COMMENT '年龄',
	`create_time` DATETIME DEFAULT CURRENT_TIMESTAMP,
	`update_time` TIMESTAMP NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '自动更新',
	PRIMARY KEY (`id`)	
)ENGINE=INNODB DEFAULT CHARSET=utf8 COMMENT='app用户表';

-- 函数创建一百万条数据
DELIMITER $$ -- 写函数之前必须要写，标志
CREATE FUNCTION mock_data() -- 函数名
RETURNS INT -- 返回值类型
BEGIN
	DECLARE	num INT DEFAULT 1000000; -- 声明变量
	DECLARE i INT DEFAULT 0; 
	
	WHILE i < num DO
		-- 插入语句
		INSERT INTO `app_user`(`name`,`email`,`phone`,`gender`,`password`,`age`)
		VALUES(CONCAT('用户',i),CONCAT(FLOOR(RAND()*1000000000),'qq.com'),CONCAT('13',FLOOR(RAND()*(999999999-100000000)+100000000)),
		FLOOR(RAND()*2),UUID(),FLOOR(RAND()*100));
		SET i = i+1;
	END WHILE;
	RETURN i;
END;

SELECT mock_data(); -- 执行创建函数

/*truncate table `app_user`; -- 清空表  使自增从0开始*/
/*drop function mock_data; -- 删除函数 drop function 函数名;*/
```

执行mock_data()函数插入的一百万条数据如下（一部分展示）：

![image-20210512213841847](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/index2.png)

下面测试不使用索引和使用索引的差别：

```sql
-- 不使用索引进行检索
SELECT * FROM `app_user` WHERE `name`='用户99999'; -- 0.461 sec
SELECT * FROM `app_user` WHERE `name`='用户99999'; -- 0.463 sec
SELECT * FROM `app_user` WHERE `name`='用户99999'; -- 0.458 sec

-- 查看执行况
EXPLAIN SELECT * FROM `app_user` WHERE `name`='用户99999'; 
```

可以看到查询`用户99999`，MySQL在数据库中一共查询了`992133`条数据，平均耗时`0.4s`

![image-20210512214149783](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/index_unuse.png)

下面使用索引查询

```sql
-- 创建索引 create index 索引名 on 表(字段)
CREATE INDEX id_app_user_name ON app_user(`name`); -- 3.226 sec 为一百万条数据都创建了索引

-- 使用索引查询
SELECT * FROM `app_user` WHERE `name`='用户99999'; -- 0.010 sec 可以看到瞬间提高了近40倍的效率
-- 查看执行情况
EXPLAIN SELECT * FROM `app_user` WHERE `name`='用户99999'; 
```

可以看到，只需要查询`1`条数据即其本身。耗时`0.01s`，使用索引效率明显提高了。

![image-20210512213955905](https://raw.githubusercontent.com/zmk-c/blogImages/master/img/index_use.png)

### 7.3索引原则

- 索引不是越多越好
- 不需要对经常变动的数据加索引
- 小数据量的表不需要加索引
- 索引一般加在常用来查询的字段上

> [CodingLabs - MySQL索引背后的数据结构及算法原理](http://blog.codinglabs.org/articles/theory-of-mysql-index.html)

## 8.权限管理

### 8.1用户管理

```sql
-- =============  权限管理 ================
-- 创建用户 create user 用户名 identified by 密码
CREATE USER xiaobai IDENTIFIED BY '123456';
-- 查询创建的用户
SELECT * FROM mysql.`user` ;
-- 修改密码 (修改当前用户密码)
SET PASSWORD = PASSWORD('111111');
-- 修改密码 (指定用户密码)
SET PASSWORD FOR xiaobao = PASSWORD('111111');

-- 重命名 rename user 旧名字 to 新名字
RENAME USER xiaobai TO dabai;

-- 用户授权 grant all privileges全部的权限 on 库.表 to 用户
GRANT ALL PRIVILEGES ON *.* TO dabai; 
GRANT ALL PRIVILEGES ON *.* TO 'dabai'@'%';

-- 查看权限
SHOW GRANTS FOR dabai;

-- 撤销权限 revoke 哪些权限 在哪个库 给谁撤销
REVOKE ALL PRIVILEGES ON *.* FROM dabai;
```

### 8.2数据库备份

为什么要备份：

- 保证重要的数据不丢失
- 数据转移

MySQL数据库备份的方式

- 直接拷贝物理文件
- 使用命令行导出  `mysqldump`

```sql
-- mysqldump -h主机 -u用户名 -p密码 数据库 表名[表名2...] > 物理磁盘位置/文件名
mysqldump -hlocalhost -uroot -p123456 school student > D:/a.sql;

-- 导入
-- 登录的情况下，切换到指定数据库
-- source 备份文件
source d:/a.sql;

-- mysql -u用户名 -p密码 库名<备份文件
mysql -uroot -proot school<d:/a.sql;
```

## 9.规范数据库设计

### 9.1为什么需要设计

==当数据库比较复杂的时候，我们就需要设计了==

**糟糕的数据库设计**：

- 数据冗余，浪费空间
- 数据库插入删除都会很麻烦，异常 【屏蔽使用物理外键】
- 程序的性能差

**良好的数据库设计**：

- 节省内存空间
- 保证数据库的完整性
- 方便开发系统

**软件开发中，关于数据库的设计**：

- 分析需求：分析业务和需要处理的数据库需求
- 概要设计：设计关系图E-R图

**设计数据库的步骤**：（个人博客）

- 收集信息，分析需求
  - 用户表（用户登录注销，用户的个人信息，写博客，创建分类）
  - 分类表（文章分类，谁创建的）
  - 文章表（文章的信息）
  - 评论表
  - 友链表（友链信息）
  - 自定义表（系统信息，某个关键的字，或者一些主字段）
- 标识实体（把需求落地到每个字段）
- 标识实体之间的关系
  - 写博客：user->blog
  - 创建分类：user->category
  - 关注：user->user
  - 友链：links
  - 评论：user-user-blog

### 9.2三大范式

**第一范式（1NF）：**

​	要求数据库的每一列都是不可分割的原子数据项。

**第二范式（2NF）：**

​	前提必须满足第一范式。

​	第二范式中的每张表只描述一件事。

**第三范式（3NF）：**

​	前提必须满足第一范式和第二范式。

​	第三范式消除依赖，保证数据表中的每一列数据都和主键直接相关，不能间接相关。

> **规范性和性能的问题**：关联查询的表不得超过三张

- 考率商业化的需求和目标（成本和用户体验）数据库的性能更加重要
- 在规范性能的问题的时候，需要适当考虑一下规范性！
- 故意给某些表增加一些冗余的字段。（从多表查询中变为单表查询）
- 故意增加一些计算列（从大数据量降低为小数据量的查询：索引）

## 结束

MySQL基础记录到此结束啦:laughing: ~ 感谢阅读！

> MySQL视频连接：[【狂神说Java】MySQL最新教程通俗易懂_哔哩哔哩 (゜-゜)つロ 干杯~-bilibili](https://www.bilibili.com/video/BV1NJ411J79W)