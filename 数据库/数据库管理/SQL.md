# SQL

所有者: H34V3N

在此开始，我们学习SQL语言

[工具的安装和基本使用](SQL/工具的安装和基本使用%2029705a2041d280c68639f5ca6d24a41d.md)

# 概述

SQL语言包含下面几个部分：

- 数据定义语言（DDL）：提供定义关系模式，删除关系，修改关系的命令
- 数据操纵语言（DML）：提供从数据库查询信息，插入，删除，修改元组的能力
- 完整性（Integrity）：DDL包含用于完整性约束的命令，数据库中存储的数据必须满足这些约束，不满足则不会应用
- 视图定义（View definition）：定义视图的指令
- 事务控制（Transaction control）：SQL提供指定事务起止点的命令
- 嵌入式SQL和动态SQL
- 授权（Authorization）：为关系和视图指定访问权限

# SQL数据定义

数据库的关系集合通过DDL指定，SQL 的DDL不但允许我们指定一组关系，还能为每个关系指定下面的信息

- 每个关系的模式（schema）。
- 与每个属性关联的取值类型。
- 完整性约束。
- 每个关系需要维护的索引集合。
- 每个关系的安全与授权信息。
- 每个关系在磁盘上的物理存储结构。

## SQL数据的基本类型

- **char(n)**：定长字符串，长度为用户指定的 n。可用全称 **character**。
- **varchar(n)**：变长字符串，最大长度为 n。全称 **character varying** 与之等价。
- **int**：整数（机器相关的有限子集）。全称 **integer** 等价。
- **smallint**：小整数（整数类型的机器相关子集）。
- **numeric(p, d)**：定点数，精度由用户指定。数值总共 p 位（含符号），其中 d 位在小数点右侧。例如 **numeric(3,1)** 可精确存储 44.5，但不能精确存储 444.5 或 0.32。
- **real, double precision**：浮点与双精度浮点（精度与机器相关）。
- **float(n)**：至少 n 位精度的浮点数。
- null：空值，表示缺失或未知的值，但是这种值未必在每个关系下都会使用

## cerate table

我们可以使用create table这个操作来创建一个SQl关系，其语言格式为：

create table {关系的名称}

(

属性名称1 属性基本类型（其他要求）,

属性名称2 属性基本类型（其他要求）,

属性名称3 属性基本类型（其他要求）

)； 分号用于区分代码，如果只是一个操作则可写可不写

现在我们假设设置一个有关大学学生管理的关系，包含学生的学号，姓名，性别，入学年份等信息

```sql
create table students
(
studentID char(11),
studentName varchar(20),
registerYear int,
sex varchar(10)
constraint sexCheck check (sex in('male','female'))
)

/*
反馈：
CREATE TABLE

耗时33 毫秒 成功返回查询。
*/
```

运行结果如下：

### 完整性约束

SQL支持多种完整性约束，当语言不符合完整性约束时，系统就会报错，不会应用你命令对应的更改

**primary key (Aj1, Aj2, …, Ajm)**：声明这些属性构成主键。主键属性必须**非空且唯一**：任何元组的主键属性都不能为 null，且关系中不允许两条元组在所有主键属性上取值相同。虽然声明主键是可选的，但是一般还是会声明一个主键出来 ，约定俗成的习惯

我们这样创建一个关系：

```sql
create table students2a
( studentID null primary key

)
/*报错如下：
错误： 语法错误 在 "null" 或附近的
LINE 2: ( studentID null primary key
                    ^ 

错误:  语法错误 在 "null" 或附近的
SQL 状态: 42601
字符: 37
*/
```

我们再创建一个关系：

```sql
create table students1a
(
studentID char(11) primary key,
studentName char(20)
)

```

接着我们运行下面的内容（往students1a里添加1个元组）

```sql
insert  into students1a (studentID,studentName)
values('20241003734','ccbMan')
/*
反馈：
INSERT 0 1

耗时29 毫秒 成功返回查询。
*/
```

