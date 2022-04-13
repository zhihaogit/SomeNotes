# MYSQL高级

## Linux安装 mysql

### 安装 mysql

1. 卸载预装mysql

	```bash
	rpm -qa | grep -i mysql
	rpm -e mysql-libs-x.x.x-xx --nodeps
	```

2. 上传 mysql安装包

3. 解压 mysql安装包

	```bash
	mkdir mysql
	tar -xvf Mysql-x.x.x-xx.tar -C /root/mysql
	```

4. 安装依赖包

	```bash
	yum -y install libiao.so.1 libgcc_s.so.1 libstdc++.so.6 libncurses.so.5 --setopt=protected_multilib=false
	yum update libstdc++-4.4.7-4.e16.x86_64
	```

5. 安装 mysql-client

	```bash
	rpm -ivh Mysql-client-5.6.22-1.e16.i686.rpm
	```

6. 安装 mysql-server

   ```bash
   rpm -ivh Mysql-server-5.6.22-1.e16.i686.rpm
   ```
   

### 启动 mysql服务

```bash
service mysql start
service mysql stop
service mysql status
service mysql restart
```

### 登录 mysql

mysql安装完成后，会自动生成一个随机的密码，并且保存在一个密码文件中： `/root/.mysql_secrect`

```bash
# 登录 mysql
mysql -u root -p
# 修改密码
set password = password('somepassword');
# 授权远程访问
grant all privileges on *.* to 'root' @'%' identified by 'somepassword';
flush privileges;
# 查看并关闭系统防火墙
service iptables status
service iptables stop
```

## 索引

### 简介

mysql索引的官方定义：索引（index）是帮助 mysql高效获取数据的数据结构（有序）。在数据之外，数据库系统还维护着满足特定查找算法的数据结构，这些数据结构以某种方式引用（指向）数据，这样就可以在这些数据结构上实现高级查找算法，这种数据结构就是`索引`。如下图所示：

