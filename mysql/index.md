# mysql

sql 内容整理，较乱，以后找时间调整下内容。

## 创建表结构

```sql
CREATE TABLE `xxxxx` (
id INT ( 11 ) NOT NULL PRIMARY KEY AUTO_INCREMENT,
name VARCHAR ( 64 ) NOT NULL,
age INT ( 11 ) DEFAULT NULL,
city VARCHAR ( 64 ) DEFAULT '北京'
);
```

-   增加修改删除表里的字段

```sql
  ALTER TABLE xxxxx ADD COLUMN idcard VARCHAR(64) NULL;
  ALTER TABLE xxxxx MODIFY idcard VARCHAR(128) NOT NULL;
  ALTER TABLE xxxxx DROP idcard;
```

-   如何增加约束

    ```sql
    CREATE TABLE `xxxxx` (
    id INT ( 11 ) NOT NULL,
    name VARCHAR ( 64 ) NOT NULL,
    age INT ( 11 ) DEFAULT NULL,
    city VARCHAR ( 64 ) DEFAULT '北京'
    );
    ```

    -   增加主键

        ```sql
        ALTER TABLE xxxxx ADD PRIMARY KEY(id);
        ```

    -   增加唯一约束索引

    ```sql
       ALTER TABLE xxxxx ADD UNIQUE INDEX idx_student_idcard(idcard);
    ```

    -   增加默认约束

    ```sql
       ALTER TABLE xxxxx MODIFY COLUMN city VARCHAR(64) DEFAULT '西藏';
    ```

    -   外键

    ```sql
       ALTER TABLE score ADD CONSTRAINT fk_score_student_id FOREIGN KEY(student_id) REFERENCES student(id);
    ```

-   SQL
    -   DDL Data Define Language 数据定义语言，定义数据库和表的数据结构的。
    -   DML Data Manipulate Language 数据库操作语言

## 增删改查

### 普通查询

```sql
SELECT 		<列名>
FROM 			<表名>
[WHERE 		<查询条件表达式>]
[ORDER BY <排序的列名>[ASC 或者 DESC]]
```

eg: 显示部分，更多查看[查询示例](./query.md)

```sql
-- 查询北京的同学信息 并且按照ID正序排列
SELECT id,name
FROM student
WHERE city='北京'
ORDER BY id DESC
```

```sql
-- 别名
SELECT id,name,city home
FROM student
WHERE city='北京'
ORDER BY id DESC
```

```sql
-- 如何查询空行
SELECT *
FROM student
WHERE level IS NULL
```

### 子查询

-   出现在其他 sql 语句中的 SELECT 语句，必须始终出现在圆括号中
-   子查询可以包含多个关键字和条件
-   子查询的外层查询可以是： SELECT INSERT UPDATE SET 等
-   子查询可以返回常量，一行数据，一列数据或其他子查询

-   比较运算符的子查询 - = 等于 - /> 大于 - < 小于 - />= 大于等于 - <= 小于等于 - <> 不等于 - != 不等于 - <=> 安全不等于

eg: 显示部分，更多查看[查询示例](./query.md)

```sql
-- 查询年龄大于平均年龄的同学
SELECT * FROM student WHERE age > (SELECT AVG(age) FROM student);
```

### 分组查询

分组查询 就是按照某列的值进行分组，相同的值分成一组，然后对此进行求平均，和等计算.

```sql
SELECT 列名，查询表达式
FROM <表名>
WHERE <条件>
GROUP BY <分组字段>
HAVING 分组后的过滤条件
ORDER BY 列名 [ASC,DESC]
LIMIT 偏移量，条数
```

> SELECT 列表中只能包含:
>
> -   被分组的列
> -   为每个分组返回一个值的表达式，如聚合函数

eg:显示部分，更多查看[查询示例](./query.md)

```sql
-- 统计每位学生的平均分
SELECT student_id,AVG(grade)
FROM score
-- WHERE student_id = 1
GROUP BY student_id
```

### 增

```sql
INSERT [INTO] 表名 [(列名)] VALUES (值列表)
```

eg:

```sql
INSERT INTO student ( name, idcard, age, city )
VALUES
	( '马斯', '123', 12, '陕西' );
```

### 改

```sql
UPDATE 表名 SET 列明 = 更新值 [WHERE 更新条件]
```

-   一次更新多列 可以用逗号隔开
-   可以指定更新条件，多个条件可以使用 AND OR NOT 修饰
    eg:

```sql
UPDATE student SET age=40,city='上海'  WHERE id = 5 AND name = '马斯';
```

### 删除

```sql
-- 删除整行删除，不需要列名。
-- 如果删除的表是主表，需要先删除子表。
DELETE FROM student WHERE id = 5;
```

删除表时：

-   不会重置标识种子
-   会写入日志

```sql
DELETE FROM student;
```

### 截断

```sql
-- TRUNCATE
TRUNCATE table student;
```

与 DELECT 不同，TRUNCATE 会：

-   重置标识种子（自增）
-   没有备份，不会写入日志

## 表连接

