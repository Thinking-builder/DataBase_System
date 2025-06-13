
# SQL Introduction

## 数据库设置
不区分大小写

```SQL
# 创建数据库
CREATE DATABASE IF NOT EXISTS mydb
# 使用字符集
DEFAULT CHARACTER SET utf8mb4 
# 设置排序规则
DEFAULT COLLATE utf8mb4_unicode_ci;
# 注意这里最后才是分号，上面三句是一句SQL

# 查看字符集
SHOW CHARACTER SET;
# 查看排序规则
SHOW COLLATION;

# 使用数据库
USE mydb;
```

## 表格操作

### 创建表格
if not exists不必须
```SQL
CREATE TABLE IF NOT EXISTS Sells(
    bar CHAR(20),
    beer VARCHAR(20),
    price REAL,
    PRIMARY KEY(bar, beer) -- 这里的bar和beer是联合主键
);
```

### DROP
```SQL
DROP TABLE IF EXISTS Sells; -- 删除表格

DROP DATABASE IF EXISTS mydb; -- 删除数据库

DROP VIEW IF EXISTS view_name; -- 删除视图

DROP INDEX IF EXISTS index_name ON table_name; -- 删除索引

DELETE FROM Sells; -- 删除表格中的所有数据，但是需要开启设置
```

### 增加列属性
```SQL
ALTER TABLE Sells ADD COLUMN quantity INT DEFAULT 0; -- 增加一个列属性，0是默认值
ALTER TABLE Sells DROP COLUMN quantity; -- 删除一个列属性
```

### 查看数据库表
```SQL
SHOW TABLES; -- 查看当前数据库中的所有表格
DESCRIBE Sells; -- 查看表格的结构
```
## 数据类型
SQL中有很多数据类型，下面是一些常用的数据类型：
+ `CHAR(n)`：固定长度的字符串，长度为n
+ `VARCHAR(n)`：可变长度的字符串，最大长度为n
+ `INT/INTEGER`：整数类型
+ `SMALLINT`：短整数类型
+ `NUMERIC(p, s)`：定点数类型，p为总位数，s为小数位数
+ `REAL`：浮点数类型
+ `DOUBLE PRECISION`：双精度浮点数类型
+ 
## Integrity Constraints

+ Unique：不允许出现重复值，但是可以存在空值（空值也不能重复出现）
+ PRIMARY KEY：唯一标识一行，不允许有空值

一些数据库管理系统，会自动为主键创建索引，但是不会为唯一键自动创建索引。

例如下面第一个例子，我们的(bar,beer)其实可以任意组合取空值，只要表中没有这样的组合，都可以。

```SQL
CREATEN TABLE Sells(
    bar CHAR(20),
    beer VARCHAR(20),
    price REAL,
    UNIQUE(bar,beer)
);
-- 注意下面的代码和上面的代码不一样

CREATEN TABLE Sells(
    bar CHAR(20) UNIQUE,
    beer VARCHAR(20) UNIQUE,
    price REAL
);

```

## REFERENCES Constraints

接下来我们写外键约束：`REFERENCES`关键字。

+ 在声明一个属性时候，使用`REFERENCES`关键字，指定它引用的表和属性。
`RERFERENCES <relation> (<attribute>)`

```SQL
CREATE TABLE Beers(
    name CHAR(20) PRIMARY KEY,
    manf CHAR(20) 
);
CREATE TABLE Sells(
    bar CHAR(20),
    beer VARCHAR(20) REFERENCES Beers(name),
    price REAL,
);
```

+ 作为表的一个元素单独声明

```SQL
CREATE TABLE Beers(
    name CHAR(20) PRIMARY KEY,
    manf CHAR(20) 
);
CREATE TABLE Sells(
    bar CHAR(20),
    beer CHAR(20),
    price REAL,
    FOREIGN KEY(beer) REFERENCES Beers(name)
);
```

外键约束的好处：根本在于**参照完整性**。

1. 保证数据的一致性，我们会采取三种方法，

第一种方法：默认处理方式，如果违背就拒绝指令实行，维持数据库现状并且输出报错

第二种方法：`CASCADE`集连方法

**集连删除**：如果一个表的外键指向另一个表，那么如果删除了另一个表中的数据，那么该外键也会被删除。
**集连更新**：如果一个表的外键指向另一个表，那么如果更新了另一个表中的数据，那么该外键也会被更新。

```SQL
CREATE TABLE Sells(
    bar CHAR(20),
    beer CHAR(20),
    price REAL,
    FOREIGN KEY(beer) REFERENCES Beers(name) ON DELETE CASCADE ON UPDATE CASCADE 
);
```

最后一种方法：`SEL NULL`来保持数据一致性

