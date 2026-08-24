# 9 SQL 中的空值

我们可以使用 `INSERT INTO` 把 `NULL` 值插入表中

例如：

```sql
INSERT INTO SomeTable values (value1, NULL, value3, ...);
```

- 如果不希望某一列出现 `NULL` 值，可以在表定义中使用 `NOT NULL` 约束
- 可以使用 `IS NULL` 将某个值与 `NULL` 进行比较

**未知真值（The Unknown Truth Value）**

- 存在 `NULL` 值时，我们有时无法判断某个条件是真是假
  - 此时它的值为"未知（unknown）"
- `WHERE` *不会*包含真值未知的行
  - `NATURAL JOIN` 同理，因为它本质上是笛卡尔积后接一个 `WHERE` 条件

**NULL 值对聚合的影响**

- 对某一列进行聚合时，该列中的 `NULL` 值会被忽略

| | 列 A 中部分为 NULL | 列 A 全部为 NULL |
|---|---|---|
| min(A) | 忽略 NULL | null |
| max(A) | 忽略 NULL | null |
| sum(A) | 忽略 NULL | null |
| avg(A) | 忽略 NULL | null |
| count(A) | 忽略 NULL | 0 |
| count(*) | 统计所有元组 | 统计所有元组 |

- 边界情况在不同 DBMS 中处理方式不同
  - 例如：在 `NATURAL JOIN` 时，两个 `NULL` 值是否被认为彼此不同？

**NULL 值对布尔条件的影响**

真值表：

| A | B | A and B | A or B |
|---|---|---|---|
| T | T | T | T |
| T/F 或 F/T | | F | T |
| F | F | F | F |
| T/U 或 U/T | | U | T |
| F/U 或 U/F | | F | U |
| U | U | U | U |

| A | not A |
|---|---|
| T | F |
| F | T |
| U | U |

- 如果我们写 `WHERE condition OR NOT condition`，`NULL` 值会被排除

**小结**

- 任何与 `NULL` 的比较都会得出真值"未知"
- `WHERE` 只接受 `TRUE`（`NATURAL JOIN` 同理）
- 聚合会忽略 `NULL` 值
- 对于其他涉及 `NULL` 行为的情形，请查阅所用 DBMS 的文档，因为不同 DBMS 的行为可能不同

---
