# sql 查询示例

普通查询含部分运算

## 普通查询

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

```sql
-- 常量列
SELECT id,name,city,'中国' as country
FROM student
WHERE city='北京'
ORDER BY id DESC
```

```sql
--分页
SELECT id,name,city
FROM student
LIMIT 3,3
```

```sql
-- 查询同学都来自哪些不同的城市
SELECT DISTINCT city
FROM student
```

```sql
-- SQL  + 只能用来加数字
SELECT 1+1;
SELECT 1+'1';
SELECT 1+'a'; -- 1
SELECT 'a'+'b'; -- 0
SELECT CONCAT('a','b','c'); -- abc
```

```sql
-- 模糊查询
SELECT * FROM student
WHERE name LIKE'马%'; -- 	%0个或多个字符

SELECT * FROM student
WHERE name LIKE'马_'; -- _代表一个字符
```

```sql
--    CONCAT_WS CONCAT_WS 使用指定的分隔符进行字符链接
SELECT CONCAT_WS('-',city, name) FROM student; -- CONCAT_WS使用指定的分隔符进行字符链接 查询结果 上海-李四
```

```sql
-- LOWER(str)小写 UPPER(str) 大写
-- 姓名的首字母大写 其他字符小写 用下划线拼接
-- zhAng - Z_hang
CREATE TABLE t(name VARCHAR(64));
INSERT INTO t(name) VALUES('zhAng');

SELECT
CONCAT_WS('_',UPPER(SUBSTR(name,1,1)),LOWER(SUBSTR(name,2)))
FROM t;

```

```sql
SELECT LEFT("abcdefg",3); -- abc
SELECT LENGTH('abcdefg'); -- 7
-- SUBSTR(str FROM pos FOR len)
SELECT SUBSTR('abcdef' FROM 4 FOR 2); -- de 索引从1开始
-- SUBSTR(str FROM pos)
SELECT SUBSTR('abcdef' FROM 4); -- def
-- SUBSTR(str,pos)
SELECT SUBSTR('abcdef',4); -- def
-- SUBSTR(str,pos,len)
SELECT SUBSTR('abcdef',4,2); -- de
-- indexof
SELECT INSTR('abcdefg','de'); -- 4
```

```sql
-- 如何去空格
SELECT TRIM(' xxx ');
SELECT LTRIM(' xxx ');
SELECT RTRIM(' xxx ');
```

```sql
-- pading 补齐位数
SELECT LPAD('xxx',10,'0'); -- 0000000xxx
SELECT RPAD('xxx',10,'0'); -- xxx0000000
```

```sql
-- 替换
SELECT REPLACE('abcdef','abc','eee'); -- eeedef
```

```sql
-- FORMAT(X,D); 数字的格式化 保留D位小数
SELECT FORMAT(10000,0); -- 10,000.00
-- 数学函数
-- CEIL(X) 向上取整
-- FLOOR(X) 向下取整
-- DIV 整数取
SELECT DIV(3,2);
-- MOD 取余（取模）
-- POWER(X,Y) 幂运算
-- ROUND(X) 四舍五入
SELECT ROUND(2.5); -- 3
-- ROUND(X,D)
-- TRUNCATE(X,D) 数字截取
SELECT TRUNCATE(2.955,2); -- 2.95
```

```sql
-- 日期函数
-- NOW() 当前日期和时间
SELECT NOW(); -- 2021-07-06 23:36:22
SELECT YEAR(NOW()); -- 2021
-- CURDATE() 当前日期
SELECT CURDATE(); -- 2021-07-06
-- CURTIME() 当前时间
SELECT CURTIME(); -- 23:36:59
-- DATE_ADD(date,INTERVAL expr unit) 日期变化
-- DATEDIFF(expr1,expr2) 计算日期差
-- DATE_FORMAT(date,format) 格式化日期
SELECT STR_TO_DATE('2018-09-09','%Y-%m-%d'); -- 2018-09-09 Y大写得到2018 小写得到18
SELECT DATE_FORMAT(NOW(),'%m-%d-%Y'); -- 07-07-2021
SELECT DATE_ADD(NOW(),INTERVAL 365 DAY); -- now: 2021-07-07 23:03:42 -> 2022-07-07 23:03:28
SELECT DATEDIFF('2020-7-7',NOW()); -- 365 now: 2021-07-07 23:03:42
```

```sql
--每个客户端链接上服务器之后 都会有一个链接的ID
SELECT CONNECTION_ID(); -- 2
SELECT DATABASE(); -- studb
SELECT VERSION(); -- 5.7.31-log
SELECT USER(); -- root@localhost

SELECT LAST_INSERT_ID();
SELECT MD5('root'); -- 63a9f0ea7bb98050796b649e85481845
SELECT PASSWORD('root') -- *81F5E21E35407D884A6CD4A731AEBFB6AF209E1B
```

## 子查询示例

```sql
-- 查询年龄大于平均年龄的同学
SELECT * FROM student WHERE age > (SELECT AVG(age) FROM student);
```

```sql
-- any some all
-- ANY 任意一个  ALL 全部 SOME 某些
-- 年龄大于南京任何一位同学
SELECT *
FROM student
WHERE age > ANY (SELECT age FROM student WHERE city='南京')
SELECT *
FROM student
WHERE age > ALL (SELECT age FROM student WHERE city='南京')
SELECT *
FROM student
WHERE age > SOME (SELECT age FROM student WHERE city='南京')
```

```sql
-- 查询有考试成绩的同学 IN 在某个范围内  not in 不在
SELECT *
FROM student
WHERE id IN (SELECT student_id FROM score)
```

```sql
-- exist / not exist 存在/不存在 exist找到就停止 性能比IN高
SELECT *
FROM student
WHERE EXISTS (SELECT * FROM score WHERE score.student_id = student.id);
```

## 分组查询

```sql
-- 统计每位学生的平均分
SELECT student_id,AVG(grade)
FROM score
-- WHERE student_id = 1
GROUP BY student_id
```

```sql
-- 统计每门课程的最高分，并按分数从高到底排列
SELECT course_id,MAX(grade)
FROM score
GROUP BY course_id
ORDER BY MAX(grade) DESC
```

```sql
-- 多列分组 统计各省 男女总人数
SELECT province,gender,COUNT(*)
FROM student
GROUP BY province,gender
```

```sql
-- 分组的筛选
SELECT province,COUNT(*)
FROM student
GROUP BY province
HAVING COUNT(*) > 1;
```

```sql
-- 统计不及格次数大于1的同学
SELECT student_id, COUNT(*) 不及格的次数
FROM score
WHERE grade < 60
GROUP BY student_id
HAVING COUNT(*) > 1;
```