我们再运行下面的内容（再次添加一个元组 但是主键的值是和刚添加的元组相同的）

```sql
insert  into students1a (studentID,studentName)
values('20241003734','ccbWoan')
/*
报错：
错误： 重复键违反唯一约束"students1a_pkey"
键值"(studentid)=(20241003734)" 已经存在 

错误:  重复键违反唯一约束"students1a_pkey"
SQL 状态: 23505
详细： 键值"(studentid)=(20241003734)" 已经存在*/
```

**oreign key (Ak1, Ak2, …, Akn) references s**：声明这些属性的取值必须对应于关系 **s** 中**主键属性**的某个取值组合。
具体的主键外键关系前面都讲过 ，这里不再进行演示 有兴趣可以自己跑一下

## 一些SQL指令

### drop table

这个指令用来删除已经创建过的关系

格式：drop table {关系名}

我们现在删除刚才创建的students1a：

```sql
drop table students1a

/*
反馈：
DROP TABLE

耗时26 毫秒 成功返回查询。
*/
```

### delete

此指令用来删除某个关系中的所有元组，但是保留这个关系的定义，相当于格式化或清空某个关系，注意 这个指令会删除某个关系的全部信息

### alter table

我们使用 alter table 来在已有的关系中新增 或删除属性，具体格式如下：

alter table 关系名 add 属性名 属性类型 

此时 关系的所有元组都会添加这个属性，只不过刚添加时所有值都是null。

如果想要删除某个属性就按下面的格式使用：

alter table 关系名 drop 属性名称

# SQL查询的基本结构

将已有的关系作为输入，经过一系列操作后 输出新的关系作为结果

## select

用来对单个关系进行查询

我们已知一个关系：教师。现在我们需要找出所有教师的姓名，就按下面这样操作

```sql
select name from instructor;
```

当然，如果一个较大的关系可能存在某个属性之下，其值会有多次重复的问题，有时会影响我们观看，此时我们可以使用distinct实现相同值只出现一次

```sql
select distinct name from instructor;
```

当然 和distinct语句对应，我们也有all语句表示不删除重复的值，但是默认情况下是按all输出的，只是多了一个显式的方案

```sql
select all name from instructor;
```

当然，对于搜索到的内容，我们可以使用运算符号进行数学运算后再显示给我们，这在涉及到统一变动一些信息的时候很有帮助。但是请注意，这个操作只是将搜索的属性，从单一的属性名称变成了表达式，并不会改变原有关系中，对应位置的值

```sql
select salary * 5 from instructor;
```

对于对搜索内容进行更深度的限制，我们可以使用where来实现在搜索条件上，添加表达式

```sql
select name from instructor;
where sex = 'female' and salary > 10000;
```

通过上面的指令，我们知道：我们要搜索的信息是：找到instructor关系中，所有工资大于10000的女性的名称

我们发现，where后面可以接多个表达式，我们可以在表达式之间使用and or not进行连接或修饰

### 多关系查询

前文已经知道了怎样搜索某一个关系中特定的数据，那么接下来，我们该怎么同时查询多个关系呢？

假设下面一个情景：我们需要查询某一个老师的姓名，以及她带的班里，有几个学生，我们知道，“老师”这个关系里面不会记录学生人数这个属性，而这个属性被保存在”课程“的属性里面，而老师会有外键，来和课程这个属性联系起来

那么我们就可以这样编写代码：

```sql
select name，course.student_amount from instructor,course;
where name = 'John' and instructor.course_id = course.id;
```

这样，我们就找到了john的姓名以及他的学生数（我们假设这里一个老师只带一个课程）。

我们发现，在select的后面，可以用关系.属性名的方式搜索属性名，这样避免了多个关系含有相同的属性名的时候产生的冲突问题。当然，我们也可以使用后面学到的重命名的方法来避免这个问题。from后面填写了多个被访问的关系名。

### 重命名操作 Rename Operation

