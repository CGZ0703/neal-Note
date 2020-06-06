# MySQL经典50题

```sql
-- 建表
-- 学生表
CREATE TABLE `Student`(
    `s_id` VARCHAR(20),
    `s_name` VARCHAR(20) NOT NULL DEFAULT '',
    `s_birth` VARCHAR(20) NOT NULL DEFAULT '',
    `s_sex` VARCHAR(10) NOT NULL DEFAULT '',
    PRIMARY KEY(`s_id`)
);
-- 课程表
CREATE TABLE `Course`(
    `c_id`  VARCHAR(20),
    `c_name` VARCHAR(20) NOT NULL DEFAULT '',
    `t_id` VARCHAR(20) NOT NULL,
    PRIMARY KEY(`c_id`)
);
-- 教师表
CREATE TABLE `Teacher`(
    `t_id` VARCHAR(20),
    `t_name` VARCHAR(20) NOT NULL DEFAULT '',
    PRIMARY KEY(`t_id`)
);
-- 成绩表
CREATE TABLE `Score`(
    `s_id` VARCHAR(20),
    `c_id`  VARCHAR(20),
    `s_score` INT(3),
    PRIMARY KEY(`s_id`,`c_id`)
);

-- 数据
-- 学生表测试数据
insert into Student values('01' , '赵雷' , '1990-01-01' , '男');
insert into Student values('02' , '钱电' , '1990-12-21' , '男');
insert into Student values('03' , '孙风' , '1990-05-20' , '男');
insert into Student values('04' , '李云' , '1990-08-06' , '男');
insert into Student values('05' , '周梅' , '1991-12-01' , '女');
insert into Student values('06' , '吴兰' , '1992-03-01' , '女');
insert into Student values('07' , '郑竹' , '1989-07-01' , '女');
insert into Student values('08' , '王菊' , '1990-01-20' , '女');
-- 课程表测试数据
insert into Course values('01' , '语文' , '02');
insert into Course values('02' , '数学' , '01');
insert into Course values('03' , '英语' , '03');
-- 教师表测试数据
insert into Teacher values('01' , '张三');
insert into Teacher values('02' , '李四');
insert into Teacher values('03' , '王五');
-- 成绩表测试数据
insert into Score values('01' , '01' , 80);
insert into Score values('01' , '02' , 90);
insert into Score values('01' , '03' , 99);
insert into Score values('02' , '01' , 70);
insert into Score values('02' , '02' , 60);
insert into Score values('02' , '03' , 80);
insert into Score values('03' , '01' , 80);
insert into Score values('03' , '02' , 80);
insert into Score values('03' , '03' , 80);
insert into Score values('04' , '01' , 50);
insert into Score values('04' , '02' , 30);
insert into Score values('04' , '03' , 20);
insert into Score values('05' , '01' , 76);
insert into Score values('05' , '02' , 87);
insert into Score values('06' , '01' , 31);
insert into Score values('06' , '03' , 34);
insert into Score values('07' , '02' , 89);
insert into Score values('07' , '03' , 98);
-- --------------------- 

-- 1、查询"01"课程比"02"课程成绩高的学生的信息及课程分数  

SELECT st.s_name,sc1.s_score 01_score,sc2.s_score 02_score
FROM Student st
LEFT JOIN Score sc1
ON st.s_id = sc1.s_id AND sc1.c_id = '01'
LEFT JOIN Score sc2
ON st.s_id = sc2.s_id AND sc2.c_id = '02'
WHERE sc1.s_score>sc2.s_score

-- 2、查询"01"课程比"02"课程成绩低的学生的信息及课程分数

SELECT st.s_name,sc1.s_score 01_score,sc2.s_score 02_score
FROM Student st
LEFT JOIN Score sc1
ON st.s_id = sc1.s_id AND sc1.c_id = '01'
LEFT JOIN Score sc2
ON st.s_id = sc2.s_id AND sc2.c_id = '02'
WHERE sc1.s_score<sc2.s_score

-- 3、查询平均成绩大于等于60分的同学的学生编号和学生姓名和平均成绩

SELECT st.s_name,AVG(sc.s_score) avg_score
FROM Student st
LEFT JOIN Score sc
ON st.s_id = sc.s_id
GROUP BY st.s_name
HAVING AVG(sc.s_score)>60


-- 4、查询平均成绩小于60分的同学的学生编号和学生姓名和平均成绩
       -- (包括有成绩的和无成绩的)

SELECT st.s_name, AVG(sc.s_score) avg_score
-- SELECT st.s_name, sc.s_score
FROM Student st
LEFT JOIN Score sc
ON st.s_id = sc.s_id
GROUP BY st.s_name
HAVING AVG(sc.s_score)<60
OR AVG(sc.s_score) is null

-- 5、查询所有同学的学生编号、学生姓名、选课总数、所有课程的总成绩

SELECT st.s_id, st.s_name, COUNT(sc.c_id), SUM(sc.s_score)
FROM Student st
LEFT JOIN Score sc
ON st.s_id = sc.s_id
GROUP BY st.s_name

-- 6、查询"李"姓老师的数量 

SELECT COUNT(t_name)
-- SELECT *
FROM Teacher
WHERE t_name LIKE '李%'

-- 7、查询学过"张三"老师授课的同学的信息 

SELECT *
FROM Student st
WHERE s_id IN(
	SELECT sc.s_id
	FROM Score sc, Course c, Teacher t
	WHERE sc.c_id = c.c_id AND c.t_id = t.t_id AND t.t_name = '张三'
)

-- 8、查询没学过"张三"老师授课的同学的信息 

SELECT *
FROM Student st
WHERE s_id NOT IN(
	SELECT s_id
	FROM Score sc, Course c, Teacher t
	WHERE sc.c_id = c.c_id AND c.t_id = t.t_id AND t.t_name =	'张三'
)


-- 9、查询学过编号为"01"并且也学过编号为"02"的课程的同学的信息

SELECT st.
FROM Student st
LEFT JOIN Score sc1
ON st.s_id = sc1.s_id AND sc1.c_id = '01'
LEFT JOIN Score sc2
ON st.s_id = sc2.s_id AND sc2.c_id = '02'
WHERE sc1.s_score is not null AND sc2.s_score is not null

-- 10、查询学过编号为"01"但是没有学过编号为"02"的课程的同学的信息

SELECT st.*
FROM Student st
LEFT JOIN Score sc1
ON st.s_id = sc1.s_id AND sc1.c_id = '01'
LEFT JOIN Score sc2
ON st.s_id = sc2.s_id AND sc2.c_id = '02'
WHERE sc1.s_score is not null AND sc2.s_score is null

-- 11、查询没有学全所有课程的同学的信息 

SELECT *
FROM Student
WHERE s_id NOT IN(
	SELECT s_id-- ,COUNT(c_id)
	FROM Score
	GROUP BY s_id
	HAVING COUNT(c_id) = (
		SELECT COUNT(*)
		FROM Course
	)
)

-- 12、查询至少有一门课与学号为"01"的同学所学相同的同学的信息 

SELECT *
FROM Student
WHERE s_id IN(
	SELECT s_id
	FROM Score
	WHERE c_id IN(
		SELECT c_id
		FROM Score
		WHERE s_id = '01'
	)
	AND s_id <> '01'
)


-- 13、查询和"01"号的同学学习的课程完全相同的其他同学的信息 

SELECT *
FROM Student
WHERE s_id IN(
	SELECT s_id
	FROM Score
	GROUP BY s_id
	HAVING GROUP_CONCAT(c_id) = (
		SELECT GROUP_CONCAT(c_id)
		FROM Score
		GROUP BY s_id
		HAVING s_id = 01
	)
	AND s_id <> '01'
)

-- 14、查询没学过"张三"老师讲授的任一门课程的学生姓名 

SELECT *
FROM Student st
WHERE s_id NOT IN(
	SELECT s_id
	FROM Score sc, Course c, Teacher t
	WHERE sc.c_id = c.c_id AND c.t_id = t.t_id AND t.t_name =	'张三'
)

-- 15、查询两门及其以上不及格课程的同学的学号，姓名及其平均成绩 

SELECT st.s_name, AVG(sc.s_score) agv_score
FROM Student st
LEFT JOIN Score sc
ON st.s_id = sc.s_id
GROUP BY st.s_id
HAVING st.s_id IN (
	SELECT sc.s_id
	FROM Score sc
	WHERE sc.s_score <60
	GROUP BY sc.s_id
	HAVING COUNT(c_id)>=2
)

-- 16、检索"01"课程分数小于60，按分数降序排列的学生信息

SELECT st.s_name, sc.s_score
FROM Student st
JOIN Score sc
ON st.s_id = sc.s_id
AND sc.c_id = '01'
AND sc.s_score < 60
ORDER BY s_score DESC

-- 17、按平均成绩从高到低显示所有学生的所有课程的成绩以及平均成绩

SELECT st.s_name, sc.s_score, savg.avg_score
FROM Student st ,(
	SELECT st.s_id, AVG(sc.s_score) avg_score
	FROM Student st
	LEFT JOIN Score sc
	ON st.s_id = sc.s_id
	GROUP BY st.s_id
) savg,Score sc
WHERE st.s_id = sc.s_id
AND st.s_id = savg.s_id
ORDER BY savg.avg_score DESC

-- 18.查询各科成绩最高分、最低分和平均分：以如下形式显示：课程ID，课程name，最高分，最低分，平均分，及格率，中等率，优良率，优秀率
-- 及格为>=60，中等为：70-80，优良为：80-90，优秀为：>=90

SELECT c.c_id,c.c_name,max(s.s_score) max,min(s.s_score) min,avg(s.s_score) avg,COUNT(s.jige)/COUNT(s.s_score) jg,COUNT(s.zhongdeng)/COUNT(s.c_id) zd,COUNT(s.youliang)/COUNT(s.c_id) yl,COUNT(s.youxiu)/COUNT(s.c_id) yx
FROM Course c
LEFT JOIN (
	SELECT s.s_id,s.c_id,s.s_score,
	IF(s1.s_score>=60 ,
		s1.s_score,
		null) as jige,
	IF(s2.s_score>=70 AND s2.s_score<80,
		s2.s_score,
		null) as zhongdeng,
	IF(s3.s_score>=80  AND s3.s_score<90 ,
		s3.s_score,
		null) as youliang,
	IF(s4.s_score>=90 ,
		s4.s_score,
		null) as youxiu
	FROM Score s, Score s1, Score s2, Score s3, Score s4
	WHERE s.s_id = s1.s_id AND s.s_id = s2.s_id AND s.s_id = s3.s_id AND s.s_id = s4.s_id AND
	s.c_id = s1.c_id AND s.c_id = s2.c_id AND s.c_id = s3.c_id AND s.c_id = s4.c_id
) s
ON c.c_id = s.c_id
GROUP BY c.c_id

-- 19、按各科成绩进行排序，并显示排名(实现不完全)

SELECT
	a.c_id,
	a.s_id,
	a.s_score,
	count( b.s_score ) + 1 AS rating
FROM
	Score AS a
	LEFT JOIN Score AS b ON a.s_score < b.s_score 
	AND a.c_id = b.c_id 
GROUP BY
	a.c_id,
	a.s_id,
	a.s_score 
ORDER BY
	a.c_id ASC,a.s_score DESC

-- 20、查询学生的总成绩并进行排名 

SELECT s_id,SUM(s_score) sum
FROM Score
GROUP BY s_id
ORDER BY sum DESC

-- 21、查询不同老师所教不同课程平均分从高到低显示 

SELECT c_id,AVG(s_score) sc
FROM Score
GROUP BY c_id
ORDER BY sc DESC

-- 22、查询所有课程的成绩第2名到第3名的学生信息及该课程成绩

SELECT st.*,sc.c_id,sc.s_score,count(s.s_score)+1 cs
FROM Score sc
left JOIN Student st
ON st.s_id = sc.s_id
LEFT JOIN Score s
ON sc.c_id = s.c_id AND sc.s_score < s.s_score
GROUP BY sc.s_id,sc.c_id
HAVING cs =2 OR cs = 3

-- 23、统计各科成绩各分数段人数：课程编号,课程名称,[100-85],[85-70],[70-60],[0-60]及所占百分比

SELECT c.c_id,c.c_name,s.*
FROM Course c,(
	SELECT c_id, 
	SUM(
		CASE 
		WHEN s_score<60 
			THEN 1 
			ELSE 0 
		END)/COUNT(*) AS '[0-60]',
	SUM(
		CASE 
		WHEN s_score>=60 AND s_score<70
			THEN 1 
			ELSE 0 
		END)/COUNT(*) AS '[60-70]',
	SUM(
		CASE 
		WHEN s_score>=70 AND s_score<85
			THEN 1 
			ELSE 0 
		END)/COUNT(*) AS '[70-85]',
	SUM(
		CASE 
		WHEN s_score>=85 AND s_score<=100
			THEN 1 
			ELSE 0 
		END)/COUNT(*) AS '[85-100]'
	FROM Score
	GROUP BY c_id
) s
WHERE c.c_id = s.c_id

-- 24、查询学生平均成绩及其名次 

SET @rowno := 0;
SELECT t.*,(@rowno := @rowno+1) rowno
FROM(
	SELECT s_id, AVG(s_score) avg_score
	FROM Score
	GROUP BY s_id
	ORDER BY avg_score DESC
) t

-- 25、查询各科成绩前三名的记录

SELECT sc.*,COUNT(s.s_score)+1 rating
FROM Score sc
LEFT JOIN Score s
ON sc.c_id = s.c_id
AND s.s_score>sc.s_score
GROUP BY sc.c_id,sc.s_id
HAVING COUNT(s.s_score)<3

-- 26、查询每门课程被选修的学生数  

SELECT c_id,COUNT(s_id) s_num
FROM Score
GROUP BY c_id

-- 27、查询出只有两门课程的全部学生的学号和姓名 

SELECT sc.s_id,st.s_name
FROM Score sc
LEFT JOIN Student st
ON sc.s_id = st.s_id
GROUP BY sc.s_id
HAVING COUNT(sc.c_id)='2'

-- 28、查询男生、女生人数 

SELECT '男' sex,COUNT(smale.s_sex) count
FROM Student smale
WHERE smale.s_sex = '男'
UNION ALL
SELECT '女' sex,COUNT(sfemale.s_sex) count
FROM Student sfemale
WHERE sfemale.s_sex = '女'

-- 29、查询名字中含有"风"字的学生信息

SELECT *
FROM Student
WHERE s_name LIKE '%风%'

-- 30、查询同名同性学生名单，并统计同名人数 

SELECT *, COUNT(s_id) s_num
FROM Student
GROUP BY s_name,s_sex
HAVING s_num > 1

-- 31、查询1990年出生的学生名单

SELECT *
FROM Student
WHERE YEAR(s_birth) = 1990

-- 32、查询每门课程的平均成绩，结果按平均成绩降序排列，平均成绩相同时，按课程编号升序排列 

SELECT avg(s_score) avg_score, c_id
FROM Score
GROUP BY c_id
ORDER BY avg_score DESC, c_id

-- 33、查询平均成绩大于等于85的所有学生的学号、姓名和平均成绩 

SELECT sc.s_id, st.s_name, AVG(sc.s_score) avg_score
FROM Score sc,Student st
WHERE sc.s_id = st.s_id
GROUP BY sc.s_id
HAVING avg_score > 85
ORDER BY avg_score DESC

-- 34、查询课程名称为"数学"，且分数低于60的学生姓名和分数 

SELECT st.s_name, sc.s_score
FROM Score sc,Student st, Course c
WHERE sc.s_id = st.s_id
AND sc.c_id = c.c_id AND c.c_name = '数学'
AND sc.s_score<60

-- 35、查询所有学生的课程及分数情况；

SELECT st.s_name,c.c_name,sc.s_score
FROM Student st
LEFT JOIN Score sc
ON st.s_id = sc.s_id
LEFT JOIN Course c
ON sc.c_id = c.c_id

-- 36、查询任何一门课程成绩在70分以上的姓名、课程名称和分数； 

SELECT st.s_name,c.c_name,sc.s_score
FROM Score sc
LEFT JOIN Student st
ON st.s_id = sc.s_id
LEFT JOIN Course c
ON sc.c_id = c.c_id
WHERE s_score>70

-- 37、查询不及格的课程

SELECT st.s_name,c.c_name,sc.s_score
FROM Score sc
LEFT JOIN Student st
ON st.s_id = sc.s_id
LEFT JOIN Course c
ON sc.c_id = c.c_id
WHERE s_score<60

-- 38、查询课程编号为01且课程成绩在80分以上的学生的学号和姓名； 

SELECT st.s_name,c.c_name,sc.s_score
FROM Score sc
LEFT JOIN Student st
ON st.s_id = sc.s_id
LEFT JOIN Course c
ON sc.c_id = c.c_id
WHERE s_score>=80
AND sc.c_id = '01'

-- 39、求每门课程的学生人数 

SELECT c.c_name,COUNT(s.s_id) s_num
FROM Score s,Course c
WHERE s.c_id = c.c_id
GROUP BY s.c_id

-- 40、查询选修"张三"老师所授课程的学生中，成绩最高的学生信息及其成绩

SELECT st.*,sc.*
FROM Score sc
LEFT JOIN Student st
ON sc.s_id = st.s_id
WHERE (sc.c_id,sc.s_score) IN
(
	SELECT sc.c_id,MAX(sc.s_score)
	FROM Score sc
	WHERE sc.c_id IN (
		SELECT c_id
		FROM Course
		WHERE t_id = (
			SELECT t_id
			FROM Teacher
			WHERE t_name = '张三'
		)
	)
	GROUP BY c_id
)

-- 41、查询不同课程成绩相同的学生的学生编号、课程编号、学生成绩 

SELECT st.*,sc.c_id, sc.s_score,s.c_id, s.s_score
FROM Score sc
LEFT JOIN Student st
ON sc.s_id = st.s_id
LEFT JOIN Score s
ON sc.s_id = s.s_id
AND sc.c_id <> s.c_id
AND sc.s_score = s.s_score
WHERE s.s_id is not null
ORDER BY sc.c_id,s.c_id DESC

-- 42、查询每门功成绩最好的前两名 

SELECT sc.*,COUNT(s.s_score)+1 rating
FROM Score sc
LEFT JOIN Score s
ON sc.c_id = s.c_id
AND s.s_score>sc.s_score
GROUP BY sc.c_id,sc.s_id
HAVING COUNT(s.s_score)<2

-- 43、统计每门课程的学生选修人数（超过5人的课程才统计）。要求输出课程号和选修人数，查询结果按人数降序排列，若人数相同，按课程号升序排列  

SELECT sc.c_id,COUNT(s_id) s_num
FROM Score sc
GROUP BY c_id
HAVING COUNT(s_id)>=5
ORDER BY s_num DESC, c_id

-- 44、检索至少选修两门课程的学生学号 

SELECT st.*
FROM Student st
LEFT JOIN Score sc
ON st.s_id = sc.s_id
GROUP BY st.s_id
HAVING COUNT(sc.c_id) >1

-- 45、查询选修了全部课程的学生信息 

SELECT *
FROM Student st
LEFT JOIN Score sc
ON st.s_id = sc.s_id
GROUP BY st.s_id
HAVING COUNT(sc.c_id) = (
	SELECT COUNT(*)
	FROM Course
)

-- 46、查询各学生的年龄

SELECT *, (YEAR(NOW())-YEAR(s_birth)) age
FROM Student

-- 47、查询本周过生日的学生

SELECT *
FROM Student
WHERE MONTH(NOW()) = MONTH(s_birth)
AND ABS((DAY(NOW()) - DAY(s_birth))) < 8

-- 48、查询下周过生日的学生

SELECT *
FROM Student
WHERE MONTH(NOW()) = MONTH(s_birth)
AND (DAY(s_birth) - DAY(NOW())) < 15
AND (DAY(s_birth) - DAY(NOW())) > 7

-- 49、查询本月过生日的学生

SELECT *
FROM Student
WHERE MONTH(NOW()) = MONTH(s_birth)

-- 50、查询下月过生日的学生

SELECT *
FROM Student
WHERE (MONTH(s_birth) - MONTH(NOW())) 
```

