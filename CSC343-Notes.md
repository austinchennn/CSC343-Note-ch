# CSC343 笔记（中文翻译版）

> 原作者：[Jenci Wei](https://github.com/jenci2114)（多伦多大学 CSC343，2022 秋季学期）
> 本文仅为中文翻译，全部内容与版权归原作者所有，译者未做任何内容增删或修改。

## 目录

1. [简介](#1-简介)
2. [关系模型](#2-关系模型)
3. [关系代数](#3-关系代数)
4. [SQL 入门](#4-sql-入门)
5. [聚合与分组](#5-聚合与分组)
6. [使用 HAVING 过滤分组](#6-使用-having-过滤分组)
7. [视图](#7-视图)
8. [SQL 中的集合与包](#8-sql-中的集合与包)
9. [SQL 中的空值](#9-sql-中的空值)
10. [连接（Join）](#10-连接join)
11. [子查询](#11-子查询)
12. [数据库修改](#12-数据库修改)
13. [SQL 模式（Schema）](#13-sql-模式schema)
14. [数据定义语言（DDL）](#14-数据定义语言ddl)
15. [嵌入式 SQL](#15-嵌入式-sql)
16. [函数依赖理论](#16-函数依赖理论)
17. [数据库设计](#17-数据库设计)
18. [实体/联系模型（E/R Model）](#18-实体联系模型er-model)
19. [从 E/R 模型到数据库模式](#19-从-er-模型到数据库模式)
20. [索引](#20-索引)

---

## 1 简介

DBMS（数据库管理系统，Database Management System）能够高效地创建和管理大量数据，并使其能够长期持久化保存。

- **数据库（Database）**：由 DBMS 管理的数据集合
- 每个 DBMS 都基于某种**数据模型（data model）**，其中包括：
  - 数据的结构
  - 对数据内容的约束
  - 对数据的操作
- DBMS 提供以下能力：
  - 能够（显式地）指定数据的逻辑结构，并强制执行
  - 能够查询或修改数据
  - 在高负载下（即海量数据、大量查询）保持良好性能
  - 数据的持久性（Durability）
  - 支持多个用户/进程并发访问

**关系数据模型（relational data model）** 基于数学中"关系"的概念

- 可以将其理解为由行和列组成的表
- 在正式定义模式（schema）之前无法记录任何数据
- 模式必须被严格遵守
- 所有行都应为所有列提供数据

**其他数据模型**

- 半结构化数据模型（Semi-structured data model）
  - 更加灵活
  - 例如：JSON、XML
- 非结构化数据（Unstructured data）
  - 不符合任何固定结构的数据
  - 例如：文档、视频等
  - 可以存储在键值存储（key-value store）中
- 图数据模型（Graph data model）
  - 一个节点可以表示一个学生或一门课程
  - 一条边可以表示"选修该课程"
  - 查询由路径定义

---

## 2 关系模型

**数学中的关系**

- **域（domain）**是一个值的集合
- 假设 $D_1, \ldots, D_n$ 是若干个域
  - **笛卡尔积（Cartesian product）** $D_1 \times \cdots \times D_n$ 是所有满足 $d_i \in D_i$（对所有 $i \in [1, n]$）的元组 $\langle d_1, \ldots, d_n \rangle$ 组成的集合
- 一个建立在 $D_1, \ldots, D_n$ 上的**（数学）关系（relation）**是该笛卡尔积的一个子集
- 数据库表就是一个关系

**模式（Schema）与实例（Instance）**

- **模式（Schema）**是关系的结构
  - 例如：Teams（球队）有 3 个属性：name（名称）、home field（主场）、coach（教练）。任意两支球队都不能同名
  - 形式化记法：Teams(Name, HomeField, Coach)
- **实例（Instance）**是关系的内容
  - 即表中的数据
- 数据库中的变化
  - 实例经常发生变化
  - 模式（应该）很少发生变化
  - 数据库通常只存储数据的当前版本
    - 记录历史的数据库被称为**时态数据库（temporal databases）**

**术语**

- **关系（Relation）**：表
- **属性（Attribute）**：列
- **元组（Tuple）**：行
- 关系的**元数（Arity）**：属性的数量
- 关系的**基数（Cardinality）**：元组的数量

**作为集合的关系**

- 关系是元组的**集合（set）**，这意味着：
  1. 不能有重复的元组
  2. 元组的顺序不重要
- 我们也可以采用一种关系是**包（bags）**的模型
  - 是集合的一种推广，允许出现重复
  - 商用 DBMS 使用的正是这种模型

**数据库模式与实例**

- **数据库模式（Database schema）**：一组关系模式的集合
- **数据库实例（Database instance）**：一组关系实例的集合

**键（Key）**

- 关系的**键（key）**是一组属性 $a_1, \ldots, a_n$，满足：
  1. 它们组合起来的值是唯一的，即不存在 $t_1, t_2$ 使得 $(t_1.a_1 = t_2.a_1) \wedge \cdots \wedge (t_1.a_n = t_2.a_n)$
  2. $a_1, \ldots, a_n$ 的任何真子集都不具有该性质/都不是键
- 键的记法：下划线
  - 例如：Teams(<u>Name, HomeField</u>, Coach)
- 键定义了一种**完整性约束（integrity constraint）**

**超键（Superkey）**

- **超键（Superkey）**：任何键的超集
- 在数据库理论中有用，但在实践中我们不声明超键

**键 vs. 超键**

- **键（Key）**：一个*最小*的属性集合，使得任意两个元组在这些属性上都不能取相同的值
- **超键（Superkey）**：某个键的超集（不一定是最小的）

**外键（Foreign Keys）**

- 关系之间经常互相引用
- 引用方的属性被称为**外键（foreign key）**，因为它引用的属性在另一张表中是键
- 外键可能需要包含多个属性
- 记法 $R[A]$ 表示关系 $R$ 中所有元组组成的集合，但只保留属性列表 $A$ 中的属性
- 我们用记法 $R_1[X] \subseteq R_2[Y]$ 来声明外键约束
  - $X$ 和 $Y$ 可以是属性列表，且元数相同
  - $Y$ 必须是 $R_2$ 中的一个键

**引用完整性约束（Referential Integrity Constraints）**

- 类似 $R_1[X] \subseteq R_2[Y]$ 的关系是一种**引用完整性约束（referential integrity constraint）**
- 并非所有引用完整性约束都是外键约束
  - 它们不必满足 $Y$ 是 $R_2$ 中*键*这一性质

---

## 3 关系代数

**简介**

- 操作数（Operands）：表
- 操作符（Operators）：
  - 只选择我们想要的行
  - 只选择我们想要的列
  - 合并多张表
  - 等等

**简化假设**

- 在关系代数中，我们假设：
  1. 关系是*集合*，因此不存在两行完全相同
  2. 每个单元格都有值
- 在 SQL 中我们会去掉这些假设

一个**查询（query）**作用于一组关系，产生一个关系作为结果

- 例如，我们有以下关系：
  - College(<u>cName</u>, state, enrolment)
  - Student(<u>sID</u>, sName, GPA, sizeHS)
  - Apply(<u>sID, cName, major</u>, decision)
- 外键约束
  - Apply[sID] $\subseteq$ Student[sID]
  - Apply[cName] $\subseteq$ College[cName]
- 最简单的查询：直接写关系名
- 可以使用*操作符*来筛选、切片和组合

**select 操作符**用于挑选某些行

- 记为 $\sigma$
- 记法：$\sigma_{\text{cond}}\ Rel$
  - 条件（Condition）是一个布尔表达式
  - 结果是一个与操作数模式相同的关系，但只包含满足条件的元组
- 例如：GPA > 3.7 的学生
  - $\sigma_{\text{GPA}>3.7}\ Student$
- 例如：GPA > 3.7 且 HS < 1000 的学生
  - $\sigma_{\text{GPA}>3.7 \wedge \text{HS}<1000}\ Student$
- 例如：申请了 Stanford CS 专业的申请记录
  - $\sigma_{\text{cName}=\text{'Stanford'} \wedge \text{major}=\text{'cs'}}\ Apply$

**project 操作符**用于挑选某些列

- 记为 $\Pi$
- 记法：$\Pi_{A_1,\ldots,A_n}\ Rel$
  - $\{A_1, \ldots, A_n\}$ 是 Rel 属性的一个子集
  - 结果是一个包含 Rel 中所有元组、但只保留 $\{A_1, \ldots, A_n\}$ 属性的关系
- 例如：所有申请记录的 ID 和结果（decision）
  - $\Pi_{\text{sID,dec}}\ Apply$
- 例如：GPA > 3.7 的学生的 ID 和姓名
  - $\Pi_{\text{sID,sName}}\ (\sigma_{\text{GPA}>3.7}\ Student)$
- 关系代数中会消除重复值
  - 这与 SQL 不同，SQL 基于多重集/包，不会消除重复

**叉积/笛卡尔积操作符**用于合并两个关系

- 记法：$R_1 \times R_2$
  - 结果是一个关系，包含 $R_1$ 中每个元组与 $R_2$ 中每个元组的所有组合的拼接
  - 模式是 $R_1$ 的所有属性后接 $R_2$ 的所有属性
- 例如：Student × Apply
  - 有 8 列：Student.sID, sName, GPA, HS, Apply.sID, cName, major, decision
  - 若 Student 有 S 个元组，Apply 有 A 个元组，则 Student × Apply 有 $S \times A$ 个元组
- 可能会引入无意义的元组，可以用 select 来去除

**自然连接（Natural Join）**

- 记法：$R \bowtie S$
- 结果由以下步骤定义：
  1. 取笛卡尔积
  2. 通过 select 保证两个关系中同名属性的值相等
  3. 通过 project 去除重复的属性
- 性质：
  - 交换律：$R \bowtie S = S \bowtie R$
  - 结合律：$(R \bowtie S) \bowtie T = R \bowtie (S \bowtie T)$
    - 因此可以写成 $R \bowtie S \bowtie T$
- 当两个关系没有共同属性时，结果就是笛卡尔积
- 当没有元组匹配时，结果是空关系
- 当两个关系的属性完全相同时，结果是交集
- 可能出现过度匹配，因为两个属性可能碰巧同名，但我们并不希望它们匹配
- 也可能出现匹配不足，因为两个属性可能名称不同，但我们希望它们匹配

**Theta 连接（Theta Join）**

- 记法：$R \bowtie_\theta S$

$$R \bowtie_\theta S = \sigma_\theta(R \times S)$$

- $\theta$ 是条件

**操作符优先级**

- 表达式可以递归组合
- 括号和优先级规则决定求值顺序

```
高 ↑
σ, Π, ρ
×, ⋈, ⋈θ
∩
∪, −
低 ↓
```

**表达式树（Expression Trees）**

- 一种拆解复杂表达式的方法
- 叶子节点是关系
- 内部节点是操作符
- 与抽象语法树非常相似（CSC111）

**赋值操作符（Assignment Operator）**

- 记法：$R(A_1, \ldots, A_n) := \text{Expression}$
- 允许我们为新关系的所有属性命名
- 即使名称没有变化，为了可读性最好也显式指定名称
- $R$ 必须是一个临时变量，而不是模式中已有的关系
- 例如：DeansListScholars(sID, sName, GPA, sizeHS) := $\sigma_{\text{GPA} \ge 3.5}Student$

**重命名操作（Rename Operation）**

- 记法：$\rho_{R_1}(R_2)$
- 另一种记法：$\rho_{R_1(A_1,\ldots,A_n)}(R_2)$
  - 既可以重命名关系，也可以重命名所有属性
  - $R_1(A_1,\ldots,A_n) := R_2 \equiv R_1 := \rho_{R_1(A_1,\ldots,A_n)}(R_2)$
- 在表达式*内部*需要重命名时很有用

**语法糖（Syntactic Sugar）**

- 有些操作并非必要，因为我们可以通过组合其他操作达到同样效果
- 例如：自然连接、theta 连接

**集合操作**

- 因为关系是集合，所以可以使用集合操作
- 只有当操作数是定义在相同属性上的关系时（即数量、名称、顺序都相同）才能使用
- 集合操作符 $\cap, \cup, -$ 在关系代数中与通常意义相同

**完整性约束（Integrity Constraints）**

- 表达关系 $R_1$、$R_2$ 之间包含依赖关系的记法：$R_1[X] \subseteq R_2[Y]$
- 假设 $R$ 和 $S$ 是关系代数中的表达式，我们可以用以下任一方式写出约束：
  - $R = \varnothing$
  - $R \subseteq S$，等价于 $R - S = \varnothing$

**特定类型查询的策略**

- *Max*（或类似地 *min*）
  - 将元组两两配对，找出那些*不是*最大值的
  - 从全体中减去，得到最大值
- *k 个或更多*
  - 构造所有满足条件的 k 个不同元组的组合（即使用自连接）
- *恰好 k 个*
  - "k 个或更多" 减去 "(k+1) 个或更多"
- *每一个（Every）*
  - 构造出所有应该发生的组合
  - 减去实际发生的，得到那些未曾发生的，即失败的情况
  - 从全体中减去这些失败情况

**关系演算（Relational Calculus）**

- 另一种关系模型的抽象查询语言
- 基于一阶逻辑
- 查询示例：$\{t \mid t \in Movies \wedge t[\text{director}] = \text{Scott}\}$
- 表达能力与关系代数相同，即"关系完备（relationally complete）"

---

## 4 SQL 入门

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

## 5 聚合与分组

**对某一列进行计算**

- SQL 提供了诸如 `sum`、`avg`、`min`、`max`、`count` 等函数，可以在 `SELECT` 子句中应用于某一列
- 称为**聚合（aggregation）**

例如：

```sql
SELECT max(grade) - min(grade)
FROM Took;
```

- 输出一列 `?column?`，只有 1 行

例如：

```sql
SELECT count(*)
FROM Took;
```

- 输出一列 `count`，只有 1 行，表示表中的行数

**重复值**

- 重复值会参与聚合计算
  - 例如，若成绩 85 在表中出现了 10 次，它们都会参与 `avg(grade)`、`count(grade)` 等的计算
- 若要让每个重复值只计算一次，使用 `DISTINCT`

例如：

```sql
SELECT count(DISTINCT dept)
FROM Offering;
```

- `DISTINCT` 不影响 `min` 或 `max`

**多重聚合**

- 各个数量需要是同类的（即一个 max、一个 min、一个 avg 等）才能产生结构完整的表

例如：

```sql
SELECT max(grade), min(grade), count(distinct oid), count(*)
FROM Took;
```

**分组（Grouping）**

- 如果我们在 *SELECT-FROM-WHERE* 表达式后接上 `GROUP BY`，那么在该属性上取值相同的元组会被视为一个*分组（group）*，每个分组会生成*一行*结果
- 需要保证 SQL 能生成结构完整的矩形表

例如：

```sql
SELECT sid, avg(grade)
FROM Took
GROUP BY sid
ORDER BY avg(grade);
```

- 所有 `sid` 相同的行会合并为结果中的一行，得到每个学生的平均成绩

- 可以根据不在 `SELECT` 子句中的内容排序

例如：

```sql
SELECT sid, avg(grade)
FROM Took
GROUP BY sid
ORDER BY count(oid);
```

- 这是有效的，因为每个 `count(oid)` 都对应一个 `sid` 分组
  - 如果我们只用 `oid` 排序则无效，因为每个 `sid` 可能对应多个 `oid`

**按多列分组**

- 如果我们想按两列分组，需要保证每个 (col1, col2) 组合对应的其他属性只有一个值

例如：

```sql
SELECT dept, cnum, count(cnum)
FROM Offering
GROUP BY dept, cnum
ORDER BY dept;
```

- `ORDER BY` 出现在 `GROUP BY` 之后

**聚合的限制**

- 如果查询中使用了聚合，那么 `SELECT` 列表中的每个元素必须是：
  1. 被聚合的，或者
  2. `GROUP BY` 列表中的一个属性
- 在 PostgreSQL 中我们可以稍微打破这条规则，例如：

```sql
SELECT sid, firstname, surname
FROM Student
GROUP BY sid;
```

- 因为每个 `sid` 对应的 `firstname` 和 `surname` 都恰好只有一个值

---

## 6 使用 HAVING 过滤分组

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

## 7 视图

有时我们想用旧事物来定义新事物

- 我们可以通过创建一张新表、并把正确的数据填入其中来实现
  - 但是，如果旧表被更新了，新创建的表就会过时
  - 也可能占用大量空间

**虚拟视图（Virtual Views）**

- 定义一个视图时，我们只是让 SQL 记住这个定义

例如：

```sql
CREATE VIEW DeansHonoursStudent AS
SELECT sid, term, avg(grade)
FROM Took JOIN Offering ON Took.oid = Offering.oid
GROUP BY sid, term
HAVING avg(grade) > 80;
```

- 当我们提到 `DeansHonoursStudent` 时，实际上指的就是它定义中的代码
- 它是*动态*的，会随着变化自动重新计算
  - 重新计算有时可能代价较大
- 之所以称为"虚拟"，是因为除了定义本身之外，它不会被存储
- 可以用来拆分一个大查询
- 提供了另一种查看相同数据的方式
- 我们可以只授予用户访问该视图的权限，而不授予其访问底层基表的权限

**物化视图（Materialized View）**

- 在定义时就被计算并存储
- 与其基表之间的关系会随时被维护
- 维护物化视图与基表之间的关系代价较高

---

## 8 SQL 中的集合与包

SQL 允许一张表存在重复的行

- 前提是这不违反已定义的任何约束（例如主键）
- SQL 中的表是**包（bags）**（也称为**多重集，multiset**）
  - 是集合的一种推广，允许重复
  - 顺序不重要
- 结果中的重复值告诉我们某件事发生了多少次
- 去除重复的代价较高

**我们可以使用 DISTINCT 来去除重复**

例如：

```sql
SELECT DISTINCT oid
FROM Took;
```

- 我们只能要求整体上的行去重，*而不能*按列去重
- 这会把查询变成一个集合

**我们可以在聚合内部使用 DISTINCT**

例如：

```sql
SELECT count(DISTINCT sid), count(DISTINCT oid)
FROM Took;
```

**SQL 中的集合并、交、差**

- 集合并：`UNION`
- 集合交：`INTERSECT`
- 集合差：`EXCEPT`
- 语法：

```sql
(<subquery>) UNION/INTERSECT/EXCEPT (<subquery>)
```

- 集合操作符的操作数必须用*圆括号*括起来
- 集合操作符的操作数必须是*完整的查询*（例如 select ... from ...）
  - *不能*简单地使用两个关系名
- 集合操作符的操作数必须具有*兼容的模式*
  - 具体细节因 DBMS 而异
- 它们按*集合语义*运作（而非包语义）
  - 集合操作之后重复会被消除
  - 操作数会*先被转换为集合*，然后再进行集合操作

**我们可以在集合操作上使用 ALL 来保留重复**

例如：

```sql
(SELECT...FROM...)
UNION ALL
(SELECT...FROM...);
```

- 包并（Bag union）：一个包接续在另一个包之后
  - 例如：$\{1,1,1,3,7,7,8\} \cup \{1,5,7,7,8,8\} = \{1,1,1,1,3,5,7,7,7,8,8,8\}$
- 包交（Bag intersection）：逐个取匹配的元素
  - 例如：$\{1,1,1,3,7,7,8\} \cap \{1,5,7,7,8,8\} = \{1,7,7,8\}$
- 包差（Bag difference）：逐个移除匹配的元素
  - 例如：$\{1,1,1,3,7,7,8\} - \{1,5,7,7,8,8\} = \{1,1,3\}$

---

## 9 SQL 中的空值

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

## 10 连接（Join）

关系代数中的所有连接在 SQL 中都有对应的写法

**对应关系：**

| 表达式 | 含义 |
|---|---|
| `R, S` | $R \times S$ |
| `R cross join S` | $R \times S$ |
| `R natural join S` | $R \bowtie S$ |
| `R join S on Condition` | $R \bowtie_{condition} S$ |

- 使用 `NATURAL JOIN` 连接表 $R$ 和 $S$ 时，$R$ 和 $S$ 必须在所有同名属性上取值一致

例如：

```
R:  A B      S:  B C
    1 2          2 3
    4 5          6 7

R NATURAL JOIN S:
    A B C
    1 2 3
```

- 使用 `JOIN ON` 连接表 $R$ 和 $S$ 时，$R$ 和 $S$ 必须满足连接条件
- 如果某一行不满足连接条件，它会被排除在结果表之外
  - 这样的行被称为**悬挂元组（dangling tuple）**
  - 使用外连接（outer join）可以将它们包含进来

**外连接（Outer Joins）**

- **外连接（outer join）**会用 `NULL` 值填补悬挂元组，从而保留它们
  - 不做填充的连接被称为**内连接（inner joins）**
- **左外连接（left outer join）**只保留左侧表中的悬挂元组

例如：

```
R:  A B      S:  B C
    1 2          2 3
    4 5          6 7

R NATURAL LEFT JOIN S:
    A B C
    1 2 3
    4 5 NULL
```

- **右外连接（right outer join）**只保留右侧表中的悬挂元组

例如：

```
R:  A B      S:  B C
    1 2          2 3
    4 5          6 7

R NATURAL RIGHT JOIN S:
    A B C
    1 2 3
    NULL 6 7
```

- **全外连接（full outer join）**保留所有的悬挂元组

例如：

```
R:  A B      S:  B C
    1 2          2 3
    4 5          6 7

R NATURAL FULL JOIN S:
    A B C
    1 2 3
    4 5 NULL
    NULL 6 7
```

**外连接的 SQL 语法**

- `NATURAL JOIN` 或 `JOIN ON`（theta 连接）都可以选择性地保留悬挂元组
- SQL 语法：

```
笛卡尔积
    A CROSS JOIN B                与 A, B 相同

Theta 连接
    A JOIN B ON C
    A {LEFT|RIGHT|FULL} JOIN B ON C   ✓

自然连接
    A NATURAL JOIN B
    A NATURAL {LEFT|RIGHT|FULL} JOIN B  ✓
```

（✓ 表示在需要时会对元组进行填充）

**自然连接是脆弱的（Natural Join is Brittle）**

- 如果模式发生变化，自然连接的含义可能会随之改变
- 连接条件是从模式中*推断*出来的，而不是由程序员显式写出的
- 使用 `JOIN ON` 比使用 `NATURAL JOIN` 更好

**JOIN USING**

- 与自然连接类似，但我们可以指定要匹配的列

例如：

```sql
SELECT sID, instructor
FROM Student
    JOIN Took Using (sID)
    JOIN Offering USING (oID)
WHERE grade > 50;
```

- 需要用*圆括号*把要匹配的属性列表括起来
- 不如 `NATURAL JOIN` 脆弱，但要求列名相同

---

## 11 子查询

**带集合操作的子查询**

- 使用 `INTERSECT`、`UNION`、`EXCEPT` 连接两个操作数，每个操作数都是一个子查询

例如：

```sql
(SELECT sid FROM Took WHERE grade > 95)
INTERSECT
(SELECT sid FROM took WHERE grade < 50);
```

**FROM 子句中的子查询**

- 查询会把子查询当作一张表
- 两条语法要求：
  1. 子查询必须用圆括号括起来
  2. 我们必须为子查询的结果命名，以便在别处引用它

例如：

```sql
SELECT ...
FROM A, (SELECT ... FROM ...) B
WHERE A.x = B.x;
```

**WHERE 子句中作为值的子查询**

- 如果子查询恰好产生 1 行，我们就可以在 `WHERE` 子句中与它进行比较

例如：

```sql
SELECT sID
FROM Student
WHERE cgpa >
    (SELECT cgpa
    FROM Student
    WHERE sID = 99999);
```

  - 由于 `sID` 是 `Student` 表的主键，子查询不可能产生超过 1 行
- 如果子查询返回空表，那么结果就是空表（因为与空值比较总是假）
- 如果子查询返回超过 1 行，则会抛出错误
- 我们*不能*在关系代数中做类似的比较

**对多个结果进行量化**

- 要将某个单一值与子查询返回的所有值进行比较，我们需要告诉 SQL 使用哪种量词
- `ALL`：全称量化（universal quantification）
  - 当比较对子查询结果中的*每一个*元组都成立时，值为真
- `SOME`：存在量化（existential quantification）
  - 也可以使用 `ANY`
  - 当比较对子查询结果中的*至少一个*元组成立时，值为真
- `IN` 和 `NOT IN`
  - 当值在（不在）子查询生成的行集合中时，值为真
  - `IN` 等价于 `= ANY`
  - `NOT IN` 等价于 `<> ALL`
- `EXISTS` 不涉及任何比较，它只检查子查询是否返回了任何行
  - 当子查询至少有 1 个元组时，值为真
  - "子查询结果中存在一行"
  - 由于一旦找到一行就会短路，因此效率较高

**作用域（Scope）**

- 如果一个名字可能指代多个事物，使用嵌套层级最近的那个
- 如果子查询只引用其内部定义的名字，那么它*只需要被求值一次*，可以在外层查询中反复使用
- 如果子查询引用了任何在其自身之外定义的名字，那么它必须*对外层查询中的每个元组都求值一次*，这被称为**相关子查询（correlated subquery）**

**相关子查询（Correlated Subqueries）**

- 如果子查询引用了外层查询的某个属性，那么它就是**相关的（correlated）**
- 每当我们考虑一行新的数据时，子查询都必须被重新计算

例如：

```sql
SELECT instructor
FROM Offering
WHERE EXISTS (
    SELECT *
    FROM Took
    WHERE Took.oID = Offering.oID AND grade > 98
);
```

- **非相关子查询（uncorrelated subquery）**不引用外层查询
  - 它可以只计算一次，然后对外层查询的每一行重复使用

**小结**

- 子查询可以出现在哪里？
  - 作为 `FROM` 子句中的一个关系
  - 作为 `WHERE` 子句中的一个值
  - 与 `ANY`、`ALL`、`IN`、`NOT IN`、`EXISTS` 一起出现在 `WHERE` 子句中
  - 作为 `UNION`、`INTERSECT`、`EXCEPT` 的操作数

```
x «comparison» ALL («subquery»)
    ∀ y ∈ «subquery results» | x «comparison» y

x «comparison» SOME («subquery»)
    ∃ y ∈ «subquery results» | x «comparison» y

x IN («subquery»)
    与 x = SOME («subquery») 相同   ⎫
                                     ⎬ 只是为了方便
x NOT IN («subquery»)
    与 x <> ALL («subquery») 相同   ⎭

EXISTS («subquery»)
    ∃ y ∈ «subquery results»
```

---

## 12 数据库修改

**插入字面值**

例如：

```sql
CREATE TABLE Ages(name TEXT, age INT);

INSERT INTO AGES VALUES
('Alice', 21), ('Bob', 25), ('Carol', NULL), ('Dave', 0);
```

**插入计算得到的值**

- 例如，我们想找出所有修读过一年级课程的人，并以 19 作为他们的年龄插入表中

```sql
INSERT INTO Ages
(SELECT DISTINCT firstname, 19 AS age
FROM Student JOIN Took USING (sID)
JOIN Offering USING (oID)
WHERE cnum <= 199);
```

- `INSERT INTO` 会把新行添加到表中已有的内容之后

**插入默认值**

- 我们为*要*提供值的属性命名

例如：

```sql
CREATE TABLE Invite(
    name TEXT,
    campus TEXT DEFAULT 'StG',
    email TEXT,
    age INT);

INSERT INTO Invite(name, email)
(SELECT firstname, email
FROM Student
WHERE cgpa > 3.5);
```

- 我们提供了 `name` 和 `email` 的值，让 DBMS 为其他列填入默认值
- 如果没有显式定义默认值，则使用 `NULL`

**删除**

- 指定一行需要满足的条件才能被删除
- 例如：删除不及格的成绩

```sql
DELETE FROM Took
WHERE grade < 50;
```

- 要删除表中的*所有*行，省略 `WHERE` 条件即可，例如：

```sql
DELETE FROM Took;
```

**更新**

- 使用 `UPDATE` 语句来修改某些行的某些属性
- 语法：

```sql
UPDATE <table> SET <list of attribute assignments> WHERE <condition on tuples>;
```

例如：

```sql
UPDATE Student
SET campus = 'UTM'
WHERE sID = 99999;
```

- 可以一次更新多行

---

## 13 SQL 模式（Schema）

**SQL 模式**

- 在关系代数中，*模式（schema）*是关系或一组关系的结构定义
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

## 14 数据定义语言（DDL）

**类型（Types）**

- 创建表时，我们必须定义每一列的类型

**内置类型**

- `CHAR(n)`：定长字符串，长度为 n，不足时用空格填充
- `VARCHAR(n)`：变长字符串，最长 n 个字符
- `TEXT`：变长、无长度限制
  - 不属于 SQL 标准，但 psql 支持
  - 例如：`'Shakespeare''s Sonnets'`
  - 必须用单引号而不是双引号括起来
  - 可以通过转义字符（同样是单引号）在字符串中放入一个字面的单引号
- `INT`, `INTEGER`
  - 例如：37
- `FLOAT`, `REAL`
  - 例如：3.14159, 3.14159e-10
- `BOOLEAN`
  - 例如：TRUE, FALSE
  - 大小写不影响
- `DATE`
  - 例如：2018-03-01, 'September 11, 2001'
- `TIME`
  - 例如：'12:34:56', '12:34:56.789'
- `TIMESTAMP`（日期 + 时间）
- 等等

**用户自定义类型**

- 基于内置类型定义
- 可以定义约束和默认值

例如：

```sql
create domain Grade as int
    default null
    check (value >= 0 and value <= 100);
create domain Campus as varchar(4)
    default 'StG'
    check (value in ('StG', 'UTM', 'UTSC'));
```

**类型约束的语义**

- **语法（Syntax）**：哪些字符串是合法的
- **语义（Semantics）**：一个合法字符串意味着什么/应该做什么
- 类型约束在每次为该类型的列赋值时都会被检查

**默认值的语义**

- 未指定值时，会使用该类型的默认值
- 这与**类型默认值（type default）**不同，后者适用于*任何*表中该类型的*每一*列

**主键约束（Primary Key Constraints）**

- 将一组一个或多个列声明为 `PRIMARY KEY` 意味着：
  - 它们的值在各行之间是唯一的
  - 它们的值永远不会为 null
  - 提示 DBMS 对基于这些列的搜索进行优化
- 每张表必须有 0 个或 1 个主键
  - 在实践中，每张表都应该有 1 个
- 可以在表定义的末尾声明

```sql
create table Person (
    ID integer,
    name varchar(25),
    primary key (ID, name)
);
```

- 括号是必需的
- 对于单列的键，可以随该列一起声明

```sql
create table Person (
    ID integer primary key,
    name varchar(25)
);
```

**唯一性约束（Uniqueness Constraints）**

- 声明一组一个或多个属性为 `UNIQUE` 意味着：
  - 它们的值在各行之间是唯一的
  - 但它们的值*可以*为 null
- 如果不希望它们为 null，需要单独声明
- 可以声明多组属性为 `UNIQUE`
- 可以在表定义的末尾声明

```sql
create table Person (
    ID integer,
    name varchar(25),
    unique (ID, name)
);
```

- 括号是必需的
- 对于单列的键，可以随该列一起声明

```sql
create table Person (
    ID integer primary key,
    name varchar(25) unique
);
```

- 对于唯一性约束，两个 null 不算相等，所以允许插入两个 null

**外键约束（Foreign Key Constraints）**

- 例如：`foreign key (sID) references Student`
  - 不指定引用哪个属性，默认引用主键
- 这张表中的 `sID` 列是一个外键，引用 `Student` 的主键
- 该表中 `sID` 的每个值都必须真实存在于 `Student` 表中
- 要求：在"所属"表中必须被声明为主键或 unique
- 可以作为单独的表元素声明，也可以随列一起声明（仅当只涉及单个列时可行）
- 只要目标列是唯一的，就可以引用非主键的列，此时需要显式指定该列

```sql
create table People (
    SIN integer primary key,
    name text,
    OHIP text unique
);
create table Volunteeers (
    email text primary key,
    OHIPnum text references People(OHIP)
)
```

**强制执行外键约束**

- DBMS 必须确保：
  - 被引用的列是 `PRIMARY KEY` 或 `UNIQUE`
    - 在模式声明时检查一次
  - 值确实存在
    - 在每次插入、删除、更新时都会检查
- 我们可以定义 DBMS 在违反约束时应该采取的行动，这被称为指定一个**反应策略（reaction policy）**

**"Check" 约束**

- 可以在用户自定义域上使用 `check` 子句
- 可以定义 check 约束：
  - 针对某一列
  - 针对表的行
  - 跨多张表

**基于列的 "Check" 约束**

- 定义在单个列上，约束它的值（在每一行中）

例如：

```sql
create table Application (
    sID integer,
    previous integer check (previous >= 0)
);
```

- `previous` 列只能取正值
- 约束该列在每一行中的值
- 条件可以是任何能出现在 `WHERE` 子句中的内容
- 有些 DBMS 允许在基于列的约束中使用子查询，但 psql 不允许
- 只有在向表中插入或更新一行时才会被检查
- 如果其他地方的变化违反了约束，DBMS 不会察觉
  - 例如，在子查询中，当另一张表中的内容发生变化时

**"Not Null" 约束**

- 一种特定的基于列的约束
- 声明表的某一列不能为空

```sql
create table Application (
    sID integer,
    previous integer not null
);
```

- 在实践中，很多列都应该被声明为 not null

**基于行的 "Check" 约束**

- 作为表模式的一个单独元素定义
- 可以引用表的任意列
- 条件可以是任何能出现在 `WHERE` 子句中的内容，也可以包含子查询

例如：

```sql
create table Student (
    sID integer,
    age integer,
    year integer,
    college varchar(4),
    check (year = age - 18),
    check (college in (select name from Colleges))
);
```

- 只有在向该表插入或更新一行时才会被检查
- 如果其他地方的变化违反了约束，DBMS 不会察觉
- 只有当 check 约束求值为假时才会失败
  - 由 null 产生的未知真值会通过检查
  - 可以用 not null 约束来防止这种情况

**命名约束**

- 可以为约束命名，从而获得更有帮助的错误信息
- 可以用于上述任何一种约束
- 在 `check (<condition>)` 之前添加 `constraint <name>`，例如：

```sql
create domain Campus as varchar(4)
    not null
    constraint validCampus check (value in ('StG', 'UTM', 'UTSC'));
```

**跨表约束：断言（Assertions）**

- 断言（Assertions）是模式顶层的元素，因此可以表达跨表约束

```sql
create assertion <name> check (<predicate>);
```

- 断言开销较大，因为每次数据库更新时都要检查，而每次检查都可能代价高昂
- 测试和维护也比较困难
- psql（以及大多数其他 DBMS）都不支持

**触发器（Triggers）**

- 我们可以指定想要响应的一类数据库事件，例如：

```sql
after delete on Courses
```

或

```sql
before update of grade on Took
```

- 我们指定响应结果，例如：

```sql
insert into Winners values (sID)
```

- 响应逻辑被封装在一个函数中
- 例如（函数）：

```sql
create function RecordWinner() returns trigger as
$$
BEGIN
    IF NEW.grade >= 85 THEN
        INSERT INTO Winners VALUES (NEW.sid);
    END IF;
    RETURN NEW;
END;
$$
LANGUAGE plpgsql;
```

- `BEGIN` 和 `END` 用于指定函数体
- 双美元符号是包裹函数体的"引号"

- 例如（触发器）：

```sql
create trigger TookUpdate
before insert on Took
for each row
execute procedure RecordWinner();
```

**反应策略（Reaction Policies）**

- `CASCADE`：将变化传播到引用方所在的表
- `SET NULL`：将引用列设置为 null
- `RESTRICT`：阻止该删除/更新操作
- 如果我们什么都不说，默认是禁止在被引用表中发生该变化（即 `RESTRICT`）

**反应策略的不对称性**

- 假设表 $R$ 引用表 $S$
- 我们可以定义从 $S$ 向 $R$ 反向传播变化的"修正"
- 我们*不能*定义从 $R$ 向 $S$ 正向传播变化的"修正"
- 语法：

```sql
foreign key (sID) references Student on delete cascade
```

- 可以使用 `on delete`（即删除操作产生悬挂引用时）或 `on update`（即更新操作产生悬挂引用时），或两者都用
  - 例如：`on delete restrict on update cascade`

**删除的语义**

- 如果我们要删除若干行，其中删除某一行会违反外键约束，DBMS 会执行以下操作之一：
  1. 遇到错误就停止，但保留之前已经删除的行
  2. 回滚之前的删除，什么都不做
  3. 执行除了违反约束的那些以外的所有删除
- 如果删除某一行会影响之后遇到的另一行的结果：
  - 删除操作首先标记所有满足 `WHERE` 条件的行
  - 然后再删除被标记的行

**更新模式（Updating the Schema）**

- 可以修改域或表

```sql
alter table Course
    add column numSections integer;
alter table Course
    drop column breadth;
```

- `DROP` 关键字用于移除域、表，或整个模式
- 如果我们要删除一张被其他表引用的表，那么必须指定 `CASCADE`，它会移除所有引用行

**关于 DDL 的更多内容**

- 可以定义*索引（indices）*，让搜索更快
- 可以定义*权限（privileges）*，指定谁可以对数据库的哪些部分做什么操作

---

## 15 嵌入式 SQL

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

## 16 函数依赖理论

我们希望得到一个处于*范式（normal form）*的模式，它能保证良好的性质

- **规范化（Normalization）**：将模式转换为范式的过程

**示例：一张设计糟糕的表**

| part | manufacturer | manAddress | seller | sellerAddress | price |
|---|---|---|---|---|---|
| 1983 | Hammers 'R Us | 99 Pinecrest | ABC | 1229 Bloor W | 5.59 |
| 8624 | Lee Valley | 102 Vaughn | ABC | 1229 Bloor W | 23.99 |
| 9141 | Hammers 'R Us | 99 Pinecrest | ABC | 1229 Bloor W | 12.50 |
| 1983 | Hammers 'R Us | 99 Pinecrest | Walmart | 5289 St Clair W | 4.99 |

- 有些数据是冗余的
  - 每个零件（part）只有 1 个厂商
  - 每个厂商只有 1 个地址
  - 每个卖家（seller）只有 1 个地址
- 冗余数据可能导致异常

**异常（Anomalies）**

- **更新异常（Update anomaly）**：例如，如果 Hammers 'R Us 搬家了，而我们只更新了 1 行，数据就会不一致
- **删除异常（Deletion anomaly）**：例如，如果 ABC 不再出售零件 8624，而 Lee Valley 只生产那一个零件，我们就会失去它的地址信息

**函数依赖的定义**

- 假设 $R$ 是一个关系，$X$ 和 $Y$ 是 $R$ 属性的子集
- $X \rightarrow Y$ 断言：如果两个元组在集合 $X$ 的所有属性上一致，那么它们也必须在集合 $Y$ 的所有属性上一致
- 我们说"$X \rightarrow Y$ 在 $R$ 中成立"，或"$X$ 函数决定（functionally determines）$Y$"
- $X \rightarrow Y$ 是一种*依赖（dependency）*，因为 $Y$ 的值依赖于 $X$ 的值
- $X \rightarrow Y$ 是*函数式的（functional）*，因为存在一个数学函数，它接受一个 $X$ 的值，给出唯一确定的 $Y$ 值
- 一个函数依赖（FD）是关于关系的*每一个*实例的断言
  - 判断某个 FD 是否成立，需要领域知识

**形式化定义**

$$A \rightarrow B: \forall \text{tuples } t_1, t_2, (t_1[A] = t_2[A]) \implies (t_1[B] = t_2[B])$$

等价于：不存在元组 $t_1, t_2$ 使得 $(t_1[A] = t_2[A]) \wedge (t_1[B] \ne t_2[B])$

**推广到多个属性**

$$A_1 A_2 \cdots A_m \rightarrow B_1 B_2 \cdots B_n: \forall \text{tuples } t_1, t_2, \bigwedge_{i=1}^{m} t_1[A_i] = t_2[A_i] \implies \bigwedge_{j=1}^{n} t_1[B_j] = t_2[B_j]$$

等价于：不存在元组 $t_1, t_2$ 使得 $\bigwedge_{i=1}^{m} t_1[A_i] = t_2[A_i] \wedge \neg \bigwedge_{j=1}^{n} t_1[B_j] = t_2[B_j]$

**示例中的 FD**

- 每个零件只有 1 个厂商：`part → manufacturer`
- 每个厂商只有 1 个地址：`manufacturer → manAddress`
- 每个卖家只有 1 个地址：`seller → sellerAddress`

**等价的 FD 集合**

- 当我们写出一组 FD 时，意味着*所有*这些 FD 都成立
- 我们常常可以用等价的方式重写 FD 集合
- 当我们说 $S_1$ 与 $S_2$ 等价时，意味着 $S_1$ 在某个关系中成立当且仅当 $S_2$ 成立

**FD 的拆分规则（Splitting Rules）**

- 我们*可以*把 FD 右侧（RHS）拆分成多个 FD

$$AB \rightarrow CD \equiv AB \rightarrow C \wedge AB \rightarrow D$$

- 我们*不能*拆分 FD 的左侧（LHS）

**FD 与键**

- 假设 $K$ 是关系 $R$ 的一组属性
- $K$ 是 $R$ 的一个*超键*，当且仅当 $K$ 函数决定 $R$ 的全部属性
- 回忆一下，超键是这样一组属性：不存在两行在这些属性上取值完全相同
- FD 是键概念的一种推广
  - 超键：$X \rightarrow R$，其中 $R$ 代表每一个属性
  - 函数依赖：$X \rightarrow Y$，其中 $Y$ 不一定是全部属性

**推断 FD（Inferring FDs）**

- 给定一组 FD，我们通常可以推断出更多的 FD
- 大任务：给定一组 FD，推断出所有其他必然成立的 FD
- 简化任务：给定一组 FD，检查*某一个给定的* FD 是否也必然成立
- 我们并不是在生成新的 FD，而是在测试某个特定的 FD
- 方法一：使用第一性原理证明某个 FD 成立
  - 通过回溯到已知成立的 FD，以及函数依赖的定义来证明
- 方法二：使用*闭包测试（closure test）*证明某个 FD 成立
  - 假设我们知道 LHS 属性的值，推算出由此能确定的所有其他内容
  - 如果其中包含 RHS 属性（即我们想要检验的那些），那么就说明 LHS → RHS 成立

```
attribute_closure(Y, S):
    # Y 是一组属性，S 是一组 FD
    # 返回 Y 在 S 下的闭包
    initialize Y+ to Y
    while Y+ changes do
        if there is an FD LHS -> RHS in S such that LHS is in Y+ then
            add RHS to Y+
```

- 如果 LHS 在 $Y^+$ 中，并且 LHS → RHS 成立，我们就可以把 RHS 加入 $Y^+$

```
fd_follows(S, LHS -> RHS):
    Y+ = attribute_closure(LHS, S)
    return RHS is in Y+
```

**投影 FD（Projecting FDs）**

- 在对模式进行规范化时，我们需要*分解（decompose）*关系
- 我们想知道分解后得到的较小关系中，哪些 FD 成立
- 我们可以把我们的 FD *投影*到新关系的属性上

```
project(S, L):
    # S 是一组 FD，L 是一组属性
    # 返回 S 在 L 上的投影，即所有从 S 推出的、且只涉及 L 中属性的 FD
    initialize T to {}
    for each subset X of L do
        compute X+ # close X and see what we get
        for every attribute A in X+ do
            if A in L then # X -> A only relevant if A in L
                add X -> A to T
    return T
```

- 如果 $A = X$，则不需要添加 $X \rightarrow A$，因为它是一个平凡 FD（trivial FD）
- 以下 $X$ 的子集不会带来任何有用信息，因此不需要计算它们的闭包：
  - 空集
  - 全体属性集合
  - 如果我们发现 $X^+ = $ 全部属性，那么可以忽略 $X$ 的任何超集，因为它只能给出"更弱"的 FD（即左侧更大的相同 FD）
- 投影的计算代价很高（子集数量呈指数增长）

**最小基（Minimal Basis）**

- 给定一组 FD $S$，我们想要找出一个*最小基（minimal basis）*，它是一个与 $S$ 等价的 FD 集合，并且满足：
  - 没有冗余的 FD
  - FD 左侧没有不必要的属性

```
minimal_basis(S):
    # S 是一组 FD
    # 返回 S 的一个最小基
    split the RHS of each FD
    for each FD X -> Y where |X| >= 2 do
        if we can remove an attribute from X and get an FD that follows from S then
            do so # it's a stronger FD
    for each FD f do
        if S - {f} implies f then
            remove f from S
```

- 通常存在*多个*最小基（取决于我们考虑各种化简的顺序）
- 一旦我们识别出某个冗余的 FD，在后续计算闭包时就不应再使用它
  - 某个 FD 之所以是冗余的，很可能是因为另一个 FD 的存在
- 在计算闭包以判断某个 FD $X \rightarrow Y$ 的左侧是否可以化简时，可以继续使用该 FD
- 这两个 for 循环的顺序必须如此
  - 否则这两个循环需要反复执行，直到没有变化为止

---

## 17 数据库设计

**函数依赖的应用场景**

- FD 可以告诉我们存在冗余，因此该设计是不好的

**分解（Decomposition）**

- 为了改进设计糟糕的模式 $R(A_1, \ldots, A_n)$，我们可以将其分解为两个更小的关系

$$R_1(B_1, \ldots, B_j) = \Pi_{B_1,\ldots,B_j}R$$
$$R_2(C_1, \ldots, C_k) = \Pi_{C_1,\ldots,C_k}R$$

满足：

$$\{B_1, \ldots, B_j\} \cup \{C_1, \ldots, C_k\} = \{A_1, \ldots, A_n\}$$
$$R_1 \bowtie R_2 = R$$

- 不遗漏任何属性；也不增加任何属性
- 我们可以重新构造出 $R$
- 一个关系可能存在许多种可能的分解方式
- **Boyce-Codd 范式（BCNF）**保证新的模式不会出现异常

**Boyce-Codd 范式**

- 一个关系 $R$ 处于 **BCNF**，当且仅当对 $R$ 中每一个非平凡的 FD $X \rightarrow Y$，$X$ 都是超键
- *非平凡（Nontrivial）*意味着 $Y$ 不包含在 $X$ 中
- 回忆一下，超键不必是最小的
- BCNF 要求：只有那些能函数决定*一切*的东西，才能函数决定*任何东西*

```
bcnf_decomp(R, F):
    # R 是一个关系，F 是一组 FD
    # 返回 R 在这些 FD 下的 BCNF 分解
    if an FD X -> Y in F violates BCNF then
        compute X+
        replace R by two relations with schemas:
            R1 = X+
            R2 = R - (X+ - X)
        project the FDs F onto R1 and R2
        Recursively decompose R1 and R2 into BCNF
```

```
1) 从违反约束的 FD 的左侧（LHS）开始
2) 对 LHS 取闭包，得到一个新关系 X+
        X 位于两个新关系中的共同部分，用于将它们联系起来
3) 除新内容之外的其余部分构成另一个新关系
        R - X+ + X
```

- 如果有多个 FD 违反 BCNF，我们可以基于其中任意一个进行分解
  - 存在多种可能的分解方式
- 我们新创建的关系可能仍不处于 BCNF，因此必须递归处理
  - 我们只保留处于"叶子节点"的关系
- 我们不需要知道任何键，因为只有超键才重要
- 我们不需要知道*所有*超键
  - 只需要检查每个 FD 的左侧是否是超键
  - 可以使用闭包测试
- 在将 FD 投影到新关系上时，检查每个新 FD 是否会导致新关系违反 BCNF
  - 如果会，就放弃这次投影，因为我们反正要丢弃这个关系（并进一步分解）

**分解的性质**

- 我们对分解的期望：
  1. **无异常（No anomalies）**
  2. **无损连接（Lossless join）**：应该能够将原关系投影到分解后的模式上，然后通过连接重新构造出原始的元组
  3. **依赖保持（Dependency preservation）**：所有原始的 FD 都应该被满足
- BCNF 分解满足前两条性质
  - 单靠 BCNF 性质本身*不*保证无损连接
  - 如果我们使用 BCNF 分解算法，那么就能保证无损连接
  - 由于 BCNF 算法会把关系分解得过细，有可能构造出一个满足最终模式中所有 FD、但违反某个原始 FD 的实例

**第三范式（3rd Normal Form）**

- **第三范式（3NF）**将 BCNF 的条件放宽
- 一个属性是**主属性（prime）**，如果它属于任意一个键
- $X \rightarrow A$ 违反 3NF，当且仅当 $X$ 不是超键，且 $A$ 不是主属性

```
3NF_synthesis(F, L):
    # F 是一组 FD，L 是一组属性
    # 综合并返回一个处于第三范式的模式
    construct a minimal basis M for F
    for each FD X -> Y in M do
        define a new relation with schema X union Y
    if no relation is a superkey for L then
        add a relation whose schema is some key
```

- 与 BCNF 的比较
  - BCNF 分解不会停止分解，直到在所有关系中，只要 $X \rightarrow A$，$X$ 就是超键
  - 3NF 允许生成这样的关系：$X \rightarrow A$，但 $X$ *不是*超键，只要 $A$ 至少是主属性
- 3NF 保证无损连接和依赖保持，但不保证无异常
  - 3NF 允许左侧非超键的 FD 存在，这会带来冗余，从而产生异常
- 分解得过细 $\implies$ 无法保留所有 FD
- 分解得不够 $\implies$ 可能存在冗余
- 如果一个模式处于 BCNF 或 3NF 中的任意一种，我们就认为它是"好的"
- 综合（Synthesis）与分解（Decomposition）
  - 综合：我们从零开始构建模式中的关系
  - 分解：我们从一个糟糕的关系模式出发，将其拆解

**测试无损连接**

- 我们将 $R$ 投影为 $R_1, \ldots, R_k$，并尝试通过连接来恢复 $R$
- 我们会得到 $R$ 的全部内容，因为 $R$ 中的任意元组都可以从它的投影片段中恢复
- 然而我们可能会得到一个在 $R$ 中并不存在的元组，这正是我们需要检查的部分
- 如果模式是通过 BCNF 分解或 3NF 综合生成的，我们就不需要测试无损连接
- 然而，仅仅满足 BCNF/3NF 并不保证无损连接

**追逐测试（Chase Test）**

- 假设元组 $t$ 出现在连接结果中
- 那么 $t$ 是 $R$ 中若干元组投影的连接结果，分解中的每个 $R_i$ 对应一个
- 从假设 $t = abc\ldots$ 开始
- 对每个 $i$，$R$ 中存在一个元组 $s_i$，它在 $R_i$ 的属性上取值为 $a, b, c, \ldots$，而 $s_i$ 在其他属性上可以取任意值
- 算法：
  1. 如果两行在某个 FD 的左侧一致，那么使它们的右侧也一致
  2. 只要可能，就用对应的无下标符号替换带下标的符号
  3. 如果我们最终得到了一行完全没有下标的记录，那么就说明投影再连接中的任意元组都存在于原关系中（即该连接是无损的）
  4. 否则，最终的表格（tableau）就是一个反例（即该连接是有损的），因为带下标的符号可能代表不同的值

---

## 18 实体/联系模型（E/R Model）

**实体/联系（ER）模型**

- **建模（Modelling）**：将现实世界中的实体与联系映射为数据库中的概念
- 基本概念：
  - *实体（Entities）*
  - 它们之间的*联系（Relationships）*
  - 描述实体和联系的*属性（Attributes）*

**定义模式**

- E/R 允许我们指定结构可以/必须是什么样子
- 我们从具体的实体和联系推广到它们所属的集合

| 实例（Instance） | 模式（Schema） |
|---|---|
| 实体（带属性） | 实体集（带属性） |
| 联系（带属性） | 联系集（带属性） |

**实体集（Entity Sets）**

- **实体集（entity set）**表示一类具有共同属性、且具有独立存在性的对象（例如：City、Department、Employee、Sale）
- **实体（entity）**是实体集的一个实例（例如：Toronto 是一个 City）

```
[Employee]   [Department]

[City]       [Sale]
```

**联系集（Relationship Sets）**

- **联系集（relationship set）**是 2 个或更多实体集之间的关联（例如：Exam 是 Student 与 Course 两个实体集之间的联系集）
- **联系（relationship）**是一个 n 元联系集的一个实例（例如：<Diane, CSC343> 这一对是联系集 Exam 的一个实例）

```
Employee --<WorkPlace>-- City
Employee --<Residence>-- City
Student  --<Exam>-------- Course
```

**递归联系（Recursive Relationships）**

- 递归联系将一个实体集与自身相关联
- 这种联系可能是不对称的
  - 如果是这样，我们需要标出该实体在联系中扮演的两个*角色（roles）*

```
[Employee] --<Colleague>--（自环）

Predecessor -[Sovereign]- Successor
              <Succession>
```

**三元联系（Ternary Relationships）**

```
Supplier   [S1, S2, S3, S4]
Product    [P1, P2, P3, P4, P5]
Department [D1, D2, D3, D4]

[Supplier] --<Supply>-- [Product]
                 |
            [Department]
```

**属性（Attributes）**

- 描述实体或联系的基本性质（例如：Surname、Salary 是 Employee 的属性）
- 可以是*单值的（single-valued）*，也可以是*多值的（multi-valued）*

```
        Mark  Date
          \   /
Student --<Exam>-- Course
Number,             Name,
EnrolmentDate       Year

           WorkPlace
Employee --<          >-- City
           BirthPlace     Name,
Surname,                  NumberOfInhabitants
Salary,
Age             DateOfBirth
```

**复合属性（Composite Attributes）**

- **复合属性（Composite attributes）**是同一个实体或联系上、含义或用途紧密相关的一组属性

```
Person -- Surname
        -- Age
        -- Sex
        -- (Address) -- Street
                      -- HouseNumber
                      -- PostCode
```

**键（Keys）**

- 记法：实心圆点
- 若为多属性键，用一条线和一个"结（knob）"连接

```
Automobile ●-- Registration
           -- Model
           -- Colour

Person ●-- DateOfBirth
       -- Surname
       -- FirstName
       -- Address
```

**基数（Cardinalities）**

- 每个实体集以一个*最小*和*最大*基数参与一个联系集
- 基数*约束*了实体实例如何参与联系实例
- 记法：为每个实体集标注一对 (min, max) 值

```
Employee (1,5)--<Assignment>--(0,50) Task
```

- $0 \le \text{min} \le \text{max} \in \mathbb{Z}$

- 最小基数 min
  - 若为 0，则该实体参与该联系是*可选的（optional）*
  - 若为 1，则该实体参与该联系是*强制的（mandatory）*
  - 也可能是其他值
- 最大基数 max
  - 若为 1，则该实体的每个实例*至多*与联系的一个实例关联
  - 若大于 1，则该实体的每个实例可以与联系的*多个*实例关联
  - $N$ 表示没有上限
  - 也可能是其他值

```
Order  (0,1)--<Sale>--(1,1)  Invoice
Person (1,1)--<Residence>--(0,N)  City
Tourist(1,N)--<Reservation>--(0,N)  Voyage
```

**联系的多重性（Multiplicity of Relationships）**

- 假设实体集 $E_1$ 和 $E_2$ 分别以基数 $(n_1, N_1)$ 和 $(n_2, N_2)$ 参与联系 $R$
- 那么 $R$ 的**多重性（multiplicity）**是 $N_1$-to-$N_2$（或 $N_2$-to-$N_1$）
- 1-to-1 意味着两侧都不允许"分叉"
- 1-to-N 意味着允许一侧分叉
  - 必须查看 ER 图才能知道是哪一侧
- N-to-N 意味着两侧都允许分叉
- 这种表示法比 (min, max) 记法传达的信息更少，因为它只说明了 max 值

**属性的基数**

- 描述一个属性可以拥有的值的最小/最大数量
- 当属性的基数为 (1,1) 时，这是一个**单值属性（single-valued attribute）**，可以在图中省略不标
- 属性的值可以为 null，也可以拥有多个值（即**多值属性，multi-valued attribute**）

```
                Surname
                 (0,1)
                  |
                License Number
                  |
[Person]-----(0,N)----○ CarRegistration#
```

- 多值属性通常可以用额外的实体来建模，例如：

```
Surname
 (0,1)
  |
License Number
  |
[Person] --(0,N)--<Owns>--(1,1)-- [Car] --○ CarRegistration#
```

**弱实体集（Weak Entity Sets）**

- **内部键（internal key）**是由实体自身的一个或多个属性组成的键
- **弱实体集（weak entity set）**是在其属性中不存在键的实体集
  - 相关实体的键会被引入进来以协助识别（即成为**外键，foreign keys**）

```
                    Registration
                    Year          ●Name
                    Surname        |
[Student]--(1,1)--<Enrolment>--(1,N)--[University]
                                       ○City
                                       ○Address

（Student 是弱实体集，其外键为多属性键：Registration, Year）
```

**联系集的键**

- 联系集的键由它所关联的各实体集的键组成
- 例如：`Made By` 联系集的键是 `Part Number` 和 `Name`

```
Name                             Address
 |                                 |
[Part]●--(1,1)--<Made By>--(1,N)--[Manufacturer]
Part Number                      ●Name
```

**对键的要求**

- 键中的每个*属性*都必须具有 (1,1) 基数
- 弱实体集的外键必须通过该实体集以 (1,1) 基数参与的联系传入
- 外键可以涉及一个自身也拥有外键的实体，只要不产生循环

- 每个实体集都必须至少拥有一个（内部或外部的）键

```
Code                     (0,1)  Management  (1,1)
Surname                    ┌───────────────────┐
Salary  [Employee]─────────┤                   ├───(1,N)  Phone
Age         │  (0,1)Membership(1,N)│      [Department]●Name
            │                                          │(1,1)
       (0,N)Participation                        (1,N)Composition
            │StartDate                                 │
        [Project]                                  [Branch]●City
Name●  Budget                                          Address
       (0,1)ReleaseDate                          (Number, Street, PostCode)
```

**ER 建模面临的挑战**

- 设计选择：某个概念应该建模为实体、属性，还是联系？
- 局限性：某些数据语义无法被捕捉
- **简约性（Parsimony）**：模型应该做到必要的复杂程度，但不多于此；只表示相关的事物

---

## 19 从 E/R 模型到数据库模式

**步骤**

1. **重构（Restructure）** ER 模式，根据某些标准对其加以改进
2. 将模式**转译（Translate）**为关系模型
3. 为模式**补充缺失的约束（Add missing constraints）**

**重构**

- 输入：E/R 模式
- 输出：重构后的 E/R 模式
- 该步骤包括：
  - 冗余分析
  - 在实体集与属性之间做出选择
  - 限制弱实体集的使用
  - 确定键
  - 为基数大于 1 的属性创建实体集以替代它们

**冗余分析**

- 在一个关系中，如果一个实体集拥有某个属性，而另一个实体集也拥有该属性，那么就存在冗余

**实体集 vs. 属性**

- 我们更偏好使用属性
- 一个实体集应该至少满足以下条件之一：
  1. 它不仅仅是某事物的名称；它至少有一个非键属性
  2. 它是某个多对一或多对多联系中的"多"的一方
- 自身独立存在的"事物"：实体集
- 关于其他事物的"细节"：属性

**限制弱实体集的使用**

- 只有在不存在能创建唯一 ID 的全局权威机构时，才使用弱实体集（例如：全世界统一的学生编号）
- 通常更好的做法是创建唯一 ID

**确定键**

- 确保每个实体集都有一个键
- 带有空值的属性不能作为主键的一部分
- 内部键优于外部键
- 被许多操作用来访问某实体实例的键，优于其他的键
- 属性*数量少（一个/几个）*的键更受偏好
- 整数类型的键更受偏好
- 应避免多属性键和字符串键
  - 它们比 4 字节整数占用更多存储空间
  - 它们会破坏封装性（即可能泄露个人信息）
  - 它们是"脆弱的（brittle）"
    - 姓名或电话号码可能会改变
    - 有些人可能没有电话号码
    - 两部电影可能有相同的标题和年份

**为基数大于 1 的属性创建实体集以替代它们**

- 关系模型不允许多值属性，因此我们需要将它们转换为实体集

例如：

```
Name    (1,N)
 |       |
City──[Company]──Phone

Name              (1,N)      (1,1)
 |                          
City──[Company]──<Possesses>──[Phone]──○ number
```

**将 E/R 模型转译为数据库模式**

- 输入：E/R 模式
- 输出：关系模式
- 从 E/R 模式出发，构建一个等价的关系模式（即一个能够表示相同信息的模式）
- 好的转译应该：
  1. 不允许冗余
  2. 不产生不必要的空值
- 思路：
  - 每个实体集变成一个关系
    - 它的属性就是该实体集的属性
  - 每个联系变成一个关系
    - 它的属性是它所连接的各实体集的键（作为键），以及该联系本身的属性

**多对多联系**

```
Surname                        Name
Salary   [Employee]──(0,N)──<Participation>──(0,N)──[Project]   Budget
Number●                      StartDate                          ●Code
```

```
Employee(Number, Surname, Salary)
Project(Code, Name, Budget)
Participation(Number, Code, StartDate)
    Participation[Number] ⊆ Employee[Number]
    Participation[Code] ⊆ Project[Code]
```

**一对多联系**

- 当"一"的一方参与是强制性的（mandatory）时
  - 可以将联系集"折叠（collapse）"进"一"的一方所在的实体集
  - 只有当该实体集在联系集中的参与是 (min=1, max=1) 时，折叠才是可行的

**一对一联系**

- 当双方参与都是强制性的时
  - 可以将联系集折叠进任意一个实体集
- 当其中一方参与是可选的时
  - 可以将联系集折叠进参与为强制性的那个实体集

**最后的考虑事项**

- 在转译期间或之后，我们应该：
  - 表达外键约束
  - 为引用属性考虑更好的命名
  - 表达 "max 1" 约束
  - 表达 "min 1" 约束
- 完成这一过程后，我们将不再拥有显式的函数依赖

**冗余有时是可取的**

- 缺点
  - 需要更多存储空间
  - 需要额外的操作来保持数据一致
- 优点
  - 速度更快：获取信息所需的访问次数更少

- 我们可以考虑是否允许一定的冗余（即**反规范化，denormalization**）
  - 冗余信息带来的操作加速
  - 这些操作的相对频率
  - 冗余信息所需的存储空间

---

## 20 索引

**加快搜索速度**

- 平衡二叉搜索树很适合用于搜索
- 数据库中的数据通常无法全部放入内存，跟随文件指针的速度非常慢
- 为了尽量减少指针跳转，我们可以使用很大的分支因子，从而得到一棵矮的树
- DBMS 通常使用 **B 树（B-trees）**来实现这些目标

**B 树**

- 一种分支因子 $M > 2$ 的树，遵循以下平衡规则：
  - 所有叶子节点都位于相同的深度
  - 每个节点至少有 $\lceil M/2 \rceil$ 个子节点，根节点除外（根节点可以不满足）
  - 根节点至少有 2 个子节点（除非它是唯一的节点）
- 对于阶为 $M$ 的 B 树，若要容纳 $n$ 个值，其高度 $\le \log_{\lceil M/2 \rceil} n$

**存储非整数值**

- B 树只包含键
- 我们沿着这些键导航，从而在叶子节点找到所需的值
- 与该值一起存储的还有一个指向实际数据的指针
- 我们把这样的 B 树称为数据上的一个**索引（index）**

**SQL 中的索引**

```sql
create index off_dept on offering(dept);
```