-   INNER JOIN 内连接
-   LEFT JOIN 左外连接
-   RIGHT JOIN 右外连接
-   ON 连接条件 也可以使用 WHERE 来代替 ，也可以使用 WHEREF 来对结果进行过滤

eg:

```sql
-- 查询学生姓名 和学生的分数 表连接
SELECT * FROM student INNER JOIN score ON student.id = score.student_id;
SELECT * FROM student,score
WHERE student.id = score.student_id
-- 左外连接
SELECT * FROM student LEFT JOIN score ON student.id = score.student_id
-- 右外连接
SELECT * FROM student RIGHT JOIN score ON student.id = score.student_id
-- 外连接
 SELECT * FROM student OUTER JOIN score ON student.id = score.student_id -- mysql 不支持

 -- 多表连接
 -- 学生姓名 课程名 分数
 SELECT student.name,course.name,score.grade FROM student,course,score
 WHERE score.student_id = student.id AND score.course_id = course.id

  SELECT student.name,course.name,score.grade
	FROM score INNER JOIN student ON score.student_id = student.id
	INNER JOIN course ON score.course_id = course.id

```

### 无限分类

当表中存在分类，查找分类时可以连接自身进行查询。具体查看[无限分类](./Infinite_classification.md)

## 函数-条件语句

### 条件语句

-   if

```sql
-- if
SELECT IF(1>0,true,false); -- 1
```

-   switch

```sql
-- CASE
/** CASE case_value
	WHEN when_value THEN
		statement_list
	ELSE
		statement_list
END CASE;
*/
```

eg:

```sql
-- 表没附带 但是这里很简单，没难度。
SELECT
CASE age
	WHEN 32 THEN
		'老3'
	WHEN 40 THEN
		'老4'
	ELSE
		CONCAT('其他',age)
END
FROM student; -- 老4 其他12 其他123 老3 老3 老3

SELECT
CASE
	WHEN age > 40 THEN
		'老年'
	WHEN age > 30 THEN
		'中年'
	ELSE
		CONCAT('垂暮年龄：',age)
END
FROM student; -- 中年 垂暮年龄：12 老年 中年 中年 中年
```

### 自定义函数

-   自定义函数是对 mysql 的扩展，使用方式和内置函数相同
-   函数必须要有参数和返回值
-   函数可以接受任意类型的值，也可以接受这些类型的参数
-   函数体由合法的 sql 语句组成
-   函数体可以是简单的 SELECT 语句或者 INSERT 语句，如果是复合结构要使用 BEGIN...END
-   函数体也可以包含声明，循环和流程控制
-   返回值只能有一个
-   语法

        ```sql
            CREATE FUNCTION func_name RETURNS {String | Integer}
            body
        ```

    eg:

```sql
SELECT NOW();

CREATE FUNCTION ZNOW() RETURNS VARCHAR(64)
RETURN DATE_FORMAT(NOW(),'%Y年%m月%d日 %H时%i分%s秒');

SELECT ZNOW(); -- 2021年07月07日 23时42分24秒

CREATE FUNCTION FormatNow(format VARCHAR(64)) RETURNS VARCHAR(64)
RETURN DATE_FORMAT(NOW(),format);

SELECT FormatNow('%Y年%m月%d日 %H时%i分%s秒');  -- 2021年07月07日 23时42分40秒
```

-   函数体可能不止一行

```sql
CREATE FUNCTION ADD_USER(name VARCHAR(64)) RETURNS INT
BEGIN
INSERT INTO t(name) VALUES(name);
RETURN LAST_INSERT_ID();
END

SELECT ADD_USER("新加的");
```

### 聚合函数

-   一组值进行计算 返回计算后的值 一般用来统计数据。

    -   聚合函数

        ```sql
        SELECT SUM(grade) FROM score WHERE student_id = 1;
        ```

    -   max min 最大值 最小值
        ```sql
        SELECT MAX(grade),MIN(grade) FROM score;
        ```
    -   AVG 平均值
        ```sql
        SELECT AVG(grade) FROM score;
        ```
    -   总记录数

        ```sql
        SELECT COUNT(grade) FROM score;
        ```

## 存储过程

> Stored Procedure 是一组为了完成特定功能的 SQL 语句集，经过编译后存储在数据库中，用户通过指定存储过程的名字并给定参数来调用执行。

-   CREATE PROCEDURE([[IN | OUT | INOUT] 参数名 数据类型...])
-   调用 存储过程实际是一种函数，调用需要()符号
    ```sql
    CALL Avg_Price()
    ```
-   删除

    ```sql
    DROP PROCEDURE IF EXISTS Avg_Price;
    ```

-   参数
-   存储过程并不显示结果，而是把结果返回给你指定的变量

```sql
CREATE PROCEDURE SUM1(in a INT,in b INT,OUT result INT)
BEGIN
SELECT a+b INTO result;
END;

CALL SUM1(1,2,@result);
SELECT @result;
```

-   函数都且只能由一个返回值 RETURN
-   存储过程没有返回值，存储也是函数的一种。
-   缺点 不兼容 不通用

## 事务

-   原子性 一致性 隔离性 持久性
-   提交 COMMIT
-   回滚 ROLLBACK