回想前文提到的多个关系可能包含相同属性名的问题，我们可以用前缀的方式来区分每个属性，但是在内容输出的时候，又不会显示这个前缀——我们无法分辨这个属性到底是来自哪个关系，所以在查询过程中，我们可以直接在select后，指明输出的时候，这个属性名应该显示成怎样的属性名

格式：oldName as newName

那么，对于前面的查询指令，我们采用重命名的策略再写一遍

```sql
select name，student_amount as amount from instructor,course;
where name = 'John' and instructor.course_id = course.id;
```

是的，直接将原始名字后面加上as等内容就好了

不止属性名称可以使用这种重命名，关系名称也可以使用这种重命名：

当我们需要比较在相同的关系中比较不同元组，而且在使用元组名的规则不同时，我们就需要对两个相同的东西各自重命名。接下来，我们看下面的两种情景，体会用法：

我们在instroctor中找到工资最高的那个人的姓名（即工资高于这个关系的所有人）

```sql
select T.name from instructor as T , instructor as O
where T.salary >= O.salary
```

可见 符合要求的元组时T（instructor）中的某一个元组，其工资这个属性大于O（没错，还是这个instructor）中所有的元组的工资，也就表示出了最大的

当然，仅仅是嫌原来数据库中属性或关系的名字太长太复杂，想给自己减轻一些工作负担，用 as语句进行重命名也是一个不错的选择

### 字符串操作

首先必须要强调的点，和python不同，sql的字符串只能用单引号表示，双引号无法表示字符串，会报错。如果字符串本身就有单引号字符，我们就用两个单引号实现表示字符串，还是不能用双引号。

默认情况下，数据库系统对字符串并大小写并不敏感，我们可以修改一些参数来实现区分大小写

SQL有一些对字符串使用的函数，可以实现一些操作：连接，提取子串等

### like

like语句可以对字符串做模式匹配，我们可以使用百分号（表示任意子串）和下划线（匹配任意单个字符）的各种组合来实现各种功能：

- str%，匹配任何按str开头的字符串
- %str%所有包含str子串的字符串
- ____匹配任何长度为4的字符串
- ____%匹配任何长度至少是4的字符串

比如，我们要在教师的名单中，找到所有形式是John的老师：

```sql
select name from instructor
where name like %John%;
```

### select标签的子句的属性指定：*

就像python导入一些库一样：from pwn import *

*在sql中也可以表示所有

比如下面的内容表示引用当前关系的所有属性：

```sql
select * from instructor;
```

### 对输出元组进行顺序的控制：order by

order by语句可以将输出结果按照指定的顺序排列。默认情况下，order by 按照升序排列（对于字符串则是按照字母顺序），我们用desc表示降序，asc表示升序来显式规定排列的顺序

比如下面的代码，我们可以按姓名首字母顺序降序，同时按照工资升序输出输出姓名和工资

```sql
select name,salary from instructor 
order by name desc,salary 
```

### 使用between来优化搜索范围
在where语句中，我们可以指定搜索某个值的范围，我们通常的写法是：

```sql
select name from instructor 
where salary <= 100000 and salary >= 80000;
```

我们可以用between来简化这种表达，让语句看起来更清晰明白
```sql
select name from instructor 
where salary between 80000 and 10000;
```
注意，较小的数在前，较大的数在后就好
当然，如果我们想表达在某个范围之外，用not between就好

### 行为构造器
我们可以使用（n1,n2,……n）来表示一个元组，表示一群属性名
我们还可以根据这一群属性名，和另一个含同样数量的元组进行等号连接，表示搜索对应位置的属性名 = 对应位置的值的元组
这样同样是可以简化我们的搜索语言的
比如下面的搜索语句
```sql
select name, course_id
from instructor, teaches
where instructor.ID = teaches.ID and dept_name = 'Biology';
```
可以修改成：
```sql
select name, course_id
from instructor, teaches
where (instructor.ID, dept_name) = (teaches.ID, 'Biology');
```

## SQL集合操作
下面三种操作作用于关系，对应数学中的并集，交集，差集运算

