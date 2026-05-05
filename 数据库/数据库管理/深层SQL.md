# 连接表达式 Join Expressions
除了使用笛卡尔积将多个关系的信息组合起来之外，我们还可以使用join语句实现其他的连接操作，这种操作能实现一般笛卡尔积难以实现的东西

## 自然连接 The Natural Join
假设我们需要查询A班每个学生选修的课程：
```SQL
select name,course_id
from Astudent as A,takes as B
where A.id = B.id
```

我们看到，选择这个关系难免的会包含学生的编号，而表示班级成员的关系也会有，在之前，我们的方法都是重命名或用·来区分，这样的计算逻辑是：将这两个关系作笛卡尔积，然后筛选出id相等的元组作为结果，然后再做其他操作

而我们可以使用natural join'语句简化这个过程
natural join的工作原理是： 考虑两个关系中，某个属性名，属性值完全相同的两个元组，将所有符合要求的，来自不同关系的元组对进行拼接，而不符合要求的就不会出现
```SQL
select name,course_id
from Astudent natural join takes 
```

这样，运算逻辑就变成了：先将这两个关系中所有符合要求的元组拼接，在将拼接结果作其他操作
注意，自然连接的操作在多个元组互相结合的时候也是适用的、
```SQL
select A1, A2, ..., An
from r1 natural join r2 natural join ... natural join rm
where P;
```
而且，多个关系通过join相连，视为一个关系，如果我们在这个表达式之后接逗号+其他关系，也是成立的
比如，我们想查询A班同学修的课程的标题，这样就涉及到三个关系：A班名单，修课关系，以及课程信息
```SQL
select name, title
from Astudent natural join takes, course
where takes.course_id = course.course_id;
```
此时，Astudent和takes作自然连接后和course作笛卡尔积，再进行后续操作，这种操作是允许的
但是，如果将着三个关系都用自然连接连起来，那就事与愿违了

```SQL
select name, title
from Astudent natural join takes natural join course
where takes.course_id = course.course_id;
```
## 指定列连接
这样，因为student中含有dept属性，而course也有但是takes没有，这里的自然连接要求dept属性值也要相同，才能加入运算结果
这样就出现了一个差错：如果一个学生修了自己系外的课程，那么这个课程的名称将不包含在搜索结果里
当然，这种错误也有一个补救方案：r1  join r2 using（A，B）：当r1.A = r2.A 且 r1.B = r2.B时，才将其加入结果，不会在比较其他属性
我们称join using为指定列连接


## 连接条件
SQL还支持任意的连接条件，其表达在语句 on 之后，这样的连接编写更加灵活，但也需要对约束描述清楚
比如我们重复刚才的例子，结合连接条件的编写，我们就可以这样定义SQL语句：
```SQL
select name
from Astudent join takes on Astudent.id = takes.id
```

这样，系统就知道了：将这两个值相等的元组作匹配，但是从结果上和自然连接相比，其会包含两个关系的id，和自然连接的仅保留一个不同，实际上，其就是进行笛卡尔积再筛选的结果
我们可以用更暴力的形式，来实现只显示一个id：select的时候选择显示除了task的id之外的所有属性就好了
```SQL
select student.ID as ID, name, dept_name, tot_cred,
       course_id, sec_id, semester, year, grade
from student join takes on student.ID = takes.ID;
```
on支持任何SQL谓词，可在外连接的情况下使用，而on和where配合使用能更加其可读性