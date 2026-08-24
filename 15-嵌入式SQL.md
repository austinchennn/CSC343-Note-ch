# 15 嵌入式 SQL

**SQL 的问题**

- 标准 SQL 不是图灵完备的
  - 没有循环或递归
- 无法控制输出的格式
- 大多数用户不应该自己编写 SQL 查询
  - 需要运行*基于用户输入*的查询

**SQL + 一门传统语言**

- 可以通过将 SQL 与传统语言结合来解决上述问题
- 但是 SQL 基于*关系*，而传统语言没有这种类型
- 需要将 SQL 中的元组逐个"喂"给另一种语言，并把每个属性值送入一个特定的变量

**存储过程方式（Approach of Stored Procedures）**

- SQL 标准包含一种定义*存储过程（stored procedures）*的语言，它可以：
  1. 拥有参数和返回值
  2. 使用局部变量、if 语句、循环等
  3. 执行 SQL 查询
- *过程（Procedure）*：函数（function）的旧技术名称
- 定义完成后，存储过程可以按以下方式被使用：
  - 由解释器调用
  - 由 SQL 查询调用
  - 由另一个存储过程调用
  - 作为*触发器（trigger）*执行的动作
- 这不是一种非常标准的方式，因为不同的 DBMS 为存储过程定义了不同的私有语言
  - 例如：PostgreSQL 使用 PL/pgSQL
- 这种方式效率最高，但代码不可移植

**语句级接口方式（Approach of Statement-Level Interface, SLI）**

- 将 SQL 语句嵌入到诸如 C 或 Java 等传统语言的代码中
- 使用预处理器（preprocessor）将 SQL 替换为用宿主语言编写的、调用 SQL 库中所定义函数的代码
- 特殊语法用于标记哪些代码块需要预处理器进行转换

```
用户
 │
 ├─ SLI ──► 宿主语言 + 嵌入式 SQL
 │              │
 │              ▼
 │           预处理器
 │              │
 │              ▼
 │         宿主语言 + 函数调用  ◄── SQL 库
 │              │
 │              ▼
 │         宿主语言编译器
 │              │
 │              ▼
 │           目标代码程序
 │
 └─ CLI ─────────┘
```

**调用级接口方式（Approach of Call-Level Interface, CLI）**

- 我们可以自己编写 SQL 调用
- 无需再进行预处理
- 每种语言都有自己对应的库（例如：C 的 SQL/CLI，Java 的 JDBC，Python 的 psycopg2）

**SQL 注入（SQL Injections）**

- 拼接查询字符串并直接执行它，容易受到注入攻击
- 例如，用户输入为：

```sql
xyz'); DROP TABLE abc;--
```

- 字符串被闭合，`DROP TABLE` 被执行，其余部分变成了注释
- 使用（psycopg2 的）`execute` 方法的第二个参数，在运行时动态补全查询
- 永远不要使用 Python 的字符串拼接（`+`）或字符串参数插值（`%`）向 SQL 查询字符串中传递变量

---
