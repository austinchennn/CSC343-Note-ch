# 4 SQL 入门

SQL：结构化查询语言（Structured Query Language）

- DDL：数据定义语言（Data Definition Language），用于定义模式
- DML：数据操作语言（Data Manipulation Language），用于编写查询和修改数据库
- 本课程使用 PostgreSQL

**SELECT-FROM-WHERE 查询**

例如：

```sql
SELECT name       -- 选择名为 "name" 的列
FROM Course        -- 从 Course 表中
WHERE dept = 'CSC'; -- 只选择满足条件的行
```

- `WHERE` 相当于关系代数中的 $\sigma$
  - 可以使用逻辑运算符 `=, <, >, <=, >=, !=, <>`
  - `!=` 和 `<>` 都表示"不等于"
  - 可以使用布尔运算符 `AND, OR, NOT`
- `SELECT` 相当于 $\Pi$

**笛卡尔积**

- 在 `FROM` 子句中用逗号分隔列出 2 个或更多的表
- 例如：`FROM Course, Offering, Took`

**临时重命名表**

例如：

```sql
SELECT e.name, d.name
FROM Employee e, Department d;
```

- 相当于关系代数中的 $\rho$
- 自连接（self-join）时*必须*重命名

**SELECT 子句中的通配符**

- 如果希望结果包含所有列，可以使用通配符

```sql
SELECT *
FROM ...
```

**命名列**

- 可以使用 `AS` 表达式为查询结果中的列指定名称

例如：

```sql
SELECT name AS title, dept
FROM ...
```

- 如果无法确定列名（例如某个表达式），结果列名会显示为 `?column?`

**排序**

- 可以在 select-from-where 查询末尾添加 `ORDER BY` 子句对结果排序

例如：

```sql
SELECT sid, grade
FROM Took
WHERE grade > 90
ORDER BY grade;
```

- 默认按升序排列
- 可以添加 `DESC` 改为降序，例如 `ORDER BY grade DESC`
- 可以根据多个列排序，例如 `ORDER BY grade, sid`
- 可以使用表达式的值来决定排序，例如 `ORDER BY grade + sid`
- `ORDER BY` 在 `SELECT` 之前执行，因此所有属性都可用

**SELECT 子句中的表达式**

- 可以在 `SELECT` 子句中使用表达式
  - 操作数：属性、常量
  - 操作符：算术运算符、字符串运算符

例如：

```sql
SELECT sid, grade + 10 AS adjusted
FROM ...
```

例如：

```sql
SELECT dept || cnum
FROM ...
```

- `||` 是字符串连接符

**常量表达式**

- `SELECT` 子句中可以在某一列使用常量值，例如：

```sql
SELECT name, 'satisfies' as breadthRequirement
FROM Course
WHERE breadth;
```

- 该查询只提取 `breadth` 值为真的课程
- 结果中第二列每一行的值都是 `satisfies`

**模式匹配**

- `LIKE` 操作符可以将字符串与模式进行比较
- 记法：`<attribute> LIKE <pattern>` 或 `<attribute> NOT LIKE <pattern>`
- 模式是一个带引号的字符串，可以包含以下特殊字符：
  1. `_` 匹配任意单个字符
  2. `%` 匹配任意（0 个或多个）字符组成的序列

例如：

```sql
SELECT *
FROM Course
WHERE name LIKE '%to%';
```

- 结果会包含 'Intro to Databases'、'Intro to Machine Learning' 等

- `~` 操作符支持用正则表达式进行字符串匹配

例如：

```sql
SELECT *
FROM Student
WHERE surname ~ '(M|F|L)a*'
```

- 正则表达式可能会比较慢

**大小写敏感性与空白字符**

- 关键字（例如 `SELECT`）不区分大小写
- 标识符（即表名或列名）不区分大小写
- 字面字符串（例如 `'StG'`）区分大小写，且必须使用单引号
- 换行和制表符会被 SQL 忽略
- 合理的查询格式：
  1. 每个子句单独一行
  2. 关键字全部大写
  3. 表名首字母大写
  4. 列名使用小写

**SQL 是一门高级语言**

- 我们只关心想从数据库中得到什么，而不关心如何得到它
- DBMS 可以改变数据的存储方式，而不影响我们的查询
- 这就是所谓的**物理数据独立性（physical data independence）**

---
