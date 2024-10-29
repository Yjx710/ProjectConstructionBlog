# 数据库操作

## 增

> * 创建库
>
>   ```mysql
>   # 基本语法
>   create database 数据库名;
>   # 使用
>   create database test1;
>   ```
>
>   <img src="/images/mysql/image-20241025153411279.png" alt="image-20241025153411279" style="zoom:80%;" />
>
>   ```mysql
>   # 全面语法
>   create database [if not exists] 数据库名 数据库选项;
>   # 解释
>   - [if not exists]:
>     中括号代表可选;
>     if not exists是一个可选项,只有在数据库不存在的情况下才会创建数据库,可以避免因重复创建数据库而导致的错误
>   - 数据库选项：
>     character set charset_name  # 指定字符集
>     collate collation_name # 指定排序规则
>   
>   # 使用
>   create database if not exists test1 character set utf8mb4 collate utf8mb4_general_ci; 
>   ```
>   
>   * 可以看到 用了`if not exists`,之前存在`test1`也不会报错
>   
>     <img src="/images/mysql/image-20241025160055596.png" alt="image-20241025160055596" style="zoom:80%;" />
>

## 删

> * 删除库
>
>   ```mysql
>   # 语法
>   drop database [if exists] 库名
>     if exists是一个可选项,只有在数据库存在的情况下才会删除数据库,可以避免因删除不存在的数据库而导致的错误
>   # 使用
>   drop database if exists test;
>   ```
>
>   <img src="/images/mysql/image-20241025162656702.png" alt="image-20241025162656702" style="zoom:80%;" />
>
>   * 刷新一下 你会发现数据库已经不存在了! **注意**: 数据库其数据库下的表都会一并被删除掉!!! 

## 改

> * 修改库的选项信息
>
>   ```mysql
>   # 语法
>   alter database 库名 选项信息
>   # 使用
>   alter database test character set utf8 collate utf8_general_ci;
>   ```
>
>   <img src="/images/mysql/image-20241025162131992.png" alt="image-20241025162131992" style="zoom:80%;" />
>
>   <img src="/images/mysql/image-20241025162337874.png" alt="image-20241025162337874" style="zoom:80%;" />

## 查

> * 查看当前数据库
>
>   ```mysql
>   select database(); 
>   ```
>
>   <img src="/images/mysql/image-20241025152630676.png" alt="image-20241025152630676" style="zoom:80%;" />

> * 显示当前时间、用户名、数据库版本
>
>   ```mysql
>   select now(), user(), version();
>   ```
>
>   <img src="/images/mysql/image-20241025152757521.png" alt="image-20241025152757521" style="zoom:80%;" />

> * 查看已有库
>
>   ```mysql
>   # 基本使用
>   show databases;
>   ```
>
>   <img src="/images/mysql/image-20241025161043396.png" alt="image-20241025161043396" style="zoom:80%;" />
>
>   ```mysql
>   # 加上过滤条件
>   # 语法: 使用like对结果进行过滤
>   show databases like pattern;
>   # 使用: 返回t开头的数据库
>   show databases like 't%';
>   ```
>
>   <img src="/images/mysql/image-20241025161356743.png" alt="image-20241025161356743" style="zoom:80%;" />

> * 查看当前库信息
>
>   ```mysql
>   # 语法
>   show create database 数据库名;
>   # 使用
>   show create database test;
>   ```
>
>   <img src="/images/mysql/image-20241025161544180.png" alt="image-20241025161544180" style="zoom:80%;" />



# 表操作

## 增

