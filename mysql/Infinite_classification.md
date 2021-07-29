# 无限分类

## 首先创建分类表

```sql
CREATE TABLE category(
	id int PRIMARY KEY AUTO_INCREMENT NOT NULL,
	name VARCHAR(64),
	parent_id int
);

INSERT INTO category(id,name,parent_id)
VALUES (1,'数码产品',0),(2,'服装',0),(3,'食品',0),
(4,'iPad',1),(5,'李宁',2),(6,'康师傅',3);

SELECT * FROM category;
```

## 查询

-   查询所有顶级分类下面的类别的数量

```sql
SELECT c1.id,COUNT(*) FROM category c1
INNER JOIN category c2
ON c1.id = c2.parent_id
WHERE c1.parent_id = 0
GROUP BY c1.id;
```

-   要把所有的父 ID 变成名称

```sql
SELECT c1.id,c1.name,c2.name 父分类名称 FROM category c1
INNER JOIN category c2
ON c1.parent_id = c2.id;
```

-   删除重复记录

```sql
INSERT INTO category(id,name,parent_id)
VALUES (7,'iPad',1),(8,'李宁',2),(9,'康师傅',3);
INSERT INTO category(id,name,parent_id)
VALUES (10,'iPad',1),(11,'李宁',2),(12,'康师傅',3);

SELECT * FROM category;
	-- 查出来要删除的id
	SELECT * FROM category c1 LEFT JOIN
	(SELECT id,name FROM category GROUP BY name HAVING COUNT(*) > 1) c2
	ON c1.name = c2.name
	WHERE c1.id != c2.id

	-- 可以用IN NOT IN 实现
	SELECT * FROM category c1
	WHERE c1.name IN
	(SELECT name FROM category GROUP BY name HAVING COUNT(*) > 1)
	AND c1.id NOT IN
	(SELECT MIN(id) FROM category GROUP BY name HAVING COUNT(*) > 1)

	-- 真正的删除
	DELETE FROM category
	WHERE name IN
	(SELECT name FROM(SELECT name FROM category GROUP BY name HAVING COUNT(*) > 1) t1)
	AND id NOT IN
	(SELECT id FROM(SELECT MIN(id) id FROM category GROUP BY name HAVING COUNT(*) > 1) t2)
```

-   多表更新

```sql
CREATE TABLE province(id int PRIMARY KEY AUTO_INCREMENT,name VARCHAR(64))
SELECT * FROM province;

INSERT INTO province(name) SELECT DISTINCT province FROM student;

-- 更新省份
UPDATE student INNER JOIN province ON student.province = province.name
SET student.province = province.id;

ALTER TABLE student MODIFY province int NOT NULL;

ALTER TABLE student
CHANGE COLUMN province province_id int(11) NOT NULL AFTER city;
SELECT * FROM student;

-- city
CREATE TABLE city(id int PRIMARY KEY AUTO_INCREMENT,name VARCHAR(64))
-- 插入学生表city 到 city
INSERT INTO city(name) SELECT DISTINCT city FROM student;
SELECT * FROM city;
-- 更新city
UPDATE student INNER JOIN city ON student.city = city.name
SET student.city = city.id;

ALTER TABLE student CHANGE COLUMN city city_id int NOT NULL AFTER age;


DESC category;

```