```SQL
CREATE TABLE Sells(
    bar CHAR(20),
    beer CHAR(20),
    price REAL,
    FOREIGN KEY(beer) REFERENCES Beers(name) ON DELETE SET NULL
```

## Attribute Constraints

+ `NOT NULL`：属性不能为空
+ `DEFAULT <value>`：属性的默认值

```SQL
CREATE TABLE Sells(
    bar CHAR(20) NOT NULL,
    beer VARCHAR(20) NOT NULL,
    price REAL DEFAULT 0.0
```

+ `CHECK <condition>`：属性值的限制条件

```SQL
CREATE TABLE Sells(
    bar CHAR(20) NOT NULL,
    beer VARCHAR(20) CHECK(beer IN SELECT name FROM Beers),
-- 这个和外键不太一样，因为它只能在对Sells表进行操作的时候才会检查，而外键可以保证数据的一致性。
    price REAL DEFAULT 0.0,
    CHECK(price >= 0.0)
);
```

多个属性的CHECK会按照表格的每一行进行Check。再次提醒，CHECK只能对所在表进行检查。也就是说，当参照的表修改时，所在表并不修改，只有等所在表修改时，才会check。

```SQL
CREATE TABLE Sells(
    bar CHAR(20) NOT NULL, 
    beer VARCHAR(20) NOT NULL,
    price REAL DEFAULT 0.0,
    CHECK(bar = 'Joe''s Bar' OR price <= 10.0)
);
```

还可以再创建表之后在加check约束
```SQL
ALTER TABLE Sells ADD CHECK(price >= 0.0); -- 添加一个检查约束
```

## SQL Queries & SQL Injection

攻击者输出入Admin ‘--

## Pattern 通配查询

使用`LIKE`关键字进行查询。下面我们给一些具体的例子来介绍。

```SQL
SLEECT *
FROM employees
WHERE emp_name LIKE '李%';
```

+ `%`：匹配任意字符
+ `_`：匹配单个字符

```SQL
select *
from workson
where job like '%'; -- 匹配所有job但不包括NULL值

-- 如果你要查找所有空的值的记录
where job is NULL;
```

## Three Valued Logic

只有条件成立的时候(True)，才会返回，不包括`NULL`和`Unknown`。

The logic of conditions in SQL is really 3-valued logic: `TRUE = 1`, `FALSE = 0`, `UNKNOWN = 1/2`.

```SQL
AND = MIN; OR = MAX, NOT(x) = 1-x.
```

当NULL值参与比较时，返回`UNKONWN`。

```SQL
SELECT *
FROM employees
WHERE emp_name = '李四' AND emp_age > 30; -- 如果emp_age是NULL，则返回NULL
```

注意，LIKE '%'不包含NULL值。

## 排序

ASC、DESC

## 输出排序

我不再强调

## Muti-releation Queries

笛卡尔积

## Subselect
+ (select ...)

+ EXISTS

+ IN

+ NOT IN

+ ANY

+ ALL

## 交集、并集、差
下面的语句里面，要求左右子查询的列数和类型必须相同。

这里使用这些相当于先使用DISTINCT之后在做操作，原本里面的重复项都会消失

```SQL
-- INTERSECT 取交集
(SELECT * FROM table1)
INTERSECT (SELECT * FROM table2); -- 交集

-- UNION 取并集
(SELECT * FROM table1)
UNION (SELECT * FROM table2); -- 并集

-- EXCEPT 取差集，就是在左边但是除去在右边出现的
(SELECT * FROM table1)
EXCEPT (SELECT * FROM table2); -- 差集
```

## DISTINCT
可以用来消除重复的行，但是这个操作花销比较大，谨慎使用。

## 聚合函数

所有聚合操作都会忽略NULL值，但是COUNT(*)会计算所有行，包括NULL值。

+ `COUNT(*)`：计算行数
+ `COUNT(column_name)`：计算非空值的行数
+ `SUM(column_name)`：计算列的总和
+ `AVG(column_name)`：计算列的平均值
+ `MIN(column_name)`：计算列的最小值
+ `MAX(column_name)`：计算列的最大值

## 分组操作
分组操作使用`GROUP BY`关键字，可以对查询结果进行分组，并对每个分组进行聚合操作。

```SQL
SELECT bar, COUNT(*)
FROM Sells
GROUP BY bar; -- 按照bar分组，计算每个bar的销售数量
```

## Having 子句
`HAVING`子句用于对分组后的结果进行过滤，通常与`GROUP BY`一起使用。

```SQL
SELECT bar, COUNT(*)
FROM Sells
GROUP BY bar
HAVING COUNT(*) > 10; -- 只保留销售数量大于10的bar
```

