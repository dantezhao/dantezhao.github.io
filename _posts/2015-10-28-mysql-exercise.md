---
layout: post
title:  "Mysql/sql练习题"
categories: Database
tags:  mysql sql
---

* content
{:toc}

## 测试数据

测试数据集，总共四张表，以及一些初始化数据，模拟一个小的场景，练习使用。





```
create table Student(
Sid varchar(10),
Sname varchar(10),
Sage datetime,
Ssex varchar(10)
);

insert into Student values('01' , '赵雷' , '1990-01-01' , '男');
insert into Student values('02' , '钱电' , '1990-12-21' , '男');
insert into Student values('03' , '孙风' , '1990-05-20' , '男');
insert into Student values('04' , '李云' , '1990-08-06' , '男');
insert into Student values('05' , '周梅' , '1991-12-01' , '女');
insert into Student values('06' , '吴兰' , '1992-03-01' , '女');
insert into Student values('07' , '郑竹' , '1989-07-01' , '女');
insert into Student values('08' , '王菊' , '1990-01-20' , '女');
insert into Student values('09' , '孙吴昊' , '1990-01-20' , '女');
insert into Student values('10' , '赵雷' , '1990-01-20' , '女');

create table Course(
Cid varchar(10),
Cname varchar(10),
Tid varchar(10)
);


insert into Course values('01' , '语文' , '02');
insert into Course values('02' , '数学' , '01');
insert into Course values('03' , '英语' , '03');
insert into Course values('04' , '英语' , '01');


create table Teacher(
Tid varchar(10),
Tname varchar(10)
);

insert into Teacher values('01' , N'张三');
insert into Teacher values('02' , N'李四');
insert into Teacher values('03' , N'王五');
insert into Teacher values('04' , N'汪二蛋');

create table SC(
Sid varchar(10),
Cid varchar(10),
score decimal(18,1)
);

insert into SC values('01' , '01' , 80);
insert into SC values('01' , '02' , 90);
insert into SC values('01' , '03' , 99);
insert into SC values('02' , '01' , 70);
insert into SC values('02' , '02' , 60);
insert into SC values('02' , '03' , 80);
insert into SC values('03' , '01' , 80);
insert into SC values('03' , '02' , 80);
insert into SC values('03' , '03' , 80);
insert into SC values('04' , '01' , 50);
insert into SC values('04' , '02' , 30);
insert into SC values('04' , '03' , 20);
insert into SC values('05' , '01' , 76);
insert into SC values('05' , '02' , 87);
insert into SC values('06' , '01' , 31);
insert into SC values('06' , '03' , 34);
insert into SC values('07' , '02' , 89);
insert into SC values('07' , '03' , 98);
insert into SC values('08' , '03' , 98);
insert into SC values('09' , '03' , 98);
insert into SC values('10' , '04' , 59);

```


## 试题

### 题1

查询"01"课程比"02"课程成绩高的学生的信息及课程分数。

**答案1**

查询同时存在"01"课程和"02"课程的情况。

```

select a.*, b.score, c.score
from student a, sc b,sc c
where a.sid = b.sid and a.sid = c.sid
and b.cid ='01' and c.cid = '02'
and b.score > c.score;

```

**答案2**

查询：同时存在"01"课程和"02"课程的情况和存在"01"课程但可能不存在"02"课程的情况(不存在时显示为null)。

```

select a.*, b.score, c.score
from student a
left join sc b on a.sid = b.sid and b.cid = '01'
left join sc c on a.sid = c.sid and c.cid = '02'
where b.score > ifnull(c.score,0);

```

### 题2

查询平均成绩大于等于60分的同学的学生编号、学生姓名和平均成绩。

**答案1**

```
select a.sid, a.sname, avg(sc.score) avg_score
from student a, sc
where a.sid = sc.sid
group by a.sid, a.sname
having avg_score >= 60
order by a.sid;
```

**答案2**

使用cast转换精度。

```
select a.sid, a.sname, avg(cast(sc.score as decimal(18,2))) avg_score
from student a, sc
where a.sid = sc.sid
group by a.sid, a.sname
having avg_score >= 60
order by a.sid;
```

### 题3

查询平均成绩小于60分的同学的学生编号、学生姓名和平均成绩(包含无成绩的)。

**答案**

```
select a.sid, a.sname, avg(ifnull(sc.score,0)) avg_score
from student a left join sc
on a.sid = sc.sid
group by a.sname, a.sid
having avg_score < 60
order by a.sid;
```

### 题4

查询所有同学的学生编号、学生姓名、选课总数、所有课程的总成绩。


**答案1**

查询所有有成绩的SQL。

```
select a.sid, a.sname, count(b.cid), sum(b.score)
from student a inner join sc b
on a.sid = b.sid
group by a.sid, a.sname
order by a.sid;
```

**答案2**

查询所有(包括有成绩和无成绩)的SQL。

```
select a.sid, a.sname, count(b.cid), sum(b.score)
from student a left join sc b
on a.sid = b.sid
group by a.sid, a.sname
order by a.sid;
```

### 题5

查询"李"姓老师的数量。

**答案**

```
select count(tname)
from teacher
where tname like '李%';
```

### 题6

统计课程分01最高的前三名。

**答案**

```
select student.sname, sc.score, sc.cid
from student join sc on student.sid = sc.sid
where sc.cid='01'
order by sc.score desc limit 3;
```



### 题7

统计课程分最高的前三名,不分课程(分数的前三，人数可能大于三)

**答案1**

使用join。

```
select a.sname, a.score,  a.cid
from
(select Student.sname, sc.score, sc.cid
from Student
join SC on Student.Sid = SC.Sid) as a
join
(select distinct sc.score
from SC
order by score
desc limit 3) as b
on a.score = b.score;
```

**答案2**

使用in。

```
select a.sname, a.score,  a.cid
from
(select Student.sname, sc.score, sc.cid
from Student
join SC on Student.Sid = SC.Sid) as a
where a.score in
(select distinct sc.score from sc
order by sc.score
desc limit 3
);
```

### 题8

统计分数出现次数前三的分数

**答案**


```
select score, count(*)
from SC
group by score
order by count(*)
desc limit 3;
```


### 题9

统计每个学生选课总数。

**答案**

```
select a.sid, a.sname, count(b.cid)
from student a left join sc b
on a.sid=b.sid
group by a.sid,a.sname;
```

### 题10

每个老师教多少学生（一个教师给某个学生教两门课，计入两次）

**答案**

```
select t.tid, t.tname, a.countstu
from
(select c.tid as ctid, count(sc.cid) as countstu
from course c inner join sc
on c.cid = sc.cid
group by c.tid) a
right join teacher t
on t.tid = a.ctid;
```

******
2015-10-28 23:14:54 于bh xzl