![索引示意图](https://zion-bucket1.obs.cn-north-4.myhuaweicloud.com/images/2385455a525e09afbd962313190f2ea1be880d4b78129c62f2cadb8a8146b632-2022-04-13-68d60703da96b1564678be287915cbe0-bf6fe7.png)

### 优劣势

#### 优势

1. 类似于书籍的目录索引，提高数据检索的效率，降低数据库的IO成本
2. 通过索引列队数据进行排序，降低数据排序的成本，降低 CPU的消耗

#### 劣势

1. 实际上索引也是一张表，该表中保存了主键与索引字段，并指向实体类的记录，所以索引列也是要占用空间的
2. 虽然索引大大提高了查询效率，同时也降低更新表的速度，如对表进行 Insert, update, delete。因为更新表时，mysql不仅要保存数据，还要保存一下索引文件每次更新添加了索引列的字段，都会调整因为更新所带来的键值变化后的索引信息

### 索引结构

索引是在 mysql的存储引擎层中实现的，而不是在服务器层实现的。所以每种存储引擎的索引都不一定完全相同，也不是所有的存储引擎都支持所有的索引类型的。mysql目前提供了以下 4种索引：

- `BTREE`索引：最常见的索引类型，大部分索引都支持B树索引

- `Hash`索引：只有 Memory引擎索引，使用场景简单

- `R-TREE`索引（空间索引）：空间索引是 mylsam引擎的一个特殊索引类型，主要用于地理空间数据类型，通常使用较少

- `Full-text`索引（全文索引）：全文索引是 mylsam的一个特殊索引类型，主要用于全文索引，InnoDB从 mysql5.6版本开始支持全文索引

  **MyISAM、InnoDB、Memory三种存储引擎对各种索引类型的支持情况**

| 索引      | InnoDB引擎    | MyISAM引擎 | Memory引擎 |
| --------- | ------------- | ---------- | ---------- |
| BTREE     | 支持          | 支持       | 支持       |
| HASH      | 不支持        | 不支持     | 支持       |
| R-TREE    | 不支持        | 支持       | 不支持     |
| Full-text | 5.6版本后支持 | 支持       | 不支持     |

平常所说的索引，如果没有特别说明，都指 B+树（多路搜索树，并不一定是二叉的）结构组织的索引。其中聚集索引、复合索引、前缀索引、唯一索引默认都是使用 B+tree树索引，统称为索引。

#### BTREE结构

BTREE又叫多路平衡搜索树，一棵 m叉的 BTREE特性如下：

- 树种每个节点最多包含 m个孩子
- 除根节点与叶子节点外，每个节点至少有 `ceil(m / 2)`个孩子
- 若根节点不是叶子节点，则至少有两个孩子
- 所有的叶子节点都在同一层
- 每个非叶子节点由 n个 key与 n + 1个指针组成，其中 `ceil(m / 2) - 1<= n <= n -1`

![BTREE](https://zion-bucket1.obs.cn-north-4.myhuaweicloud.com/images/8e8d8a90ec4b92bdedfea267735f292f34a30b9e25995a3f84d225552665ab6e-20220413101704991-2022-04-13-bbd1f0b691a866c7f1e978fba04f79f3-9668db.png)

BTREE树与二叉树相比，查询效率更高。因为对于相同的数据量来说，BTREE的层级结构比二叉树小，因此搜索速度快。

#### B+TREE结构

B+TREE是 BTREE的变种，其区别如下：

- n叉 B+TREE最多含有 n个 key，而 BTREE最多含有 n - 1个 key
- B+TREE的叶子节点包含所有的 key信息，依 key大小顺序排列
- 所有的非叶子节点都可以看作是 key的索引部分

![B+TREE](https://zion-bucket1.obs.cn-north-4.myhuaweicloud.com/images/56aa4e5f50a0f6a4cbc386545fa05d1983d1a72ce6c0827332502a2c8d973f35-20220413101709291-2022-04-13-423f1d2f60b6672266157e04ffd9b6d6-e6bc4b.png)

由于B+TREE只有叶子节点保存 key信息，查询任何 key都要从 root走到叶子。所以 B+TREE的查询效率更加稳定

#### MYSQL中的B+TREE结构

mysql索引数据结构对经典的 B+TREE进行了优化。在原 B+TREE的基础上，增加一个指向相邻叶子节点的链表指针，就形成了带有顺序指针的 B+TREE，提高了区间（范围）访问的性能。

![mysql_B+TREE](https://zion-bucket1.obs.cn-north-4.myhuaweicloud.com/images/6662230a5c89103a937949a386128c2bb125541555c70e67dcce5e9ca48242b9-20220413101714802-2022-04-13-4f25b9b53537cf14e4ca886533d9718f-aa62aa.png)

### 索引分类

- 单值索引：即一个索引只包含单个列，一个表可以有多个单列索引
- 唯一索引：索引列的值必须唯一，但允许有空值
- 复合索引：即一个索引包含多个列

### 索引语法

#### 创建索引

```sql
CREATE [UNIQUE|FULLTEXT|SPATIAL] INDEX index_name [USING index_type] ON tbl_name(index_col_name, ...)

index_type: B+TREE
index_col_name: column_name[(length)][ASC | DESC]

-- 例子：为 city表中 city_name列创建普通索引（索引名称为 idx_city_name）
create index idx_city_name on city(city_name);

-- 创建复合索引
CREATE INDEX idx_name_email_status ON tb_seller(NAME, email, STATUS);
-- 相当于对 name创建了索引，
-- 			对 name, email创建了索引，
--      对 name, email, status创建了索引
```

#### 查看索引

```sql
show index from table_name;

-- 例子：查看 city表中的索引信息
show index from city\G;
```

#### 删除索引

```sql
DROP INDEX index_name ON tbl_name;

-- 例子：删除 city表上的索引名称为 idx_city_name的索引
drop index idx_city_name on city; 
```

#### ALTER命令

```sql
-- 添加一个主键，意味着索引值必须是唯一的，且不能为 NULL
alter table tbl_name add primary key(column_list);

-- 创建索引的值必须是唯一的（除了 NULL外，NULL可能会出现多次）
alter table tbl_name add unique index_name(column_list);

-- 添加普通索引，索引值可以出现多次
alter table tbl_name add index index_name(column_list);

-- 该语法指定了索引为 FULLTEXT，用于全文索引
alter table tbl_name add fulltext index_name(column_list);
```

### 索引设计原则

索引的设计可以遵循一些已有的原则，创建索引的时候尽量考虑符合这些原则，便于提升索引的使用效率，更高效的使用索引。

- 对查询频次较高，且数据量比较大的表建立索引
- 索引字段的选择，最佳候选列应当从 where子句的条件中提取，如果 where子句中的组合比较多，那么应当挑选最常用、过滤效果最好的列的组合
- 使用唯一索引，区分度较高，使用索引的效率较高
- 索引可以有效的提升查询数据的效率，但索引数量不是多多益善，索引较多，维护索引的代价自然也就水涨船高。对于插入、更新、删除等 DML操作比较频繁的表来说，索引过多，会引入相当高的维护代价，降低 DML操作的效率，增加相应操作的时间消耗。另外索引过多的话，mysql也会犯选择困难症，虽然最终仍然会找到一个可用的索引，但无疑提高了选择的代价
- 使用短索引，索引创建之后也是使用硬盘来存储的，因此提升索引的I/O效率，也可以提高总体的访问效率。假如构成索引的字段总长度比较短，那么在给定大小的存储块内可以存储更多的索引值，相应的可以有效的提升 mysql访问索引的 I/O效率
- 利用最左前缀，N个列组合而成的组合索引，那么相当于是创建了N个索引，如果查询时 where子句中使用了组成该索引的前几个字段，那么这条查询 sql可以利用组合索引来提升查询效率

## 视图

### 简介

视图（View）是一种虚拟存在的表。视图并不在数据库中实际存在，行和列数据来自定义视图的查询中使用的表，并且在使用视图时动态生成。通俗来讲，视图就是一条 SELECT语句执行后返回的结果集。所以在创建视图的时候，主要工作落在了创建这条 sql查询语句上。

视图相对于普通的表的优势：

- 简单：使用视图的用户完全不用关心后面对应的表结构、关联条件和筛选条件，对用户来说已经是过滤好的复合条件的结果集
- 安全：使用视图的用户只能访问被允许查询的结果集，对表的权限管理并不能限制到某个行某个列，但是通过视图就可以简单的实现
- 数据独立：一旦视图的结构确定了，可以屏蔽表结构变化对用户的影响，源表增加列队视图没有影响。源表修改列名，则可以通过修改视图来解决，不会造成对访问者的影响

### 创建和修改

#### 创建视图

```sql
CREATE [OR REPLACE] [ALGORITHM = {UNDEFUNED | MERGE | TEMPTABLE}] VIEW view_name [(column_list)] AS select_statement [WITH [CASCADED | LOCAL] CHECK OPTION]

-- 选项：
-- [WITH [CASCADED | LOCAL] CHECK OPTION]决定了是否允许更新数据使记录不再满足视图的条件
-- LOCAL: 只要满足本视图的条件就可以更新
-- CASCADED: 必须满足所有针对该视图的所有视图的条件才可以更新

-- 例子：创建一个 country和 city的联合查询视图
create view view_city_country as select c.*, t.country_name from city c, country t where c.country_id = t.country_id;
-- 查看该视图中的数据
select * from view_city_country;
-- 更新该视图中的数据
update view_city_country set city_name = '西安市' where city_id = 1;
```

#### 修改视图

```sql
ALTER [ALGORITHM = {UNDEFUNED | MERGE | TEMPTABLE}] VIEW view_name [(column_list)] AS select_statement [WITH [CASCADED | LOCAL] CHECK OPTION]
```

### 查看视图

从 mysql 5.1版本开始，使用 `show tables`命令不仅可以显示表的名字，同时也显示视图的名字，而不存在单独显示视图的 ~~show views~~命令。

同样在使用 `show table status`命令的时候，不但可以显示表的信息，同时也可以显示视图的信息。

`show create view view_city_country `查看创建 view_city_country视图的 select语句。

### 删除视图

```sql
DROP VIEW [IF EXISTS] view_name [, view_name] ... [RESTRICT | CASCADE]

-- 例子：删除 view_city_country视图
DROP VIEW view_city_country;
DROP VIEW IF EXISTS view_city_country;
```

## 存储过程和函数

### 简介

存储过程和函数是事先经过编译并存储在数据库中的一段 sql语句的集合，调用存储过程和函数可以简化应用开发人员的很多工作，减少数据在数据库和应用服务器之间的传输，对于提高数据处理的效率是有好处的。

存储过程和函数的区别在于函数必须有返回值，而存储过程没有。

函数：是一个有返回值的过程。

过程：是一个没有返回值的函数。

### 创建存储过程

```sql
CREATE PROCEDURE procedure_name ([proc_parameter [, ...]])
begin
	-- sql语句
end;

-- 例子：
delimiter $
create procedure pro_test1()
begin
	select 'Hello Mysql';
end$
delimiter ;
```

> DELIMITER
>
> 该关键字用来声明 SQL语句的分隔符，告诉解释器，该段命令是否已经结束了，mysql是否可以执行了。
>
> 默认情况下 delimeter是分号 ;
>
> 在命令行客户端中，如果有一行命令以分好结束，那么回车后，mysql将会执行该命令。

### 调用存储过程

```sql
call procedure_name();

-- 例子：注意 delimiter是什么
call pro_test();
```

### 查看存储过程

```sql
-- 查询 db_name数据库中的所有存储过程
select name from mysql.proc where db = 'db_name';

-- 查询存储过程的状态信息
show procedure status;

-- 查询某个存储过程的定义  
show create procedure test.pro_test1\G;
```

### 删除存储过程

```sql
DROP PROCEDURE [IF EXISTS] sp_name;
```

### 语法

存储过程是可以编程的，意味着可以使用变量，表达式，控制结构来完成比较复杂的功能。

##### 变量

###### DECLARE

```sql
-- 通过 DECLARE可以定义个局部变量，该变量的作用范围只能在 BEGIN...END块中
DECLARE var_name[,...] type [DEFAULT value]

-- 例子：
delimiter $
create procedure pro_test2()
begin
	declare num int default 5;
	select num+10;
end$
delimiter ;
```

###### SET

```sql
-- 直接赋值使用 SET，可以赋常量或者表达式
SET var_name = expr [, var_name = expr] ...

-- 例子：
delimiter $
create procedure pro_test3()
begin
	declare NAME varchar(20);
	set NAME = 'MYSQL';
	SELECT NAME;
end$
delimiter ;

-- 也可以使用 select...into方式进行赋值操作：
delimiter $
create procedure pro_test4()
begin
	declare countnum int;
	select count(*) into countnum from city;
	select countnum;
end$
delimiter ;
```

#####  IF条件判断

```sql
if search_condition then statement_list
	[elseif search_condition then statement_list] ...
	[else statement_list]
end if;

-- 例子：根据身高判断身材类型
delimiter $
create procedure pro_test5()
begin
	declare height int default 175;
	declare description varchar(50) default '';
	if height >= 180 then
		set description = 'nice';
	elseif height >= 170 and height < 180 then
		set description = 'standard';
	else
		set description = 'normal';
	end if;
	select concat('身高：', height, '，对应身材类型为：', description);
end$
delimiter ;
```

##### 传递参数

```sql
create procedure procedure_name([IN/OUT/INOUT] 参数名 参数类型)
...
```

###### IN输入

该参数可以作为输入，也就是需要调用方传入，默认

```sql
-- 例子：根据定义的身高变量，判断身材类型
delimiter $
create procedure pro_test6(in height int)
begin
	declare description varchar(50) default '';
	if height >= 180 then
		set description = 'nice';
	elseif height >= 170 and height < 180 then
		set description = 'standard';
	else
		set description = 'normal';
	end if;
	select concat('身高：', height, '，对应身材类型为：', description);
end$
delimiter ;
-- 调用
call pro_test6(170);
```

###### OUT输出

该参数作为输出，也就是该参数可以作为返回值

```sql
-- 例子：根据传入的身高变量，返回身材类型
delimiter $
create procedure pro_test7(in height int, out description varchar(50))
begin
	if height >= 180 then
		set description = 'nice';
	elseif height >= 170 and height < 180 then
		set description = 'standard';
	else
		set description = 'normal';
	end if;
end$
delimiter ;
-- 调用
call pro_test6(170, @description);
select @description;
```

> @description：这种在变量名称前面加上 “@“符号，叫做用户会话变量，在整个会话过程中，它都是有作用的，类似于全局变量
>
> @@global.sort_buffer_size: 这种在变量名称前面加上 ”@@”符号，叫做系统变量

###### INOUT 输入输出

既可以作为参数，也可以作为输出参数

##### case结构

```sql
-- 语法结构：
-- 方式一：
CASE case_value
	WHEN when_value THEN statement_list
	[WHEN when_value THEN statement_list] ...
	[ELSE statment_list]
END CASE;

-- 方式二：
CASE
	WHEN search_condition THEN statment_list
	[WHEN search_condition THEN statement_list]
	[ELSE statment_list]
END CASE;

-- 例子：给定一个月份，然后计算出所在的季度
delimiter $
create procedure pro_test8(in month int)
begin
	declare result varchar(10);
	case
		when month >= 1 and month <= 3 then
			set result = '第一季度';
		when month >= 4 and month <= 6 then
			set result = '第二季度';
		when month >= 7 and month <= 9 then
			set result = '第三季度';
		else
			set result = '第四季度';
	end case;
	select concat('传递的月份为：', month, '计算结果为：', result) as content;
end$
delimiter ;
```

##### while循环

有条件的循环语句，当满足条件时**进入循环**

```sql
while search_condition do
	statement_list
end while;

-- 例子：计算从 1加到 n的值
delimiter $
create procedure pro_test9(in n int)
begin
	declare total int default 0;
	declare num int default 1;
	while num <= n do
		set total = total + num;
		set num = num + 1;
	end while;
	select total;
end$
delimiter ;
```

##### repeat结构

有条件的循环控制语句，当满足条件时**退出循环**

```sql
REPEAT
	statement_list
	UNTIL search_condition
END REPEAT;

-- 例子：计算从 1加到 n的值
delimiter $
create procedure pro_test10(in n int)
begin
	declare total in default 0;
	repeat
		set total = total + n;
		set n = n - 1;
		until n = 0 -- 条件末尾没有分号
	end repeat;
	select total;
end$
delimiter ;
```

##### loop语句

loop实现简单的循环，退出循环的条件需要使用其他的语句定义，通常可以使用 leave语句实现

```sql
[begin_label:] LOOP
	statement_list
END LOOP [end_label]

-- begin_label：开始时设置别名
-- end_label: 结束别名
```

如果不在 statement_list中增加退出循环的语句，那么 LOOP语句可以用来实现简单的死循环

##### leave语句

用来从标注的流程构造中退出，通常和 BEGIN...END或者循环一起使用

```sql
-- 使用 LOOP和 leave的简单例子，退出循环
delimiter $
create procedure pro_test11(in n int)
begin
	declare total int default 0;
	c:loop
		set total = total + n;
		set n = n - 1;
		if n = 0 then
			leave c;
		end if;
	end loop c;
	select total;
end$
delimiter ;
```

##### 游标/光标

游标是用来存储查询结果集的数据类型，在存储过程和函数中可以使用光标对结果集进行循环的处理。光标的使用包括光标的声明、OPEN、FETCH和 CLOSE，其语法如下：

```sql
-- 声明光标
DECLARE cursor_name CURSOR FOR select_statement;

-- OPEN光标
OPEN cursor_name;

-- FETCH光标
-- 每 fetch一次，指针向下一行移动一次
-- fetch到没有数据的时候，报 no data
-- var_name(变量名称)需要和查询表的字段名，字段类型保持一致
FETCH cursor_name INTO var_name [, var_name] ...

-- CLOSE光标
CLOSE cursor_name;
```

示例如下：

```sql
delimiter $
create procedure pro_test12()
begin
	declare e_id int(11);
	declare e_name varchar(50);
	declare e_age int(11);
	declare e_salary int(11);
	
	-- 直接进行 fetch操作
	declare emp_result cursor for select * from emp;
	open emp_result;
	fetch emp_result into e_id, e_name, e_age, e_salary;
	select concat('id=', e_id, ',name=', e_name, ',age=', e_age, ',salary:', e_salary);
	
	fetch emp_result into e_id, e_name, e_age, e_salary;
	select concat('id=', e_id, ',name=', e_name, ',age=', e_age, ',salary:', e_salary);
	close emp_result;
	
	-- 使用循环进行 fetch操作，并且避免 no data异常
	declare has_data int default 1;
	declare emp_result cursor for select * from emp;
	declare exit HANDLER FOR NOT FOUND set has_data = 0;
	open emp_result;
	repeat
		fetch emp_result into e_id, e_name, e_age, e_salary;
		select concat('id=', e_id, ',name=', e_name, ',age=', e_age, ',salary:', e_salary);
		until has_data = 0
	end repeat;
	close emp_result;
end$
delimiter ;
```

### 存储函数

语法结构：

```sql
CREATE FUNCTION funtion_name([param type ...])
RETURNS type
BEGIN
	...
END;
```

例子：定义一个存储函数，请求满足条件的总记录数

```sql
delimiter $
create function fun1(in countryId int)
returns int
begin
	declare cnum int;
	select count(*) int cnum from city where country_id = countryId;
	return cnum;
end;
delimiter ;
```

调用存储函数

```sql
select fun1(10);
```

删除存储函数

```sql
DROP FUNCTION fun1;
```

## 触发器

### 简介

触发器是与表有关的数据库对象，指在 insert/update/delete之前或之后，触发并执行触发器中定义的 sql语句集合。触发器的这种特性可以协助应用在数据库端确保数据的完整性，日志记录，数据校验等操作。

使用别名 OLD和 NEW来引用触发器中发生变化的记录内容，这与其他数据库是相似的。

现在触发器只支持行级触发，不支持语句级触发。

| 触发器类型     | NEW和 OLD的使用                                      |
| -------------- | ---------------------------------------------------- |
| INSERT型触发器 | NEW表示将要或者已经新增的数据                        |
| UPDATE型触发器 | OLD表示修改之前的数据，NEW表示将要或已经修改后的数据 |
| DELETE型触发器 | OLD表示将要或者已经删除的数据                        |

### 创建触发器

语法结构

```sql
create trigger trigger_name
before/after insert/update/delete
on tbl_name
[for each row] -- 行级触发器
begin
	trigger_stmt;
end;
```

例子：通过触发器记录 emp表的数据变更日志，包含增加，修改，删除

```sql
delimiter $
-- 插入的触发器
create trigger emp_insert_trigger
after insert
on emp
for each row
begin
	insert into emp_logs(id, operation, operation_time, operation_id, operation_params)
	values (null, 'insert', now(), new.id,
          concat('插入后(id:', new.id, ', name:', new.name, ',age:', new.age, ',salary:', new.salary, ')'));
end$

-- 修改的触发器
create trigger emp_update_trigger
after update
on emp
for each row
begin
	insert into emp_logs(id, operation, operation_time, operation_id, operation_params)
	values (null, 'update', now(), new.id,
          concat('修改前(id:', old.id, ', name:', old.name, ',age:', old.age, ',salary:', old.salary,
                 '),修改后(', new.id, ', name:', new.name, ',age:', new.age, ',salary:', new.salary));
end$

-- 删除的触发器
create trigger emp_delete_trigger
after delete
on emp
for each row
begin
	insert into emp_logs(id, operation, operation_time, operation_id, operation_params)
	values (null, 'delete', now(), new.id,
          concat('删除前(id:', old.id, ', name:', old.name, ',age:', old.age, ',salary:', old.salary, ')'));
end$
delimiter ;
```

### 删除触发器

```sql
-- 语法结构
drop trigger [schema_name.]trigger_name;

-- 例子：
drop trigger emp_delete_trigger;
```

如果没有指定 schema_name，默认为当前数据库

### 查看触发器

可以通过执行 `SHOW TRIGGERS`命令查看触发器的状态、语法等信息

```sql
-- 语法结构
show triggers;
```

## Mysql的体系结构

![体系结构](https://zion-bucket1.obs.cn-north-4.myhuaweicloud.com/images/71e20eb10f6c31dbf57e0fdc55ac43eedad032d5d97b38c8e1b2770f051582cf-20220413101734347-2022-04-13-ef65c6106749be27012a93b0e8670d48-893970.png)

### 组成部分

整个 Mysql server由以下组成：

- Connection Pool： 连接池组件
- Management Services & Utilities： 管理服务和工具组件
- SQL Interface：SQL接口组件
- Parser：查询分析器组件
- Optimizer：优化器组件
- Caches & Buffers：缓冲池组件
- Pluggable Storage Engines：存储引擎
- File System：文件系统

### 四层结构

##### 连接层

最上层是一些客户端和链接服务，包含本地 sock通信和大多数基于客户端/服务端工具实现的类似于 TCP/IP的通信。主要完成一些类似于连接处理、授权认证及相关的安全方案。在该层引入了**线程池**的概念，为通过认证安全接入的客户端提供线程。同样在该层上可以实现基于SSL的安全链接。服务器也会为安全接入的每个客户端验证它所具有的操作权限。

##### 服务层

该层完成大多数的核心服务功能，如SQL接口，缓存的查询，SQL的分析与优化，部分内置函数的执行。所有跨存储引擎的功能也是在这一层实现的，如过程，函数等。在该层，服务器会解析查询并创建相应的内部解析树，并对其完成相应的优化如确定表的查询顺序，是否利用索引等，最后生成相应的执行操作。如果是 select语句，服务器还会查询内部的缓存，如果缓存空间足够大，这样在解决大量读操作的环境中能够很好的提升系统的性能。

##### 引擎层

存储引擎层，存储引擎真正的负责了Mysql中数据的存储和提取，服务器通过API和存储引擎进行通信。不同的存储引擎具有不同的功能，这样我们可以根据需要来选取合适的存储引擎。

##### 存储层

数据存储层，主要是将数据存储在文件系统之上，并完成与存储引擎的交互。



> 和其他数据库相比，Mysql的架构可以在多种不同场景中应用并发挥良好作用。主要体现在存储引擎上，插件式的存储引擎架构，将查询处理和其他的系统任务以及数据的存储提取分离。这种架构可以根据业务的需求和实际需要选择或者拓展存储引擎。

## 存储引擎

### 简介

- 存储引擎就是存储数据，建立索引，更新查询数据等技术的实现方式
- 存储引擎是基于表的，而不是基于库的，所以存储引擎也可被称为表类型
- Oracle, SqlServer等数据库只有一种存储引擎
- mysql5.0支持的存储引擎：InnoDB、 MyISAM、BDB、MEMORY、MERGE、EXAMPLE、NDB Cluster、ARCHIVE、CSV、BLACKHOLE、FEDERATED等
- InnoDB和 BDB提供事务安全表，其他是非事务安全表
- `show engines`命令查询当前数据库支持的存储引擎
- mysql5.5之前默认存储引擎是 MyISAM，之后改为 InnoDB
- `show variables like '%storage_engine%'`命令  查看默认存储引擎

### 各种存储引擎特性

#### 简介

| 特点         | InnoDB                 | MyISAM   | MEMORY | MERGE | NDB  |
| ------------ | ---------------------- | -------- | ------ | ----- | ---- |
| 存储限制     | 64TB                   | 有       | 有     | 没有  | 有   |
| 事务安全     | **支持**               |          |        |       |      |
| 锁机制       | **行锁（适合高并发）** | **表锁** | 表锁   | 表锁  | 表锁 |
| B树索引      | 支持                   | 支持     | 支持   | 支持  | 支持 |
| 哈希索引     |                        |          | 支持   |       |      |
| 全文索引     | 支持（5.6版本后）      | 支持     |        |       |      |
| 集群索引     | 支持                   |          |        |       |      |
| 数据索引     | 支持                   |          | 支持   |       | 支持 |
| 索引缓存     | 支持                   | 支持     | 支持   | 支持  | 支持 |
| 数据可压缩   |                        | 支持     |        |       |      |
| 空间使用     | 高                     | 低       | N/A    | 低    | 低   |
| 内存使用     | 高                     | 低       | 中等   | 低    | 高   |
| 批量插入速度 | 低                     | 高       | 高     | 高    | 高   |
| 支持外键     | **支持**               |          |        |       |      |

#### InnoDB

默认存储引擎，InnoDB提供了具有提交、回滚、崩溃恢复能力的事务安全。但是对比 MyISAM的存储引擎，InnoDB写的处理效率差一些，并且会占用更多的磁盘空间以保留数据和索引。

##### 事务控制

```sql
start transaction;
insert into goods_innodb(id, name) values (null, 'Meta20');
commit;
```

##### 外键约束

InnoDB是 mysql中唯一支持外键的存储引擎。在创建外键时，要求父表必须有对应的索引，子表在创建外键时，也会自动创建对应的索引。

```sql
create table country_innodb(
	country_id int not null auto_increment,
  name varchar(100) not null,
  primary key (country_id)
) engine=InnoDB default charset=utf8;

create table country_innodb(
	city_id int not null auto_increment,
  city_name varchar(100) not null,
  country_id int not null,
  primary key (city_id),
  key idx_fk_country_id(country_id),
  constraint `idx_fk_country_id` foreign key(country_id) references country_innodb(country_id) on delete restrict on update cascade
) engine=InnoDB default charset=utf8;

desc country_innodb;
desc country_innodb;
```

在创建外键时，可指定在删除，更新父表时，对子表进行相应操作，包括 RESTRICT、CASCADE、SET NULL和 NO ACTION

RESTRICT和 NO ACTION相同，是指限制在子表有关联记录的情况下，父表不能更新

CASCADE表示父表在更新或删除时，更新或者删除子表对应记录

SET NULL表示父表在更新或删除时，子表的对应数据被 set null

##### 存储方式

mysql的存储目录为 `/var/lib/mysql`

存储表和索引有两种方式

- 使用共享表空间存储，这种方式创建的表结构保存在 `.frm`文件中，数据和索引保存在` innodb_data_home_dir`和 `innodb_data_file_path`定义的表空间中，可以是多文件
- 使用多表空间存储，这种方式创建的表结构保存在` .frm`文件中，但每个表的数据和索引单独保存在 `.ibd`中

#### MyISAM

myisam不支持事务，不支持外键，其优势是访问速度快，对事务的完整性没有要求或者以 select, insert为主的应用基本都可以使用

##### 不支持事务

##### 文件存储方式

每个 MyISAM在磁盘上存储 3个文件，其文件名和表名相同，但拓展名分别是：

- `.frm`（存储表结构定义）
- `.MYD`（MYData，存储数据）
- `.MYI `（MYIndex，存储索引）

#### MEMORY

- 表的数据存在内存中
- 每个 MEMORY表实际对应一个磁盘文件，格式是 `.frm`，只存储表结构，数据在内存中
- 有利于数据的快速处理，提高查表的效率
- 默认使用 HASH索引
- 服务关闭，数据可能丢失

#### MERGE

特点：

- MERGE存储引擎是一组 MyISM表的组合，这些表的结构完全相同
- MERGE表本身不存储数据，对MERGE表进行查询、更新、删除实际上是对内部的 MyISAM表进行操作
- 对于MERGE表的插入操作，是通过 INSERT_METHOD子句定义插入的表，可以有3个不同的值，使用 FIRST或 LAST值似的插入操作作用于第一或最后一个表中，不定义或者定义为 no，表示不能对这个 MERGE表执行插入操作
- 可以对 MERGE表进行 DROP操作，实际只是删除 MERGE表的定义，对内部分表没有任何影响

例子：

```sql
create table order_all(
	order_id int,
  order_money double(10, 2),
  order_address varchar(50),
  primary key (order_id)
) engine = merge union = (order_1990, order_1991) INSERT_METHOD=LAST default charset=utf8;
```

### 存储引擎的选择

- InnoDB：需要支持事务，并发条件下的数据一致性，有效的降低由于删除和更新导致的锁定，还可以确保事务的完整提交和回滚，适合计费系统或者财务系统等对数据准确性较高的系统
- MyISAM：应用是以读操作和插入操作为主，只有很少的更新和删除操作，并且对事务的完整性、并发性要求不高
- MEMORY：需要快速定位记录和其他类似数据环境下，可以提供几块的访问。MEMORY的缺陷是对表的大小有限制，太大无法缓存在内存中，其次需要确保表的数据可以恢复。适合更新不太频繁的小表，用以快速得到访问结果
- MERGE：突破单个 MyISAM表的大小限制，并通过将不同表分布在多个磁盘上，有效改善 MERGE表的访问效率，对于存储诸如数据仓储等 VLDB环境十分合适

## 优化 SQL步骤

随着生产数据量的急剧增加，很多 SQL语句暴露出性能问题，成为整个系统性能的瓶颈

### 查看 SQL执行效率

Mysql客户端连接成功后，通过 `show [session|global] status`命令可以提供服务器状态信息。该命令可以根据需要加上参数 session或 global来显示 session级（当前连接）的计算结果和 global级（自数据库上次启动至今）的统计结果，默认为 session。

```sql
show [session|global] status

-- 查看当前 session中所有统计参数的值
show status like 'Com_______'; -- 七个_符号

-- 查看全局状态下的 innodb状态
show global status like 'Innodb_rows_%';
```

### 定位低效率执行 SQL

可通过以下两种方式进行定位：

- 慢查询日志：通过慢查询日志定位那么执行效率较低的 SQL语句，用 `--log-slow-queries[=file_name]`选项启动时，mysqld写一个包含所有执行时间超过 `long_query_time`秒的SQL语句的日志文件
- show processlist：慢查询日志在查询结束以后才记录，所以在应用反应执行效率出现问题的时候查询慢查询日志并不能定位问题，可以使用 show processlist命令查看当前 mysql在进行的线程，包括线程的状态，是否锁表等，可以实时查看 sql的执行情况，同时对一些锁表操作进行优化

### explain分析执行计划

通过以上步骤查询到效率低的 SQL语句后，可以通过 EXPLAIN或者 DESC命令获取 mysql如何执行 select语句的信息，包括在 select语句执行过程中表如何连接和连接的顺序

```sql
explain select * from tb_item where id = 1;
explain select * from tb_item where title = 'hahaha';
```

#### explain之 id

id字段是 select查询的序列号，是一组数字，表示查询中执行 select子句或者是操作表的顺序。

有三种情况：

- id相同表示加载表的顺序是从上到下
- id不同，id值越大，优先级越高，越先被执行
- id有相同，也有不同，同时存在。id相同的可以认为是一组，从上往下顺序执行。在所有的组中，id的值越大，优先级越高，越先执行

#### explain之 select_type

表示 select的类型，常见的取值，如下表

| select_type  | 含义                                                         |
| ------------ | ------------------------------------------------------------ |
| SIMPLE       | 简单的 select查询，查询中不包含子查询或者 UNION              |
| PRIMARY      | 查询中若包含任何复杂的子查询，最外层查询标记为该标识         |
| SUBQUERY     | 在 select和 where列表中包含了子查询                          |
| DERIVED      | 在 FROM列表中包含的子查询，被标记为 DERIVED(衍生)mysql会递归执行这些子查询，把结果放在临时表中 |
| UNION        | 若第二个 select出现在 union之后，则标记为 union。若union包含在 from子句的子查询中，外层 select将被标记为 derived |
| UNION RESULT | 从 union表获取结果的 select                                  |

#### explain之 table

表示这一行的数据是关于哪一张表的

#### explain之 type

表示访问类型，是比较重要的一个指标

| Type   | 含义                                                         |
| ------ | ------------------------------------------------------------ |
| NULL   | mysql不访问任何表，索引，直接返回结果                        |
| system | 表只有一行记录（等于系统表），这是 const类型的特例，一般不出现 |
| const  | 表示通过索引一次就找到了，const用于比较 primary key或者 unique索引。因为只匹配一行数据，所以很快。如将主键置于 where列表中，mysql就能将该查询转换为一个常量。const会将“主键”或“唯一”索引的所有部分与常量值进行比较 |
| eq_ref | 类似ref，区别在于使用的是唯一索引，使用主键的关联查询，查询出的记录只有一条。常见于主键和唯一索引扫描 |
| ref    | 非唯一性索引扫描，返回匹配某个单独值的所有行。本质上是一种索引访问，返回所有匹配某个单独值的所有行（多个） |
| range  | 只检索给定返回的行，使用一个索引来选择行。where之后出现 between, <, >, in等操作 |
| index  | index和 all的区别为 index类型只遍历了索引树，通常比 all快，all是遍历数据文件 |
| All    | 将遍历全表以找到匹配的行                                     |

结果值从最好到最坏的顺序：

```
null > system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > all
```

**一般来说，需要保证查询至少达到 range级别，最好达到 ref**

#### explain之 key

```sql
possible_keys: 显示可能应用在这张表的索引，一个或多个
key: 实际使用的索引，如果为 null，则没有使用索引
key_len: 表示索引中使用的字节数，该值为索引字段最大可能长度，并非实际使用长度，在不损失精确性的前提下，长度越短越好
```

#### explain之 rows

扫描行的数量

#### explain之 extra

其他的额外执行计划信息

| extra                    | 含义                                                         |
| ------------------------ | ------------------------------------------------------------ |
| using filesort           | 说明 mysql会对数据使用一个外部的索引排序，而不是按照表的索引顺序进行读取，称为“文件排序”，效率低 |
| using temporary          | 使用了临时表保存中间结果，mysql在对查询结果排序时使用临时表。常见于 order by和 group by，效率低 |
| using index              | 表示相应的 select操作使用了覆盖索引，避免访问表的数据行，效率不错 |
| using where              | 在查找使用索引的情况下，需要回表查询所需的数据               |
| using index condition    | 查找使用了索引，但需要回表查询数据                           |
| using index; using where | 查找使用了索引，但需要的数据都在索引列中能找到，故不需要回表查询数据 |

### show profile分析 SQL

```sql
-- 查看是否支持使用 profile
select @@have_profiling;

-- 查看是否开启 profile，0未开启
select @@profiling;

-- 开启 profile
set profiling =  1;

-- 执行 sql语句
select * from tb_item where id < 4;

-- 查看查询的记录
show profiles;

-- 查看该 sql执行过程中每个线程的状态和消耗时间
show profile for query query_id;

-- 进一步查看 all, cpu, block io, context switch, page faults等明细类型查看 mysql消耗
show profile cpu for query query_id;
```

> Sending data状态表示 mysql线程开始访问数据行并把结果返回给客户端，而不仅仅是返回给客户端。由于在 sending data状态下，mysql线程往往需要做大量的磁盘读取操作，所以经常是整个查询中最耗时的阶段。

### trace分析优化器执行计划

mysql提供了对 sql的跟踪 trace，通过 trace文件进一步了解优化器选择 a计划，不是选择 b计划

```sql
-- 打开 trace，设置格式为 json，并设置 trace最大能够使用的内存，避免解析过程中因为避免内存过小不能完整显示
SET optimizer_trace="enabled=on", end_markers_in_json=on;
SET optimizer_trace_max_mem_size=1000000;

-- 执行 sql语句
select * from tb_item where id < 4;

-- 最后检查 information_schema.optimizer_trace就可以知道 mysql如何执行 sql的
select * from information_schema.optimizer_trace\G;
```

## 索引的使用

索引是数据库优化最常用，也是最重要的手段之一。帮助用户解决大多数的mysql性能优化问题

### 验证索引提升查询效率

建立索引提高通过索引列查询的效率

### 索引的使用

#### 环境准备

```sql
create index idx_seller_name_sta_addr on tb_seller(name, status, address);
```

#### 避免索引失效

1. 全值匹配，对索引中所有列都指定具体值，查询表中的索引列。该情况下，索引生效，执行效率高。

	```sql
	explain select * from tb_seller where name='小米科技' and status='1' and 	address='北京市'\G;
	```

2. 复合索引需要符合**最左前缀法则**。如果索引了多列，要遵守最左前缀法则。指的是查询从索引的最左前列开始，并且不跳过索引的列

   ```sql
   -- 匹配最左前缀法则，走索引
   explain select * from tb_seller where name='小米科技';
   explain select * from tb_seller where name='小米科技' and status='1';
   explain select * from tb_seller where name='小米科技' and status='1' and address='北京市';
   explain select * from tb_seller where status='1' and address='北京市' and name='小米科技'; -- 优化器会优化顺序
   
   -- 违反最左前缀法则（跳过最左列或者中间列），索引失效
   explain select * from tb_seller where status='1';
   explain select * from tb_seller where address='北京市';
   explain select * from tb_seller where status='1' and address='北京市';
   explain select * from tb_seller where name='小米科技' and address='北京市'; -- 只走 name索引，没走 address索引
   ```
   
3. 范围查询右边的列，不能使用索引。

   ```sql
   -- 根据前面的两个字段 name, status查询是走索引的，但最后一个条件 address没有用到索引
   explain select * from tb_seller where name = '小米科技' and status > '1' and address = '北京市';
   ```

4. 不要在索引列上进行运算操作，索引将失效

   ```sql
   explain select * from tb_seller where substring(name, 3, 2) = '科技';
   ```

5. 字符串不加单引号，造成索引失效

   ```sql
   -- 字段 status是 varchar类型，不是 int类型，需要加单引号，避免隐式类型转换
   explain select * from tb_seller where name = '科技' and status = 0;
   ```

6. 尽量使用覆盖索引（只访问索引的查询列（索引列完全包含查询列）），减少 `select *`。如果查询列超过索引列，会降低性能

   ```sql
   explain select * from tb_seller where name='小米科技' and status='1' and address='北京市';
   explain select name, status, address from tb_seller where name='小米科技' and status='1' and address='北京市';
   -- 查询列超过索引列
   explain select name, status, address, password from tb_seller where name='小米科技' and status='1' and address='北京市';
   ```

7. 用 or分隔开的条件，如果 or前的条件中的列有索引，后面的列中没有索引，那么涉及的索引都不会被用到

   ```sql
   -- name字段是索引列，password不是索引列，中间 or进行连接是不走索引的
   explain select * from tb_seller where name='小米科技' or password='12345';
   
   -- and走索引
   explain select * from tb_seller where name='小米科技' and password='12345';
   ```

8. 以 %开头的 like模糊查询，索引失效。如果仅仅是尾部模糊匹配，索引不会失效。头部模糊匹配，索引失效

   ```sql
   -- 尾部模糊匹配，走索引
   explain select * from tb_seller where name like '科技%';
   
   -- 头部模糊匹配，不走索引
   explain select * from tb_seller where name like '%科技';
   
   -- 头尾部模糊匹配，不走索引
   explain select * from tb_seller where name like '%科技%';
   
   -- 通过覆盖索引的方式解决，name字段是索引列
   explain select name from tb_seller where name like '%科技%';
   explain select name, password from tb_seller where name like '%科技%'; -- password不是索引列，无法覆盖索引，失效
   ```

9. 如果 mysql评估使用索引比全表更慢，则不使用索引

10. is null，is not null **有时**索引失效

11. in走索引，not in索引失效

12. 单列索引和复合索引，尽量使用复合索引，而少使用单列索引

    ```sql
    -- 创建复合索引
    -- 相当于创建了三个索引：
    -- name
    -- name + status
    -- name + status + address
    create index idx_name_sta_address on tb_seller(name, status, address);
    
    -- 创建单列索引，数据库会选择一个最优的索引（辨识度最高）来使用，并不会使用全部索引
    create index idx_seller_name on tb_seller(name);
    create index idx_seller_status on tb_seller(status);
    create index idx_seller_address on tb_seller(address);
    ```

### 查看索引使用情况

```sql
-- 查看当前客户端会话中的索引使用情况
show status like 'Handler_read%';

-- 查看全局的索引使用情况
show global status like 'Handler_read%';
```

## SQL优化

### 大批量插入数据

对于 InnoDB类型的表，有以下几种方式提高导入效率

1. 主键顺序插入

   因为 InnoDB类型的表是按照主键的顺序保存的，所以将导入的数据按照主键顺序排列，可以有效提高导入数据效率。如果该表没有主键，那么系统会自动默认创建一个内部列作为主键。

   ```sql
   -- 脚本文件介绍：
   -- 	sql1.log -> 主键有序，插入效率高
   --  sql2.log -> 主键无序，插入效率低
   load data local infile '/root/sql1.log' into table `tb_user` fields terminated by ',' lines terminated by '\n';
   ```

2. 关闭唯一性校验

   ```sql
   -- 当前表存在唯一索引，可以在导入前关闭唯一性校验，提高导入效率
   SET UNIQUE_CHECKS = 0
   
   load data local infile '/root/sql1.log' into table `tb_user` fields terminated by ',' lines terminated by '\n';
   
   -- 导入完成恢复唯一性校验
   SET UNIQUE_CHECKS = 1
   ```

3. 手动提交事务

   ```sql
   -- 导入前关闭自动提交，提高导入效率
   SET AUTOCOMMIT = 0
   
   load data local infile '/root/sql1.log' into table `tb_user` fields terminated by ',' lines terminated by '\n';
   
   -- 导入完成恢复唯一性校验
   SET AUTOCOMMIT = 1
   ```

### 优化 insert语句

进行 insert操作，可以考虑以下几种优化方案

1. 尽量使用多个值表的 insert语句，缩短客户端与数据库的连接、关闭等消耗。比分开执行单个 insert语句效率高

   ```sql
   -- 原始方式
   insert into tb_test values(1, 'tom');
   insert into tb_test values(2, 'jerry');
   insert into tb_test values(3, 'cat');
   
   -- 优化后
   insert into tb_test values (1, 'tom'), (2, 'jerry'), (3, 'cat');
   ```

2. 在事务中进行数据插入

   ```sql
   start transaction;
   insert into tb_test values(1, 'tom');
   insert into tb_test values(2, 'jerry');
   insert into tb_test values(3, 'cat');
   commit;
   ```

3. 数据有序插入

   ```sql
   -- 原始方式
   insert into tb_test values(3, 'cat');
   insert into tb_test values(1, 'tom');
   insert into tb_test values(2, 'jerry');
   
   -- 优化后
   insert into tb_test values(1, 'tom');
   insert into tb_test values(2, 'jerry');
   insert into tb_test values(3, 'cat');
   ```

### 优化 order by语句

#### 环境准备

```sql
-- 建立复合索引
create index idx_emp_age_salary on emp(age, salary);
```

#### 两种排序方式

- 通过对返回数据进行排序，指的是 `filesort`排序（所有不通过索引直接返回排序结果的排序）

  ```sql
  explain select * from emp order by age desc;
  ```

- 通过有序索引顺序扫描直接返回有序数据，指的是 `using index`，不需要额外排序，操作效率高

  ```sql
  explain select `id` from emp order by age asc;
  
  -- 多字段排序，索引字段需要同时升序或者降序，还要注意位置顺序，不然会需要额外操作，出现 filesort
  explain select id, age, salary from emp order by age asc, salary asc; -- 效率高
  explain select id, age, salary from emp order by salary asc, age asc; -- 效率低
  explain select id, age, salary from emp order by age asc, salary desc; -- 效率低
  ```

#### Filesort优化

对于 filesort，mysql有两种排序算法：

- 两次扫描算法：mysql4.1之前只有该算法。首先根据条件取出排序字段和行指针信息，然后在排序区 sort buffer中排序，如果 sort buffer不够，则在临时表 temporay table中存储排序结果。完成排序后，再根据行指针回表读取记录，该操作会进行大量随机 I/O操作
- 一次扫描算法：一次性取出满足条件的所有字段，然后在排序区 sort buffer中排序后直接输出结果集。排序时内存开销较大，但效率比两次扫描算法高

mysql通过比较系统变量 max_length_for_sort_data的大小和 query语句取出的字段总大小，来决定使用何种算法，max_length_for_sort_data更大，使用一次扫描算法，否则使用两次扫描。

可以适当提高 sort_buffer_size和 max_length_for_sort_data系统变量，来增大排序区的大小，提高排序效率

```sql
show variables like 'max_length_for_sort_data';
show variables like 'sort_buffer_size';
```

### 优化 group by语句

Group by会先排序再分组，只比 order by操作多了分组操作，如果分组后使用了聚合函数，也会有一些聚合计算上的消耗

- 禁用排序

	```sql
	-- 执行 order by null，禁用排序
	explain select age, count(*) from emp group by age order by null;
	```

- 创建索引提高分组效率

  ```sql
  create index idx_emp_age_salary on emp(age, salary);
  explain select age, count(*) from emp group by age;
  ```


### 优化嵌套查询

使用子查询可以一次性完成很多逻辑上需要多个步骤才能完成的 sql操作，同时避免事务和表锁死，并且写起来也很容易。尽量使用更高效的 join来代替子查询

```sql
explain select * from t_user where id in (select user_id from user_role);

-- 优化后
explain select * from t_user u, user_role ur where u.id = ur.user_id;
```

### 优化 OR条件

没有索引的话，考虑增加索引，有索引的话，or之间的每个条件列都必须使用索引，而且不能使用复合索引

```sql
show index from emp;
explain select * from emp where id = 1 or age = 30\G;
```

建议使用 union替换 or

```sql
explain select * from emp where id = 1 union select * from emp where id = 10;
```

### 优化分页查询

一般分页查询时，通过创建覆盖索引能够比较好地提高性能。一个常见问题：`limit 2000000 10`，此时需要 mysql排序前 2000010条记录，仅仅返回 2000000 - 2000010的记录，其他记录丢弃，查询排序的代价非常大

```sql
explain select * from tb_item limit 2000000, 10;
```

#### 思路一

在索引上完成排序分页操作，最后根据主键关联回原表查询所需要的其他列内容

```sql
explain select * from tb_item t, (select id from tb_item order by id limit 2000000, 10) a where t.id = a.id;
```

#### 思路二

该方案适用于主键自增的表，可以把 limit查询转换成某个位置的查询

```sql
explain select * from tb_item where id > 2000000 limit 10;
```

### 使用 SQL提示

SQL提示，是优化数据库的一个重要手段，简单来说，就是在 sql语句中加入一些人为的提示来达到优化操作的目的

#### USE INDEX

在查询语句中表名的后面，添加 use index来提供希望 mysql去参考的索引列表，就可以让 mysql不再考虑其他可用的索引

```sql
create index idx_seller_name on tb_seller(name);
explain select * from tb_seller use index(idx_seller_name) where name = '小米科技';
```

#### IGNORE INDEX

如果用户只是单纯的想让 mysql忽略一个或多个索引，则可以使用 ignore index作为 hint

```sql
explain select * from tb_seller ignore index(idx_seller_name) where name '小米科技';
```

#### FORCE INDEX

为强制 mysql使用一个特定的索引，可在查询中时候用 force index作为 hint

```sql
explain select * from tb_seller force index(idx_seller_name) where name '小米科技';
```

## 应用优化

### 使用连接池

对于访问数据库，建立连接的代价比较昂贵，频繁创建关闭连接，比较耗费资源。有必要建立连接池以提高访问的性能

### 减少对 mysql的访问

#### 避免对数据进行重复检索

编写应用代码，需要能够清理对数据库的访问逻辑。能够一次连接获取到结果，就不用两次连接，可以大大减少对数据库无用的重复请求

#### 增加 cache层

在应用中增加缓存层来减轻数据库负担。

可以把部分数据从数据库抽取出来放到应用端以文本方式存储，或者使用 mybatis，Hibernate提供的一级缓存/二级缓存，或者使用 redis数据库来缓存数据

### 负载均衡

使用某种均衡算法，将固定的负载量分布到不同的服务器上，以此降低单台服务器的负载，达到优化效果

#### 利用 mysql复制分流查询

通过 mysql的主从复制，实现读写分离，使增删改操作走主节点，查询操作走从节点，从而降低单台服务器的读写压力

#### 采用分布式数据库架构

分布式数据库架构适合大数据量、负载高的情况，具有良好的拓展性和高可用性。通过在多台服务器之间分布数据，可实现在多台服务器之间的负载均衡，提高访问效率

## Mysql中查询缓存优化

### 简介

开启 mysql查询缓存，当执行完全相同的 sql语句时，服务器就会直接从缓存中读取结果，当数据被修改，之前的缓存失效，修改比较频繁的表不适合做查询缓存

### 操作流程

![操作流程](https://zion-bucket1.obs.cn-north-4.myhuaweicloud.com/images/34c7c9d462e94d31a8228b369c5e82ffef8525a44509fdd95a4fd3ab10e723a7-20220413101755448-2022-04-13-5752765809242a16859e1b649b49a34b-a801d8.png)

### 查询缓存配置

```sql
-- 查看当前 mysql数据库是否支持查询缓存
SHOW VARIABLES LIKE 'have_query_cache';

-- 查看当前 mysql是否开启了查询缓存
SHOW VARIABLES LIKE 'query_cache_type';

-- 查看查询缓存的占用大小
SHOW VARIABLES LIKE 'query_cache_size';

-- 查看查询缓存的状态变量
SHOW STATUS LIKE 'Qcache%';
-- Qcache_hits 查询缓存命中数
-- Qcache_inserts 添加到查询缓存的查询数
```

### 开启查询缓存

mysql的查询缓存默认是关闭的，需要手动修改配置文件中的配置项，并重启服务使其生效

```bash
# 在 /usr/my.cnf配置文件中，添加以下配置
# 开启 mysql的查询缓存
query_cache_type = 1
# off或 0 		-> 查询缓存功能关闭
# on或 1  		-> 查询缓存功能开启，复合缓存条件的 select的结果会被缓存，显式指定 SQL_NO_CACHE不予缓存
# demand或 2 -> 查询缓存功能按需进行，显式指定 SQL_CACHE语句才会缓存

# 重启服务
service mysql restart
```

### 查询缓存 select选项

可以在 SELECT语句中指定两个与查询缓存相关的选项：

SQL_CACHE：如果查询结果是可缓存的，并且 query_cache_type = on或 demand，则缓存查询结果

SQL_NO_CACHE：服务器不使用查询缓存。它既不检查查询缓存，也不检查结果是否已缓存，也不缓存查询结果

```sql
SELECT SQL_CACHE id, name FROM customer;
SELECT SQL_NO_CACHE id, name FROM customer;
```

### 查询缓存失效的情况

- SQL语句不一致的情况。SQL语句一致才可以命中缓存，区分大小写
- 查询语句中有一些不确定的时。如：`now(), current_date(), curdate(), curtime(), rand(), uuid(), user(), database()`
- 不使用任何表查询语句。如：`select 'A';`
- 查询 `mysql`，`information_schema`或 `performance_schema`数据库中的表时。如：`select * from information_schema.engines;`
- 在存储的函数，触发器或事件的主体内执行的查询
- 如果表更改，则使用该表的所有高速缓存查询都将变为无效并从缓存中删除。这包括使用 `MERGE`映射到已更改表的表的查询，`INSERT, UPDATE, DELETE, TRUNCATE TABLE, ALTER TABLE, DROP TABLE, DROP DATABASE`

## Mysql内存管理及优化

### 内存优化原则

1. 分配尽可能多的内存给 mysql做缓存
2. MyISAM存储引擎的数据文件读取依赖于操作系统自身的IO缓存，需要预留更多的内存给操作系统做 IO缓存
3. 排序区、连接区等缓存是分配给每个数据库会话（session）专用的，其默认值的设置要根据最大连接数合理分配，如果设置过大，不但浪费资源，而且会在并发连接较高时导致物理内存耗尽

### MyISAM内存优化

该存储引擎使用 key_buffer缓存索引块，加速 myIsam索引的读写速度。对于 myisam表的数据块，mysql没有单独的缓存机制，完全依赖于操作系统的 IO缓存

#### key_buffer_size

Key_buffer_size决定 myisam索引块缓存区的大小，直接影响到 myisam表的存取效率。可以手动设置该值，建议知道将 1/4可用内存分配给 key_buffer_size

```bash
# /usr/my.cnf文件
key_buffer_size=512M
```

#### read_buffer_size

如果需要经常顺序扫描 myisam表，可以增大该值来改善性能。但 read_buffer_size是每个 session独占的，默认值过大，会造成内存浪费

#### read_rnd_buffer_size

如果需要做排序的 myisam表，如带有 order by子句的 sql，适当增加该值来改善性能。同样是每个 session独占的，默认值过大，会造成内存浪费

### InnoDB内存优化

innodb用一块内存区做 IO缓存池，该缓存池不仅用来缓存 innodb的索引块，同时还缓存 innodb的数据块

#### innodb_buffer_pool_size

该变量决定了 innodb表数据和索引数据的最大缓存区大小。该值越大，缓存命中率越高，访问 innodb表需要的磁盘 IO越少，性能越高

```bash
innodb_buffer_pool_size=512M
```

#### innodb_log_buffer_size

该变量决定了 innodb重做日志缓存的大小，对于可能产生大量更新记录的大事务，增加该值的大小，可以避免 innodb在事务提交前就执行不必要的日志写入磁盘操作

```bash
innodb_log_buffer_size=10M
```

## Mysql并发参数调整

mysql server是多线程结构，包括后台线程和客户服务线程。多线程可以有效利用服务器资源，提高数据的并发性能

### max_connections

该变量控制允许连接到 mysql数据库的最大数量，默认值是 151。如果状态变量 `connection_errors_max_connections`不为零，并且一直增长，则说明不断有连接请求因数据库连接数已达上限而失败，这时可以考虑增大 `max_connections`的值。

mysql最大可支持连接数，取决于很多因素，包括操作系统的线程库质量、内存大小、每个连接的负荷、cpu的处理速度、期望的响应时间等。在 linux平台下，性能不错的机器可以支持 500 - 1000个连接

### back_log

该变量控制 mysql监听 tcp端口设置的积压请求栈大小。如果 mysql连接数达到 max_connections，新来的请求会被存在堆栈中，以等待某一连接释放资源，该堆栈的数量即为 back_log，如果连接数量超过 back_log，将不授予连接资源，直接报错。5.6.6版本之前默认值为 50，之后的版本默认为` 50 + (max_connections / 5)`，最大不超过 900。

增大 back_log的值可以提高数据库在短时间内处理大量连接请求的能力。

### table_open_cache

该变量控制所有 sql语句执行线程可打开表缓存的数量，而在执行 sql语句时，每个 sql执行线程至少要打开 1个表缓存。该参数的值根据 max_connections以及每个连接执行关联查询中涉及的表的最大数量来设定：`max_connections * N`

### thread_cache_size

为了加快连接数据库的速度，mysql会缓存一定数量的客户服务线程以备重用，通过该变量可控制 mysql缓存客户服务线程的数量。类似于线程池的概念

### innodb_lock_wait_timeout

该变量是用来设置 innodb事务等待行锁的时间，默认值是 50秒，可以根据需要进行动态设置。对于需要快速反馈的业务系统来说，可以将行锁的等待时间调小，以避免事务长时间挂起。对于后台运行的批量处理程序来说，可以将行锁的等待时间增大，以避免发生大的回滚操作

## Mysql锁问题

### 简介

锁是计算机协调多个进程或线程并发访问某一资源的机制（避免争抢）

保证数据并发访问的一致性、有效性是所有数据库必须解决的问题，锁冲突是影响数据库并发访问性能的一个重要因素

### 锁分类

从对数据操作的粒度分：

- 表锁：操作时，锁定整个表
- 行锁：操作时，锁定当前操作行

从对数据操作的类型分：

- 读锁（共享锁）：针对同一份数据，多个读操作可以同时进行而不会互相影响
- 写锁（拍它所）：当前操作没有完成之前，它会阻断其他写锁和读锁

### Mysql锁

引擎对锁的支持情况

| 存储引擎 | 表级锁 | 行级锁 | 页面锁 |
| -------- | ------ | ------ | ------ |
| MyISAM   | 支持   | 不支持 | 不支持 |
| InnoDB   | 支持   | 支持   | 不支持 |
| MEMORY   | 支持   | 不支持 | 不支持 |
| BDB      | 支持   | 不支持 | 支持   |

三种锁的特性

| 锁类型 | 特点                                                         |
| ------ | ------------------------------------------------------------ |
| 表级锁 | 偏向 MyISAM存储引擎，开销小，加锁快，不会出现死锁，锁定粒度大，锁冲突概率高，并发度最低 |
| 行级锁 | 偏向 InnoDB存储引擎，开销大，加锁慢，会出现死锁，锁定粒度最小，锁冲突概率低，并发度最高 |
| 页面锁 | 开销和加锁时间介于表锁和行锁之间，会出现死锁，锁定粒度介于表锁和行锁之间，并发一般 |

表级锁更适合以查询为主，只有少量按索引条件更新数据的应用，如 web应用

行级锁更适合大量按索引条件并发更新少量不同数据，同时又有并查询的应用，如一些在线事务处理（OLTP）系统

### MyISAM表锁

MyISAM只支持表锁

#### 加表锁

##### 自动加锁机制

无需手动加锁，执行查询语句前会自动给涉及的表加读锁，执行更新操作前会自动给涉及的表加写锁

##### 显式加锁

```sql
-- 显式加读锁
lock table tbl_name read;

-- 显式加写锁
lock table tbl_name write;

-- 解锁
unlock tables;
```

#### 读锁案例

![读锁](https://zion-bucket1.obs.cn-north-4.myhuaweicloud.com/images/c008a988378e6f6e6e10d42e5268351224c2875561ee52ef013a405d96002b2d-20220413101803789-2022-04-13-fc0f7ffdf3a3a76fb20856eb72014da5-92e03c.png)

#### 写锁案例

![写锁](https://obs-bucket-zino-1.obs.cn-north-4.myhuaweicloud.com/images/64d7a84fe67cb3c51d3c1b669bf17740bbad071c96f92d15ea0d88aa7d94b280.png)

#### 查看锁的竞争情况

```sql
show open tables;
-- In_user: 表当前被查询使用的次数，该值为零，则表是没锁的，而且当前没被使用
-- Name_locked: 表名称是否被锁定。名称锁定用于取消表或表进行重命名操作

show status like 'Table_locks%';
-- Table_locks_immediate: 能够立即获得表级锁的次数，每立即获取锁，该值加 1
-- Table_locks_waited: 不能立即获取表级锁而需要等到的次数，没等待一次，该值加 1。值越大，说明表级锁竞争越严重
```

#### 总结

MyISAM读锁会阻塞写，但不会阻塞读，写锁会阻塞读，又会阻塞写。

读写锁调度是写优先，造成了 myisam不适合用于写为主的表的存储引擎，写锁后，其他线程不能做任何操作，大量更细会使查询很难得到锁，从而造成永远阻塞

### InnoDB行锁

#### 简介

行锁特点：偏向 InnoDB存储引擎，开销大，加锁慢，会出现死锁，锁定粒度最小，锁冲突概率低，并发度最高

InnoDB和 MyISAM最大不同的两点：支持事务，支持行级锁

#### 背景知识

##### 事务及其 ACID属性

事务是由一组 SQL语句组成的逻辑处理单元

| ACID属性             | 含义                                                         |
| -------------------- | ------------------------------------------------------------ |
| 原子性（Atomicity）  | 事务是一个原子操作单元，对数据的修改要么全部成功，要么全部失败 |
| 一致性（Consistent） | 在事务开始和完成时，数据都必须保持一致状态                   |
| 隔离性（Isolation）  | 数据库系统提供一定的隔离机制，保证事务在不受外部并发操作影响的“独立”环境下运行 |
| 持久性（Durable）    | 事务完成之后，对数据的修改是永久的                           |

##### 并发事务处理带来的问题

| 问题                               | 含义                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| 丢失更新（Lost Update）            | 当两个或多个事务选择同一行，最初的事务修改的值，会被后面的事务修改的值覆盖 |
| 脏读（Dirty Reads）                | 当一个事务正在访问数据，并对数据进行了修改，而该修改还未提交到数据库中，这时另外一个事务也访问了这个数据，然后使用了该数据 |
| 不可重复读（Non-Repeatable Reads） | 一个事务在读取某些数据后的某个时间，再次读取以前读过的数据，却发现和以前读出的数据不一致 |
| 幻读（Phantom Reads）              | 一个事务按照相同的查询条件重新都以前查询过的数据，却发现其他事务插入了满足其查询条件的新数据 |

##### 事务隔离级别

数据库提供了事务隔离机制来解决事务并发问题。

事务隔离越严格，并发副作用越小，但代价也越大，因为事务隔离实质上是使用事务在一定程度上“串行化”进行，与“并发”相矛盾。

| 隔离级别         | 丢失更新 | 脏读 | 不可重复读 | 幻读 |
| ---------------- | -------- | ---- | ---------- | ---- |
| Read uncommitted | 否       | 是   | 是         | 是   |
| Read committed   | 否       | 否   | 是         | 是   |
| Repeatable read  | 否       | 否   | 否         | 是   |
| Serializable     | 否       | 否   | 否         | 否   |

mysql的默认隔离级别是 Repeatable read

```sql
show variables like 'tx_isolation';
```

#### InnoDB的行锁模式

##### 两种类型行锁

- 共享锁（S）：读锁，简称S锁，共享锁就是多个事务对同一数据可以共享一把锁，都能访问导数据，但只读不可改
- 排它锁（X）：写锁，简称X锁，排它锁不能与其它锁共存，如果一个事务获取了一行数据的写锁，其他事务不能再获取该行的共享锁和排它锁，获取排它锁的事务可以读取和修改数据行

##### 自动加锁机制

对于更新语句，InnoDB会自动给涉及数据集加排它锁（X）

对于查询语句，InnoDB不会加锁

##### 显式加锁

```sql
-- 显式加共享锁（S）
SELECT * FROM tbl_name WHERE ... LOCK IN SHARE MODE;

-- 显式加排它锁（X）
SELECT * FROM tbl_name WHERE ... FOR UPDATE;
```

#### 案例准备工作

#### 行锁基本演示

#### 无索引行锁升级为表锁

如果不通过索引条件检索数据，那么 InnoDB将对表中的所有记录加锁，实际效果跟表锁一样

```sql
-- 查看当前表的索引
show index from test_innodb_lock;
```

#### 间隙锁危害

当我们用范围条件，而不是使用相等条件检索数据，并请求共享或排它锁时，InnoDB会给符合条件的已有数据加锁。对于键值在条件范围内但并不存在的记录，叫做**间隙（GAP）**。InnoDB会对这个**间隙**加锁，这种锁机制就是所谓的**间隙锁（Next-Key锁）**

示例

![间隙锁](https://zion-bucket1.obs.cn-north-4.myhuaweicloud.com/images/1184f987a972e8656de5c30a7d6585f636d2dcf8ac40ff45a8ffd305d6cdfb6c-20220413101811736-2022-04-13-a6975c70781e755d0d4d82c6bfc7194d-90e6ef.png)

#### InnoDB行锁竞争情况

```sql
show status like 'innodb_row_lock%';

-- Innodb_row_lock_current_waits: 当前正在等待锁定的数量
-- Innodb_row_lock_time: 从系统启动到现在锁定总时间长度
-- Innodb_row_lock_time_avg: 每次等待所花平均时长
-- Innodb_row_lock_time_max: 从系统启动到现在等待最长的一次所花的时间
-- Innodb_row_lock_waits: 系统启动后到现在总共等待的次数
-- 当等待次数很高，而且每次等待的时长也不小时，就需要分析出现过多等待的原因，并制定优化方案
```

#### 总结

InnoDB由于实现了行锁，性能损耗会比表锁高，但在整体并发处理能力还是高于 MyISAM的表锁，并发量越高，性能表现越有优势。行锁使用不当，会使 InnoDB整体性能下降。

##### 优化建议

- 尽可能通过索引来检索数据，避免无索引情况下，行锁升级为表锁
- 合理设计索引，尽量缩小锁的范围
- 尽可能减少索引条件，索引范围，避免间隙锁
- 尽量控制事务大小，减少锁定资源量和时间
- 尽可能使用低级别事务隔离（优先考虑满足业务需求）

## 常用 SQL技巧

### SQL执行顺序

#### 编写顺序

```sql
SELECT DISTINCT <select_list>
FROM <left_table> <join_type>
JOIN <right_table> ON <join_condition>
WHERE <where_condition>
GROUP BY <group_by_list>
HAVING <having_condition>
ORDER BY <order_by_condition>
LIMIT <limit_params>
```

#### 执行顺序

```sql
FROM <left_table>
ON <join_condition>
<join_type> JOIN <right_table>
WHERE <where_condition>
GROUP BY <group_by_list>
HAVING <having_condition>
SELECT DISTINCT <select_list>
ORDER BY <order_by_condition>
LIMIT <limit_params>
```

### 正则表达式使用

```sql
select * from tb_book where regexp '^j';
select * from tb_book where regexp 'j$';
select * from tb_book where regexp '[uvw]';
```

### Mysql常用函数

```sql
-- 数值型
abs -- 绝对值
sqrt -- 二次方根
mod -- 余数
ceil|ceiling -- 向上取整
floor -- 向下取整
rand -- 生成 0 - 1之间的随机数，传入整数参数是用来产生
round -- 对所传参数进行四舍五入
sign -- 返回参数的符号
pow|power -- 求所传参数的次方
sin -- 正弦值
asin -- 反正弦值
cos -- 余弦值
acos -- 反余弦值
tan -- 正切值
atan -- 反正切值
cot -- 余切值

select abs(-10); -- 10

-- 字符串
length -- 字符串长度
concat -- 合并字符串，支持多参
insert -- 替换字符串
lower  -- 转小写字母
upper  -- 转大写
left	 -- 从左侧截取字符串
right  -- 从右侧截取字符串
trim	 -- 两侧清空格
replace -- 替换
substring -- 截取字符串
reverse -- 反转顺序

-- 日期
curdate|current_date -- 当前系统的日期值
curtime|current_time -- 当前系统的时间值
now|sysdate -- 当前系统的日期和时间值
month -- 指定日期的月份
monthname -- 指定日期的英文名称
dayname -- 指定日期对应的星期几英文名称
dayofweek -- 指定日期对应的一周的索引位置值
week -- 指定日期是一年中的第几周，0 - 52或 1 - 53
dayofyear -- 指定日期是一年中的第几天，1 - 366
dayofmonth -- 指定日期是一个月中的第几天，1 - 31
year -- 获取年份，1970 - 2069
time_to_sec -- 时间参数换算秒
sec_to_time -- 秒参数换算时间
date_add|adddate -- 向日期添加指定时间间隔
date_sub|subdate -- 向日期减去指定时间间隔
addtime -- 时间加法运算，在原始时间添加指定时间
subtime -- 时间减法运算
datediff -- 获取两个时间的间隔，返回值为参数 1减去参数 2
date_format -- 根据参数返回指定格式的日期
weekday -- 获取日期在一周内对应的工作日索引

-- 聚合函数
max
min
count
sum
avg  	-- 平均值
```

## Mysql常用工具

### mysql

mysql服务，指的是mysql的客户端工具

```shell
mysql [options] [database]
```

#### 连接选项

```shell
# 参数
-u, --user=name  # 指定用户名
-p, --password[=name] # 指定密码
-h, --host=name  # 指定服务器Ip或域名
-P, --port=#		 # 指定连接端口号，大写字母 P

# 示例
mysql -h127.0.0.1 -P3306 -uroot -p12345
```

#### 执行选项

```shell
-e, --execute=name # 执行 sql语句并退出
# 此选项可以在 mysql客户端执行 sql语句，而不用连接到 mysql数据库在执行，对于一些批处理脚本，这种方式十分方便
mysql -uroot -p12345 db01 -e "select * from tb_book";
```

### mysqladmin

mysqladmin是一个执行管理操作的客户端程序。可以用来检查服务器的配置和当前状态、创建并删除数据库等。

```shell
# 查看该指令的帮助文档
mysqladmin --help

# 创建数据库
mysqladmin -uroot -p12345 create 'db01';
# 删除数据库
mysqladmin -uroot -p12345 drop 'db01';
# 查看版本号
mysqladmin -uroot -p12345 version;
```

### mysqlbinlog

由于服务器生成的二进制日志文件以二进制格式保存，所以如果想要检查这些文本的文本格式，就会使用到 mysqlbinlog日志管理工具

```shell
mysqlbinlog [options] log-files1 log-files2 ...
# 选项
-d, --database=name  # 指定数据库名称，只列出指定的数据库相关操作
-o, --offset=#			 # 忽略掉日志中的前 n行命令
-r, --result-file-name # 将输出的文本格式日志输出到指定文件
-s, --short-form		 # 显示简单格式，省略掉一些信息
--start-datetime=date1 --stop-datetime=date2 # 指定日期间隔内的所有日志
--start-position=pos1  --stop-position=pos2  # 指定位置间隔内的所有日志
```

### mysqldump

mysqldump客户端工具用来备份数据库或在不同数据库之间进行数据迁移。备份内容包含创建表，及插入表的sql语句

#### 语法

```shell
mysqldump [options] db_name [tables]
mysqldump [options] --database/-B db1 [db2 db3...]
mysqldump [options] --all-databases/-A
```

#### 连接选项

```shell
# 参数
-u, --user=name # 指定用户名
-p, --password[=name] # 指定密码
-h, --host=name # 指定服务器IP或域名
-P, --port=#		# 指定连接端口
```

#### 输出内容选项

```shell
# 参数
--add-drop-database  # 在每个数据库创建语句前加上 Drop database语句
--add-drop-table		 # 在每个表创建语句前加上 Drop table语句，默认开启。不开启(--skip-add-drop-table)

-n, --no-create-db   # 不包含数据库的创建语句
-t, --no-create-info # 不包含数据表的创建语句
-d, --no-data				 # 不包含数据

-T, --tab=name			 # 自动生成两个文件：一个 .sql文件，创建表结构。一个 .txt文件，数据文件，相当于 select info outfile
```

#### 例子

```shell
# 备份 db01数据库
mysqldump -uroot -p12345 db01 --add-drop-database --add-drop-table > a.sql

# 备份 db01数据库中 tb_book这个表
mysqldump -uroot -p12345 db01 tb_book --add-drop-database --add-drop-table > b.sql

# 备份 db01数据库中 tb_book这个表到 /tmp目录下
mysqldunp -uroot -p12345 -T /tmp db01 tb_book
```

### mysqlimport

mysqlimport是客户端数据导入工具，用来导入 mysqldump加 `-T`参数后导出的文本文件

```shell
# 语法，运行在 shell中
mysqlimport [options] db_name textfile1 [textfile2...]

# 示例
mysqlimport -uroot -p12345 db01 /tmp/tb_book.txt
```

### source

如果需要导入 sql文件，可以使用 mysql中的 source指令，运行在 sql命令行

```sql
source /root/tb_book.sql
```

### mysqlshow

mysqlshow是客户端对象查找工具，用来快速的查找存在哪些数据库、数据库中的表、表中的列或索引

```shell
# 语法
mysqlshow [options] [db_name [table_name [col_name]]]

# 参数
--count # 显示数据库及表的统计信息（数据库，表均可以不指定）
-i			# 显示指定数据库或者指定表的状态信息

# 示例
# 查询每个数据库的表的数量及表中记录的数量
mysqlshow -uroot -p12345 --count
# 查询 db01库中每个表中的字段数及行数
mysqlshow -uroot -p12345 db01 --count
# 查询 db01库中 tb_book表的详细情况
mysqlshow -uroot -p12345 db01 tb_book --count
```

## Mysql日志

msql有 4中不同类型的日志，分别是错误日志、二进制日志、查询日志和慢查询日志，这些日志记录着数据在不同方面的踪迹。

### 错误日志

错误日志是mysql中最重要的日志之一，它记录了 mysqld启动和停止时，以及服务器在运行过程中发生任何严重错误时的相关信息。当数据库出现任何故障导致无法正常使用时，可以首先查询此日志。

该日志默认开启，默认存放目录为 mysql的数据目录（var/lib/mysql），默认日志文件名为 hostname.err（hostname是主机名）

查看日志位置

```sql
show variables like 'log_error%';
```

查看日志内容

```shell
tail -f /var/lib/mysql/xaxh-server.err
```

### 二进制日志

#### 简介

二进制日志（BINLOG）记录了所有的 DDL（数据定义语言）语句和DML（数据操纵语言）语句，但不包括数据查询语句。此日志对灾难时的数据恢复起着极其重要的作用，mysql的主从复制，就是通过该 binlog实现的。

二进制日志，默认情况下是没有开启的，需要到 mysql的配置文件中开启，并配置 myql日志的格式

配置文件位置：/usr/my.cnf

日志存放位置：配置时，给定了文件名但是没有指定路径，日志默认写入 mysql的数据目录

```shell
# 配置开启 binlog日志，日志的前缀为 mysqlbin ---> 生成的文件名如：mysqlbin.000001, mysqlbin.000002
log_bin=mysqlbin

# 配置二进制日志的格式
binlog_format=STATEMENT
```

#### 日志格式

##### STATEMENT

该日志格式记录的都是 sql语句（statement），每一条对数据进行修改的 sql都会记录在日志文件中，通过 mysql提供的 mysqlbinlog工具，可以清晰的查看每条语句的文本。主从复制的时候，从库（slave）会将日志解析为原文本，并在从库重新执行一次

##### ROW

该日志格式记录的是每一行的数据变更，而不是记录 sql语句。比如：update tb_book set status='1'; 如果是 statement格式，在日志中会记录一行 sql文件。如果是 row，由于是对全表进行更新，也就是每一行记录都会发生变更，row格式的日志中会记录每一行的数据变更

##### MIXED

mysql默认的日志格式，混合了 statement和 row两种格式。默认情况下采用 statement，但在一些特殊情况下采用 row进行记录。mixed格式能尽量利用两种模式的优点，避开缺点

#### 日志读取

由于日志以二进制方式存储，不能直接读取，需要用 `mysqlbinlog`工具查看

```shell
mysqlbinlog log-file;
```

##### 查看 statement日志

查看日志文件

- myqslbin.index 该文件是日志索引文件，记录日志的文件名
- mysqlbin.000001 日志文件

查看日志内容

```shell
mysqlbinlog mysqlbing.000001
```

##### 查看 row日志

配置

```shell
log_bin=mysqlbin
binlog_format=row
```

如果日志格式是 row，可以在 mysqlbinlog后面加上参数 `-vv`查看格式化之后的内容

```shell
mysqlbinlog -vv mysqlbin.000002
```

#### 日志删除

对于比较繁忙的系统，由于每天生成的日志量大，这些日志如果长时间不清除，将会占用大量的磁盘空间

##### 方式一

通过 Reset Master指令删除全部 binlog日志，删除之后，日志编号，将从 xxx.000001重新开始

```sql
Reset Master
```

##### 方式二

该命令将删除 \*\*\*\*\*编号之前的所有日志

```sql
purge master logs to 'mysqlbin.*****';
```

##### 方式三

该命令将删除日志为 “yyyy-mm-dd hh24:mi:ss”之前产生的所有日志

```sql
purge master logs before 'yyyy-mm-dd hh24:mi:ss';
```

##### 方式四

设置参数 `--expire_logs_days=#`，此参数的含义是设置日志的过期天数，过了指定的天数后日志将会被自动删除，这样有利于减少 DBA管理日志的工作量

```shell
# 配置如下
log_bin=mysqlbin
binlog_format=ROW
--expire_logs_days=3
```

### 查询日志

查询日志记录了客户端所有操作语句，而二进制日志不包含查询数据的 sql语句，默认不开启查询日志

```shell
# 该选项来开启查询日志，可选值：0（关闭）或 1（开启）
general_log=1
# 设置日志的文件名，如果没有指定，默认的文件名为 host_name.log
general_log_file=file_name
```

在 mysql的配置文件 /usr/my.cnf中配置如下内容

```shell
log_bin=mysqlbin
binlog_format=ROW

expire_logs_days=3

# 开启查询日志
general_log=1
# 配置查询日志的文件名
general_log_file=mysql_query.log
```

### 慢查询日志

慢查询日志记录了所有执行时间超过参数 long_query_time设置值并且扫描记录数不小于 min_examined_row_limit的所有的 sql语句的日志。long_query_time默认为 10秒，最小为 0，精度可以到微秒

##### 文件位置和格式

慢查询日志默认是关闭的。可以通过两个参数来控制慢查询日志

```shell
# 该参数用来控制慢查询日志是否开启，0（关闭）或 1（开启）
slow_query_log=1

# 该参数用来指定慢查询日志的文件名
slow_query_log_file=slow_query.log

# 该选项用来配置查询的时间限制，超过这个时间将认为值慢查询，将需要进行日志记录，默认为 10s
long_qeury_time=10
```

##### 日志的读取

和错误日志、查询日志一样，慢查询日志记录的格式也是纯文本，可以被直接读取

###### 查询 long_query_time

```sql
show variables like 'long_query_time';
```

###### 查看慢查询日志

- 直接查看日志内容：`cat slow_query.log`
- 慢查询日志内容很多，使用自带工具对慢查询日志进行分类汇总： `mysqldumpslow slow_query.log`
- 持续查看追加内容：`tail -f slow_query.log`

## Mysql复制

### 简介

复制是指将主数据库的 DDL和DML操作通过二进制日志传到从库服务器中，然后再从库上对这些日志重新执行（也叫重做），从而使得从库和主库的数据保持同步

mysql支持一台主库同时向多台从库进行复制，从库同时也可以作为其他从服务器的主库，实现链式复制

### 复制原理

![主从复制](https://zion-bucket1.obs.cn-north-4.myhuaweicloud.com/images/0a2737ec7aada93440c737f49de40f107991c1dd0b964d11c2b3606dfcd10078-20220413101826005-2022-04-13-8fd38cd7f0d170705dc228ab2580b79c-d824f1.png)

从上图来看，复制分为三步：

- master主库在事务提交时，会把数据变更作为时间 events记录在二进制日志文件 binlog中
- 主库推送二进制日志文件 binlog中的日志时间到从库的中继日志 relay log
- slave重做中继日志中的事件 ，将改变反应它自己的数据

### 复制优势

- 主库出现问题，可以快速切换到从库提供服务
- 可以在从库中执行查询操作，用主库更新数据，实现读写分离，降低主库访问压力
- 可以在从库中执行备份，以避免备份期间影响主库的服务

### 搭建步骤

#### master

- 在 master的配置文件（/usr/my.cnf）中配置如下内容:

  ```shell
  # mysql服务 ID，保证整个集群环境中唯一
  server-id=1
  # mysql binlog日志的存储文件和文件名
  log-bin=/var/lib/mysql/mysqlbin
  # 错误日志，默认已经开启
  # log-err
  # mysql的临时目录
  # basedir
  # mysql的数据存放目录
  # datadir
  # 是否只读，1（只读），0（读写）
  read-only=0
  # 忽略的数据，指不需要同步的数据库
  binlog-ignore-db=mysql
  ```

- 执行完毕，重启 mysql `service mysql restart`

- 创建同步数据的账户，并且进行授权操作

  ```sql
  grant replication slave on *.* to 'slave_account'@'192.168.1.131' identified by '12345';
  flush privileges;
  ```

- 查看 master状态

  ```sql
  show master status
  
  -- file: 从哪个日志文件开始推送日志文件
  -- position: 从哪个文件开始推送日志
  -- Binlog_Ignore_DB: 指定不需要同步的数据库
  ```

#### slave

- 在 slave端配置文件

  ```shell
  # mysql服务端 id，唯一
  server-id=2
  
  # 指定 binlog日志
  log-bin=/var/lib/mysql/mysqlbin
  ```

- 执行完毕，重启 mysql `service mysql restart`

- 指定如下指令，指定从库对应的主库的 ip地址，用户名，密码，从哪个日志文件开始的哪个位置开始同步推送日志

  ```sql
  change master to master_host='192.168.1.130', master_user='slave_account', master_password='12345', master_log_file='mysqlabin.000001', master_log_pos=413;
  ```

- 开启同步操作

  ```sql
  start slave;
  show slave status;
  ```

- 停止同步操作

  ```sql
  stop slave;
  ```
