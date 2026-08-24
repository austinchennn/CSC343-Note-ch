# 13 SQL 模式（Schema）

**SQL 模式**

- 在关系代数中，*模式（schema）* 是关系或一组关系的结构定义
- 在 SQL 中，模式是一个用来防止名称冲突的命名空间
- 默认情况下，所有定义都放在 `public` 模式中
- 可以创建模式

```sql
CREATE SCHEMA eatInToronto;
```

- 要在某个模式内创建表，命名方式为 `schema.table`

```sql
CREATE TABLE eatInToronto.Restaurant (
    name TEXT,
    cuisine TEXT
)
```

- 可以使用全名引用这样的表

```sql
SELECT name
FROM eatInToronto.Restaurant;
```

**搜索路径（The Search Path）**

- 查看当前的搜索路径：

```sql
show search_path;
```

- 我们可以设置搜索路径，以避免用全名引用表

```sql
SET SEARCH_PATH TO university, eatInToronto, public;
```

- 首先查找 `university`，然后是 `eatInToronto`，再然后是 `public`
- 如果我们定义表时没有指定模式，它会被创建在搜索路径中的第一个模式里
- 默认搜索路径是 `$user, public`
  - `$user` 是当前用户的名称，它不会被自动创建
  - 如果它被创建了，则会位于默认搜索路径的最前面
- 可以用以下方式查看当前的搜索路径

```sql
SHOW SEARCH_PATH;
```

**删除模式**

- 我们可以通过以下方式删除模式

```sql
DROP SCHEMA university;
```

- 如果模式中已经定义了任何内容，删除时会产生错误信息
- 如果我们打算删除该模式并移除其中的所有内容，添加关键字 `CASCADE`

```sql
DROP SCHEMA university CASCADE;
```

- 为避免在模式不存在时报错，添加 `if exists`
- 在 DDL 文件的开头，通常先写一条 `DROP SCHEMA` 语句，接一条 `CREATE SCHEMA` 语句，再接一条 `SET SEARCH_PATH` 语句是很有用的做法

```sql
DROP SCHEMA IF EXISTS university CASCADE;
CREATE SCHEMA university;
SET SEARCH_PATH TO university;
```

---
