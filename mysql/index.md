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

## 函数

### 聚合函数

## 存储过程

## 事务

## 索引

## 锁