### 并集union
语法：（某个关系，或查询结果的表格）union（另一个关系，或查询结果的表格）
比如，我们想要找到所有计算机系老师和物理系老师的名单
```sql
(select name
from instructor
where major = 'CS')
union
(
select name
from instructor
where major = 'Phy'
)
```
这是一种使用方法，但是在其他情况里，难免会出现两个关系都有的元组，在默认情况下，union操作会自动删掉相同的元组，只保留一个，但是我们可以使用union all 而非union来保留重复的元组

### 交集intersect
使用的格式和之前union相似，不过这次输出的是两个关系都有的元组
```sql
(select name
from instructor
where major = 'CS')
intersect
(
select name
from instructor
where major = 'Phy'
)
```
同样，我们使用intersect all来实现保留重复项

### 差集except
用法还是同上，这个运算结果是在except前的那个表格中，而不在except之后的表格中的元组
保留重复项，还是使用except all
```sql
(select name
from instructor
where major = 'CS')
except all
(
select name
from instructor
where major = 'Phy'
)
```

## 空值Null Values 和未知unknown
空值这个概念前面已经提到过很多次了，这里不多赘述了，简单来说就是这个元组的这个值没有
在运算中涉及空值会产生较大的问题
对于比较问题，如果和一个空值进行比较（除了 is （not） null），他的输出结果是unknown
注意，哪怕是null = null，返回的也是unknown
同时，这个结果还可以和True False进行布尔运算，运算规则如下：
- and：true and unknown 的结果是 unknown，false and unknown 的结果是 false，unknown and unknown 的结果是 unknown。
- or：true or unknown 的结果是 true，false or unknown 的结果是 unknown，unknown or unknown 的结果是 unknown。
- not：not unknown 的结果是 unknown
同时，SQL还有is unknown 和is not unknown来判断unknown
```sql
select name
from instructor
where salary > 10000 is unknown;
```

# 聚合函数
聚合函数将一组值（集合或多重集）作为输入内容，经过计算后输出一个单一值得函数
其有下面的几个函数：
- 平均值 avg
- 最小值 min
- 最大值 max
- 求和 sum
- 计数 count
其中，只有求和和平均值操作要求输入内容必须是数字集合

## 基本聚合
比如需要查询物理学院的教师的平均工资，我们可以使用avg聚合函数来计算
``` SQL
select avg(salary) from instructor
where dept_name = 'physics';
```
我们可以看到，输入的内容是整个学院名称为物理的教师的工资，而输出是一个单一的平均数

 ### count
 count是一种计数的函数，这种函数能计算一个关系的元组数，一个属性的值数
 假设我们要统计物理学院一共有多少个老师：
 ``` SQL
 select count(instructor_name) from instructor
 where dept_name = 'physics'
 ```

 ### distinct
同时，SQL提供了一个方法在计算之前消除重复项：使用distinct语句
 ``` SQL
 select avg(distinct salary) from instructor
 where dept_name = 'physics'
 ```
 同时，对于count( * )操作来说，其是不能使用distinct来小曲重复项的，因为星号表示了选择所有
 对于max和min来说，distinct是合法的，尽管这个操作不会让函数的结果发生实质性改变

## 分组聚合——group by
前面的聚合操作都只是输入一个关系或一组内容，输出一个结果，现在，使用group by可以实现按你自己想要的方式拆分输入的内容，并挨个计算输出结果

比如，我们想算这个学校每个学院的老师的平均工资，我们就不用傻傻的挨个输入一遍dept_name =  ， 而是利用group_by
``` SQL
select avg(salary) from instructor
group by(dept_name)
```

现在进阶一点，我们可以更细化地，搜索各个学院大二学生的平均成绩
``` SQL
select avg(credit) from students
where grade = 2
group by dept_name
```
注意，group by写在where语句之后

### having字句
如果我们想对group by结果进行进一步筛选，得到自己想要的结果，那having语句是一个不错的选择
同时，因为having语句是在分块之后运算的，所以having语句应该写在group语句之后
假如我想筛选最高薪水大于10000的学院的教师的平均薪水
``` SQL
select avg(salary) from instructors
group by dept_name
having max(salary) > 10000
```