> * 创建表
>
>   ```mysql
>   # 语法
>   create [temporary] table [if not exists] [库名.]表名 ( 表的结构定义 )[ 表选项]
>   # 解释
>     [temporary]: 可选项,表示创建的表是一个临时表,仅在当前会话中可见.一旦会话结束,临时表将被自动删除
>     [if not exists]: 可选项,只有在表不存在的情况下才会创建表。如果表已经存在,也不会报错
>     表的结构定义: 定义表的结构,包括列名、数据类型、约束等
>     	-- 字段定义: 字段名 数据类型 [not null | null] [default default_value] [auto_increment] [unique [key] | [primary] key] [comment 'string']
>     	  -- 各选项的含义
>     	    字段名: 指定字段的名称
>     	    数据类型: 指定字段的数据类型，例如 int, varchar(255), date, float, boolean 等
>     	    [not null | null]: 指定该字段不能为 null 值，必须有具体的值,如果未指定 not null 或 null，默认情况下字段可以接受 null 值
>     	    [default default_value]: 
>     	    	指定该字段的默认值。如果在插入数据时未显式指定该字段的值，则使用默认值;
>     	    	例如: default 'unknown' 表示如果插入时未指定值，则使用 'unknown' 作为默认值
>     	    [auto_increment]: 
>     	    	通常用于主键字段，表示该字段的值会自动递增。适用于唯一标识符，如 ID;
>     	    	通常与 primary key 一起使用
>     	    [unique [key] | [primary] key]:
>     	    	| 符号表示“或”（OR）的关系,表示可以选择其中之一来应用到字段上
>     	    	[unique [key]:
>            		指定该字段的值必须是唯一的，不能有重复值;
>            		unique 关键字可以单独使用，也可以加上 [KEY]（KEY 是可选的，通常可以省略）
>               [primary] key]:
>               	指定该字段为主键，用于唯一标识表中的每一行记录;
>               	主键字段不能有重复值，并且通常也不能为 null 值;
>               	一个表只能有一个主键，但可以由多个字段组成复合主键;
>            [comment 'string']:
>            	为字段添加注释，用于说明字段的目的或用途;
>            	例如: comment '用户姓名' 可以为 name 字段添加注释
>     表选项: 可选项,用于指定表的一些额外选项,如存储引擎、字符集等
>       -- 字符集
>       	charset = charset_name
>       	如果表没有设定，则使用数据库字符集
>       -- 存储引擎
>       	engine = engine_name
>           表在管理数据时采用的不同的数据结构，结构不同会导致处理方式、提供的特性操作等不同
>           常见的引擎：InnoDB MyISAM Memory/Heap BDB Merge Example CSV MaxDB Archive
>           不同的引擎在保存表的结构和数据时采用不同的方式
>           MyISAM表文件含义：.frm表定义，.MYD表数据，.MYI表索引
>           InnoDB表文件含义：.frm表定义，表空间数据和日志文件
>           SHOW ENGINES -- 显示存储引擎的状态信息
>           SHOW ENGINE 引擎名 {LOGS|STATUS} -- 显示存储引擎的日志或状态信息
>       -- 自增起始数
>       	auto_increment = 行数
>      	-- 数据文件目录
>      		data directory = '目录'
>      	-- 索引文件目录
>      		index directory = '目录'
>      	-- 表注释
>      		comment = 'string'
>      	-- 分区选项
>      		partition by ... (详细见手册)
>   # 注意
>     1. 每个字段必须有数据类型
>     2. 最后一个字段后不能有逗号
>   # 使用
>   create table if not exists test.student (
>   	id bigint not null auto_increment primary key comment '主键',
>   	name varchar(255) not null comment '姓名',
>   	age int comment '年龄'
>   ) charset = utf8mb4 collate = utf8mb4_general_ci comment = '学生表';
>   ```
>
>   <img src="/images/mysql/image-20241025180343916.png" alt="image-20241025180343916" style="zoom:80%;" />
>
>   <img src="/images/mysql/image-20241025180442539.png" alt="image-20241025180442539" style="zoom:80%;" />

## 删

## 改

> * 修改表本身的选项（指更改表的整体属性或特性，而不是修改表中的列或索引，例如：表的存储引擎，字符集，排序规则，注释等等都是可以被修改的）
>
>   ```mysql
>   # 语法
>   alter table 表名 表的选项;
>   # 解释
>   表的选项这里就不一一列举了，遇到了百度下即可
>   # 使用：修改表的备注
>   alter table student comment = '公司表';
>   ```
>
>   <img src="/images/mysql/image-20241028174940381.png" alt="image-20241028174940381" style="zoom:80%;" />

