# 14 数据定义语言（DDL）

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

- 假设表 `R` 引用表 `S`
- 我们可以定义从 `S` 向 `R` 反向传播变化的"修正"
- 我们*不能*定义从 `R` 向 `S` 正向传播变化的"修正"
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