## 含有NULL或布尔值的聚合
除了count（* ）之外，所有聚合函数运算都会自动忽略输入的空值

SQL:1999 引入了可以取 true、false、unknown 的布尔类型。聚合函数 some 和 every 可以应用于布尔集合，分别计算集合的析取（or）和合取（and）。

## 注意
如果select语句后出现了聚合函数，如sum（）或avg（），我们知道聚合函数只输出单一结果，则和其并列的被select项不能是没有聚合的项
看下面的例子
``` SQL
select dept_name, ID, avg(salary)
from instructor
group by dept_name;
```
我们发现，如果select了dept_name还好，那么输出的表格会按照dept_name下对应学院的平均薪水来输出，但是后面的ID就有问题了，一个学院会对应其下所有的ID，这在SQL的逻辑上是无法表示的
如果解决这个问题，那么我们需要将ID也加入分组，这样就能确定自己的ID之下仅仅有一个值

# 嵌套子查询
一种SQL提供的语法，实现在where语句之后再筛选我们想要的处理的部分
这些操作都是在大的查询里面放小的查询

## 是否是集合成员 Set Membership
我们可以使用 in 和 not in语句实现将选择的元组进一步筛选，是不是某个集合里的成员
例如，我们要找2017年讲课过，而不在2018年讲课的叫教授

``` SQL
select name from instructor as I
where I.course_id in (select C.course_id from courses as C 
where year = 2017)
and I.course_id not in (select C.course_id from courses AS C 
where year = 2018)
```

或者单纯有选择地列举一些东西
``` SQL
select name from instructor 
where name not in ('ccb','herman')
```


## 集合比较  Set Comparision

假设我们想查询：所有工资至少比生物系某一位老师的工资高的老师

这里涉及到：至少高于某个集合中的一个
可以使用> some()表示
``` SQL
select name from instructor
where salary > some(select salary from instructor
	where dept_name = 'Biology')
```

这样就可以实现了，some'之后的内容就是一个子查询
同时，任意的比较符号 + some都能实现特定功能
其中 = some 和in功能相同，但是<>some和 not in功能不同

同时，如果要求大于集合内的所用 ，我们可以用all（）表示所有
其使用方式和功能和some相同，但是<>all 和not in功能相同，=all 和in功能不同
（自己推）

## 空关系测试（Test for Empty Relations）
SQL包含一个测试子查询的结果是否为空的功能，使用exists语法或not exists（或except）语法
假设，我们查询生物学院的所有的老师：dansi
``` SQL
select name from instructor
where dept_name 
```



## 重复元组缺失测试
判断子查询结果是否有重复元组，我们可以用unique语句
比如，我们需要查询2017年最多开设一次的所有课程，其包含两个部分：只开设一次和没开设
``` SQL
select T.course_id from course as T
where unique(select R.course_id from section as R
where T.course_id = R.course_id and R.year = 2017)
```

对于这个子查询，先查询section中year = 2017的课程 再和course中的课程id对照，寻找符合条件的course中的课程id，同时unique函数保证筛选仅出现一次的
注意，如果unique内部是空集，unique会将其视作True处理

同理，not unique函数可以查询子查询里面的重复元组，即出现次数大于一的元组

# FROM子句中的子查询
在之前的嵌套查询中，我们用in接嵌套子查询的形式来表示一个查询结果，而指令可以接着这个结果操作，其实FROM也可以实现这种操作，其可以使用子查询表达式
比如，我们现在寻找平均薪水大于42000的部门名称和平均老师薪水，我们就可以这样编写指令：
```SQL
select dept_name ,avg_salary 
from(select dept_name avg(salary) as avg_salart 
	from instructor
	group by dept_name)
where avg_salary > 42000
```
我们发现，这里的group by语句在使用后无需再添加having语句，就能实现对结果筛选，就是因为子查询你在from中，而得到的结果还可以用where筛选
同时，我们也可以用as语句，对子查询的结果命名

