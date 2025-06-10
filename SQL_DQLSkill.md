
# SQL DQL Skill

## 充分理解多表查询

多表查询一般的形式如下：

+ `FROM TABLE1, TABLE2, TABLE3`
+ `TABLE1 JOIN TABLE2`
+ `TABLE1 NATURAL JOIN TABLE2`

**理解多表查询的本质其实就是合并成为一张大的表格**。

## Use of Aggregation Functions

我们现在来关注一个情况，我们想要查询`Sells(*bar,*beer,price)`,我们现在来查询所有酒吧中‘bud’的价格最低的。

最直接的想法可能是下面这个错误的案例：

```sql
SELECT bar,MIN(price)
FROM Sells
WHERE bar = 'bud'
GROUP By bar;
```

但是这样查询的结果是每个酒吧的最小的价格
如果你想当然的去掉`Group By bar`，那么SQL的语意是错误的，对于聚合操作来说，SELECT中必须是盒聚合条件相关的列。

**那么此时我们需要通过子查询构建更多的列实现复杂的查询！**

```sql
SELECT bar,price
FROM Sells
WHERE beer = 'bud' and price <= ALL(SELECT price FROM Sells WHERE bar = 'bud')
```

## 多次查询取交集、并集

这允许我们写多个查询，但是使用这个技巧的出发条件是，**多个查询的属性列是相同的**。此时如果你写子查询会复杂了。

e.g. 属于部门d2并且参与了项目p2的员工的员工号

```sql
(select empno from Employeewhere deptno=‘d2’)
INTERSECT
(select empno from Worksonwhere projectno = ‘p2’);
```
