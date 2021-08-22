# mysql练习题45题

## 数据准备

```mysql
# 学生表 Student：

create table Student(

SId varchar(10) ,

Sname varchar(10),

Sage datetime,

Ssex varchar(10));




# 教师表 Teacher

create table Teacher(

TId varchar(10),

Tname varchar(10)); 



# 科目表 Course

create table Course(

CId varchar(10),

Cname nvarchar(10),

TId varchar(10)); 

# 成绩表 SC

create table SC(

SId varchar(10),

CId varchar(10),

score decimal(18,1)); 


------------ 插入数据语句-----------------

# 学生表 Student：

insert into Student values('01' , '赵雷' , '1990-01-01' , '男');

insert into Student values('02' , '钱电' , '1990-12-21' , '男');

insert into Student values('03' , '孙风' , '1990-05-20' , '男');

insert into Student values('04' , '李云' , '1990-08-06' , '男');

insert into Student values('05' , '周梅' , '1991-12-01' , '女');

insert into Student values('06' , '吴兰' , '1992-03-01' , '女');

insert into Student values('07' , '郑竹' , '1989-07-01' , '女');

insert into Student values('09' , '张三' , '2017-12-20' , '女');

insert into Student values('10' , '李四' , '2017-12-25' , '女');

insert into Student values('11' , '李四' , '2017-12-30' , '女');

insert into Student values('12' , '赵六' , '2017-01-01' , '女');

insert into Student values('13' , '孙七' , '2018-01-01' , '女');



# 科目表 Course

insert into Course values('01' , '语文' , '02'); 

insert into Course values('02' , '数学' , '01'); 

insert into Course values('03' , '英语' , '03'); 



# 教师表 Teacher

insert into Teacher values('01' , '张三');
 
insert into Teacher values('02' , '李四'); 

insert into Teacher values('03' , '王五'); 



# 成绩表 SC

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
```



## 1 

查询01课程比02课程成绩高的学生的学生信息及课程分数

```mysql
select s.*,sc1.score as 01_score,sc2.score as 02_score
from student s
left join (
	select * from sc
	where CId='01'
)sc1
on sc1.SId=s.SId
left join (
	select * from sc where CId='02'
)sc2
on sc2.SId=s.SId
where sc1.score>sc2.score

-- 教程解法
select *
from student t1
inner join sc t2
on t2.SId=t1.SId
inner join sc t3
on t3.SId=t1.SId 
	and t2.CId='01' and t3.CId='02'
where t2.score>t3.score
```



1.1 查询同时存在01课程和02课程的学生

```mysql
select t1.SId
from sc t1
inner join sc t2
on t2.SId=t1.SId and t1.CId='01' and t2.CId='02'

-- 教程解法
select *
from (select * from sc where CId='01')a
inner join (select * from sc where CId='02')b
on a.SId=b.SId
```



1.2 查询存在01课程但是可能不存在02课程的情况，不存在显示null

```mysql
select *
from (select * from sc where CId='01') t1
left join (select * from sc where CId='02') t2
on t2.SId=t1.SId 

-- 教程解法
select *
from sc t1
left join sc t2
on t2.SId=t1.SId and t2.CId='02' 
where t1.CId='01'
```

知识点：在使用inner join时，过滤条件放在on 和 where后面没有区别，但是left和right连接，on后面的过滤条件对主表无效，必须放在where后面。



1.3 查询不存在01课程但是存在02课程的情况，不存在显示null

```mysql
select *
from (select * from sc where CId='02') t1
left join (select * from sc where CId='01') t2
on t2.SId=t1.SId 
WHERE t2.SId is null
```



## 2

查询平均成绩大于等于60分的同学的编号,姓名和平均成绩

```mysql
select t1.SId,t2.Sname,t1.avg_score
from (
	select SId,avg(score)avg_score from sc group by SId HAVING avg(score) >= 60
)t1
inner join (
	select * from student
)t2
on t1.SId=t2.SId
```



## 3

查询平在sc表存在成绩的学生信息

```mysql
select t2.*
from (
	select distinct SId from sc 
)t1
inner join (
	select * from student
)t2
on t1.SId=t2.SId
```



## 4

查询所有同学的学生编号、学生姓名、选课总数、所有课程的总成绩

```mysql
select t1.SId,t1.Sname,t2.corse_count,t2.sum_score
from student t1
left join (
	select SId,count(*)corse_count,sum(score)sum_score
	from sc group by SId
)t2
on t1.SId=t2.SId

-- 解法2
select  t1.SId,count(t2.CId)corse_count,sum(t2.score)sum_score
from student t1
left join sc t2
on t1.SId=t2.SId
group by t1.SId
```



## 5

查询李姓老师数量



## 6

查询学过张三老师授课的同学信息

```mysql
select  t4.*
from (
	select TId from teacher where Tname='张三'
)t1
inner join course t2
on t2.TId=t1.TId
inner join sc t3
on t3.CId=t2.CId
inner join student t4
on t4.SId=t3.SId
```



## 7

查询没有学全所有课程的同学信息

```mysql
select t1.*
from student t1
left join (
	select SId,count(*)count
	from sc
	GROUP BY SId
)t2
ON t2.SId=t1.SId
where t2.count < (select count(*) from course) or t2.count is null
```



##  8

查询至少有一门课与01同学所学相同的同学信息

```mysql
select distinct t3.*
from sc t1
inner join (
	select CId
	from sc
	where SId='01'
)t2
on t2.CId=t1.CId
left join student t3
on t3.SId=t1.SId
```



## 9

查询与01同学所学课程完全相同的同学信息

```mysql
select *
from student where SId in (
select t1.SId
from(
	select * from sc 
	where SId not in (select SId from sc where CId not in (select CId from sc where SId='01'))
	and SId != '01')t1
group by t1.SId
having count(t1.SId) = (select count(*) from sc where SId='01')
)
```



## 10

查询没学过张三老师讲授的任意一门课程的学生姓名

```mysql
select * from 
student where SId not in (
select SId from sc where CId in (
select CId from course where TId=(select TId from teacher where Tname='张三'))
)

-- 教程解法
select * from 
student where SId not in (
	select SId
	from sc t1
	left join course t2
	on t2.CId=t1.CId
	left join teacher t3
	on t3.TId=t2.TId
	where t3.Tname='张三'
)
```



## 11

查询两门及以上课程不及格的同学的学号，姓名及成绩

```mysql
select t1.SId,AVG(t2.score),t3.Sname
from (select SId from sc
	where score < 60
	group by SId
	having count(*) > 1
)t1
inner join sc t2
ON t2.SId=t1.SId
left join student t3
on t3.SId=t1.SId
group by t1.SId
```



## 12

查询01课程小于60分，按分数降序排列学生的信息

```mysql
select t1.*
from student t1
left join sc t2
on t2.SId=t1.SId
where t2.CId='01'
and score < 60
order by score desc
```



## 13

按照平均成绩从高到低，显示所有学生的所有课程成绩及平均成绩

```mysql
select t1.*,t2.avg_score
from sc t1
left join (
select SId,avg(score)avg_score
from sc GROUP BY SId
)t2
on t2.SId=t1.SId
order by t2.avg_score desc
```



## 14