下面是另一个例子，我们将找出所有部门中，讲师总薪水之和的最大值
```SQL
select sum_salary 
from(select dept_name,sum(salary) as sum_salary
from instructor froup by dept_name)
```


# WITH字句
with字句提供了一种定义临时关系的方法，该临时关系的定义仅对包含with子句的查询可用
```SQL
with max_budget (value) as
    (select max(budget)
     from department)
select budget
from department, max_budget
where department.budget = max_budget.value;
```

查询中的 `WITH` 子句定义了一个临时关系 **`max_budget`**，其中包含定义该关系（即计算最大预算）的子查询结果。该关系只能在同一查询的后续部分中使用。

例如，假设我们要查找所有总薪水大于所有部门平均总薪水的部门。我们可以使用 `WITH` 子句编写查询，如下所示：



```SQL
with dept_total (dept_name, value) as
    (select dept_name, sum(salary)
     from instructor
     group by dept_name),
dept_total_avg(value) as
    (select avg(value)
     from dept_total)
select dept_name
from dept_total, dept_total_avg
where dept_total.value > dept_total_avg.value;
```


# 标量子查询
SQL允许子查询出现在任何允许表达式返回单个值得位置，条件是该子查询只返回一个元组（一行）包含单个属性（一列）；这类子查询称为标量子查询
例如，子查询可用于 `SELECT` 子句中，如下例所示，该示例列出了所有部门以及每个部门的讲师人数：


```SQL
select dept_name,
       (select count(*)
        from instructor
        where department.dept_name = instructor.dept_name)
       as num_instructors
from department;
```

此示例中的子查询保证只返回一个值，因为它有一个没有 `GROUP BY` 的 `COUNT(*)` 聚合函数。该示例还说明了**关联变量**（Correlation Variables）的用法，即外部查询 `FROM` 子句中关系的属性，例如上例中的 `department.dept_name`。

标量子查询可以出现在 `SELECT`、`WHERE` 和 `HAVING` 子句中。标量子查询也可以定义时不带聚合函数。在编译时并不总是能确定一个子查询是否会返回多个元组；如果在执行子查询时结果包含多个元组，则会发生**运行时错误**。

请注意，从技术上讲，标量子查询的结果类型仍然是一个**关系**，即使它只包含一个元组。但是，当标量子查询用于需要值的表达式中时，SQL 会**隐式地**从该关系中**单一行、单列**中提取该值并返回。
## 无FROM子句的标量
某些查询需要进行计算，但不需要引用任何关系（表）。类似地，某些查询可能包含带有 `FROM` 子句的子查询，但顶级查询本身不需要 `FROM` 子句。

举个例子，假设我们希望求出**平均每位讲师所教授的课程节数**（不考虑年份或学期），其中由多位讲师教授的课程节数只计一次。我们需要计算 `teaches` 表中的元组数量（总教授节数）并计算 `instructor` 表中的元组数量（讲师人数）。然后简单的除法即可得到所需结果。可以将其写成：


```SQL
(select count (*) from teaches) / (select count (*) from instructor);
```

虽然这在某些系统中是合法的，但其他系统可能会因为**缺少 `FROM` 子句**而报告错误。在这种情况下，可以创建一个特殊的**虚拟关系**，例如命名为 **`dual`**，其中包含一个元组。这使得前面的查询可以写成：



```SQL
select (select count (*) from teaches) / (select count (*) from instructor)
from dual;
```

Oracle 提供了一个预定义的关系 `dual`，其中包含一个元组，用于上述用途（该关系有一个属性，但这与我们的目的无关）；如果您使用任何其他数据库，可以创建一个等效的关系。

由于上述查询是将一个整数除以另一个整数，因此在大多数数据库上，结果将是一个整数，这会导致**精度损失**。如果您希望将结果作为浮点数，可以在执行除法操作之前，将两个子查询结果中的一个乘以 `1.0`，将其转换为浮点数。