# 6 使用 HAVING 过滤分组

`HAVING` 允许我们对聚合后的值进行过滤

- 语法：

```sql
...
GROUP BY <attributes>
HAVING <condition>
```

- 只有满足条件的分组会被保留

例如：

```sql
SELECT oid, avg(grade), count(*)
FROM Took
GROUP BY oid
HAVING count(*) > 1
```

- 也可以根据某个未被聚合的属性的值进行过滤（只要该属性在 `GROUP BY` 子句中）
- 可以对不在 `SELECT` 子句中的内容进行过滤
  - `HAVING` 在 `SELECT` 子句*之前*执行

**HAVING 子句的限制**

- 只能引用被聚合的属性，或者是 `GROUP BY` 列表中的属性

**执行顺序**

1. `FROM` 子句
   - 决定要检查哪些表
2. `WHERE` 子句
   - 过滤行
   - 由于 `SELECT` 尚未执行，我们无法引用它定义的任何列名（例如重命名）
3. `GROUP BY` 子句
   - 将行组织成分组，每个分组会在结果表中对应一行
4. `HAVING` 子句
   - 过滤分组
   - 由于这发生在 `SELECT` 子句*之前*，它*可以*引用未包含在 `SELECT` 中的属性
5. `SELECT` 子句
   - 选择要包含在结果中的列
   - 可能会引入新的列名
6. `ORDER BY` 子句
   - 对结果表的行排序
   - 由于它出现在 `SELECT` 子句*之后*，可以引用其中引入的列名

---
