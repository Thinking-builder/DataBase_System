
# SQL Introduction

## Integrity Constraints

+ Unique：可以有一次空值
+ PRIMARY KEY：唯一标识一行，不允许有空值

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

多个属性的CHECK会按照表格的每一行进行Check。再次提醒，CHECK只能对所在表进行检查。

```SQL
CREATE TABLE Sells(
    bar CHAR(20) NOT NULL, 
    beer VARCHAR(20) NOT NULL,
    price REAL DEFAULT 0.0,
    CHECK(bar = 'Joe''s Bar' OR price <= 10.0)
);
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

## 输出排序

我不再强调

## Muti-releation Queries