```sql
CREATE TABLE account(
id INT PRIMARY KEY AUTO_INCREMENT,
name VARCHAR(64),
balance DECIMAL(10,2)
)

SELECT * FROM account;

UPDATE account SET balance = balance - 10 WHERE id =1;
UPDATE account SET balance = balance + 10 WHERE id =2;
-- 原子性 一致性 隔离性 持久性
SET AUTOCOMMIT = 0;

BEGIN
UPDATE account SET balance = balance - 10 WHERE id = 1;
UPDATE account SET balance = balance + 10 WHERE id =2;
COMMIT;
ROLLBACK;
```

## 索引

-   查看索引

```sql
SHOW INDEX FROM user;
```

-   删除索引

```sql
ALTER TABLE 表名 DROP PRIMARY
ALTER TABLE DROP FROM index 索引名字
DROP INDEX 索引名 on 表名
```

-   慢查询

    -   EXPLAIN

-   索引原则

    -   比较频繁作为查询条件的字段应该创建索引
    -   唯一性太差的字段不适合单独创建索引，即使频繁作为查询条件。
    -   更新非常频繁的字段不适合做创建索引
    -   不会出现在 where 子句中的字段不该创建索引。

## 锁

锁是计算机协调多个进程或线程并发访问某一资源的机制。

-   分类
    -   从数据库操作的类型分：读锁和写锁
        -   读锁（共享锁）：针对同一份数据，多个读操作可以同时进行而不会相互影响。
        -   写锁（排他锁）：当前写操作没有完成前，他会阻断其他写锁和读锁。
    -   从数据操作的粒度分
        -   表锁
        -   行锁

### 表锁

-   表锁偏向 MyISAM 存储引擎，开销小，加锁快，锁定力度大，发生锁冲突的概率最高，并发度最低。

### 行锁

-   偏向 InnoDB 引擎，开销大，加锁慢，会出现死锁，锁定粒度最小，发生锁冲突的概率最低。并发度也高。
-   innoDB 与 MYISAM 的最大不同有两点：一是支持事务（TRANSACTION）;二是采用了行级锁。

### 并发事务处理带来的问题

-   更新丢失
    -   当两个或者多个事务选择同一行，然后基于最初选定的值更新该行时，由于每个事务都不知道其他事物的存在，就会发生丢失更新的问题-最后的更新覆盖了由其他事务所作的更新。
    -   后面的事务覆盖了前面的值。
-   脏读 dirty read
    -   一个事务正在对一条记录做修改，在这个事务完成并提交前，这条记录的数据就处于不一致的状态；这时，另一个事务也来读取同一条记录，如果不加控制，第二个事务读了这些脏数据，并据此做进一步处理。就会产生未提交的数据依赖关系。这种现象被叫做脏读。
    -   事务 A 读取了事务 B 已经修改但尚未提交的数据，还在这个数据基础上做了操作。此时如果事务 B 回滚，A 读取的数据无效，不符合一致性的要求。
    -   脏读时事务 B 里面修改了数据，这是不正常的。
    -   解决：在第一个事务提交前，任何其他事务不可读取其修改过的值，则可避免该问题。
-   不可重复读
    -   一个事务在读取某些数据后的某个时间，再次读取以前读取过的数据，却发现其读出的数据已经发生改变，或某条记录已经被删除。这种现象称作 不可重复读
    -   事务 A 读取到了事务 B 已经提交的修改数据，不符合隔离性。
    -   解决：只有在修改事务完全提交之后才能读取数据，可以避免。
-   幻读
    -   事务 A 首先根据条件索引得到 N 条数据，然后事务 B 改变了这 N 条数据之外的 M 条或者增添了 M 条符合事务 A 搜索条件的数据，导致事务 A 再次搜索发现有 N+M 条数据了，就产生了幻读。
    -   幻读时事务 B 新增了数据
    -   解决： 操作事务完成数据处理之前，任何其他事务都不可以添加新数据，则可避免。
-   事务隔离级别

    -   read uncommitted：读取尚未提交的数据：就是脏读
    -   read committed: 读取已经提交的数据：可以解决脏读
    -   repeatable read：重读读取：可以解决脏读和不可重复读 --- mysql 默认的
    -   serializable：串行化：可以解决脏读。不可重复读和幻读 --- 相当于锁表
    -   查询隔离级别

        ```sql
        SELECT @@tx_isolation;
        ```

    -   设置隔离级别

        ```sql
        SET SESSION TRANSACTION ISOLATION level 事务级别
        ```

-   死锁
    -   deadlock found when trying to get lock;try restarting transaction; 试图加锁时发现死锁，请尝试重新启动事务;
    -   索引

## 外键

外键约束（也称为引用约束或引用完整性约束）可定义表间以及表内必需的关系。

引用约束的目的是保证表关系得到维护并遵循数据输入规则。这意味着，只要引用约束有效，数据库管理器就保证，对于子表中其外键列中具有非空值的每行，相应父表中都存在一个其父键中具有匹配值的行。

外键备注:参考[IBM](https://www.ibm.com/docs/zh/db2/11.1?topic=constraints-foreign-key-referential)