> * 重命名
>
>   ```mysql
>   # 语法
>   rename table 原表名 to 新表名;
>   # 使用：把student表名字改成company
>   rename table student to company;
>   ```
>
>   <img src="/images/mysql/image-20241028175557731.png" alt="image-20241028175557731" style="zoom:80%;" />

> * 将表移动到另一个数据库；重命名的写法变种~
>
>   ```mysql
>   # 语法
>   rename table 原表名 to 库名.表名;
>   # 使用：把test库下的company表移动到text库下面去
>   rename table company to text.company;
>   ```
>
>   <img src="/images/mysql/image-20241028175847773.png" alt="image-20241028175847773" style="zoom:80%;" />

## 查

### 表

> * 查看该库下所有表
>
>   ```mysql
>   show tables;
>   ```
>
>   <img src="/images/mysql/image-20241028170053070.png" alt="image-20241028170053070" style="zoom:80%;" />

> * 查询符合条件的表
>
>   ```mysql
>   # 语法
>   show tables [like 'pattern']
>   # 查找以y字母开头的表
>   show tables like 'y%';
>   ```
>
>   <img src="/images/mysql/image-20241028170229230.png" alt="image-20241028170229230" style="zoom:80%;" />

> * 查询指定数据库下的所有表
>
>   ```mysql
>   # 语法
>   show tables from 表名
>   # 查找test数据库下的表
>   show tables from 'test'
>   ```
>
>   <img src="/images/mysql/image-20241028170404944.png" alt="image-20241028170404944" style="zoom:80%;" />

### 表结构

> * 展示的信息：最详细，相当于展示出建表语句
>
>   ```mysql
>   # 语法
>   show create table 表名;
>   # 使用
>   show create table student;
>   ```
>
>   <img src="/images/mysql/image-20241028171946431.png" alt="image-20241028171946431" style="zoom:80%;" />

> * 展示的信息：列名、数据类型、是否允许NULL值等，主要是展示字段的信息~
>
>   ```mysql
>   # 语法
>   describe 表名; 
>   desc 表名; # 简写形式
>   # 使用
>   desc student;
>   ```
>
>   <img src="/images/mysql/image-20241028172155873.png" alt="image-20241028172155873" style="zoom:80%;" />

> * 展示的信息：在`mysql`内和`desc`展示的内容一致，在其它数据库中可能是其它意思，例如：`PostgreSQL`（用于查询计划）**tip**：来自`gpt`的回答~
>
>   ```mysql
>   # 语法
>   explain 表名; 
>   # 使用
>   explain student
>   ```
>
>   <img src="/images/mysql/image-20241028172155873.png" alt="image-20241028172155873" style="zoom:80%;" />

> * 展示的信息：和`desc`也一致，可以使用`LIKE`子句进行一个筛选，也就是过滤，也是针对字段展示~
>
>   ```mysql
>   # 语法
>   show columns from 表名 [like 'pattern']
>   # 使用：查找以n字母开头的列
>   show columns from student like 'n%';
>   ```
>
>   <img src="/images/mysql/image-20241028172641835.png" alt="image-20241028172641835" style="zoom:80%;" />

> * 展示的信息：显示一个或多个表的详细状态信息。它提供了比 `desc`或 `show columns`更全面的信息
>
>   ```mysql
>   # 语法
>   show table status [from db_name] [like 'pattern']
>   # 使用：显示当前数据库中所有表的状态
>   show table status;
>   # 显示指定数据库中所有表的状态
>   show table status from test;
>   # 显示名称以 's' 开头的表的状态
>   show table status like 's%';
>   ```
>
>   <img src="/images/mysql/image-20241028173117437.png" alt="image-20241028173117437" style="zoom:80%;" />