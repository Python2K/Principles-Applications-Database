##   数据库系统的用户接口以及SQL语言*

#### DBMS的用户接口

DBMS系统必须提供给用户接口来支持访问数据库包含：

* Query languages
  * Formal query language
  * Tabular query language
  * Graphic query language
  * Limited natural language query language
* Interface and maintaining tools(GUI)
* APIs
* Class Library

QL语言不是编程语言，没有计算能力，不具备**图灵完备**

#### SQL语言：relational calculus为基础

* 数据定义语言Date Definition Language（DDL）
* Query Language（QL）查询语言
* Data Manipulation Language（DML）数据操纵语言，主要由插入、删除、更新组成
* Data Control Language（DCL）控制权限



#### 重要的术语与概念

* Base table基表（真正物理存在的）
* View视图（并不是真正的表，而是根据需求临时算出来的）
* Data Type supported数据类型
* Null空值（表示未知）保留字
* Unique（是否允许重复值，是否唯一）保留字
* Default缺省值，保留字
* Primary Key主键
* Foreign Key外键（外键是另外一张表的主键）
* Check定义一些完整性约束

#### Basic SQL Query

| SELECT | [DISTINCT] target-list：DISTINCT是否对重复元组消除 |
| :----: | :------------------------------------------------: |
|  FROM  |                   relation-list                    |
| WHERE  |                   qualification                    |

   

* relation-list:所要涉及到的表
* target-list:要查询的目标属性的列表，要查询的若干属性的列表
* qualification:应该满足于什么条件

**概念上一条查询语句的执行**：仅仅是概念上的，如果这样做的话，效率会**非常非常低**（暂时这样理解，后面有查询优化器等。。。）

1. 首先做词法分析、
2. 根据from子句的关系列表做迪卡乘积（拼成一张大表）
3. 根据where子句的条件，把不满足的查询要求的tuple剔除掉
4. 根据select子句的target-list做一个投影
5. 看是否有DISTINCT，删除重复值

---

#### SQL查询方法

Sailors水手表：sid编号，sname姓名，rating等级，age年龄

Reserves预订表：sid编号，bid船号，day日期

**例1：下图为：要求查询预订了103这条船的水手姓名**

SELECT S.sname #取出S表中的sname

FROM Sailors S, Reserves R #涉及到的表，同时给Sailors取一个**别名S**，Reserves取了个**别名R**

WHERE S.sid=R.sid AND R.bid=103 #满足条件

![sqlsearch](https://github.com/Python2K/Principles-Applications-Database/blob/master/images/sqlsearch.png?raw=true)

---

**例2：至少预订过一条船的水手**

方法1：不需要做迪卡尔乘积，只需要将R表的sid投影至S表的sid列，只要有sid重复，那么那个sid必然订过船

方法2：做迪卡尔乘积，取出S.sid=R.sid的条件的编号

```sql
#方法2
SELECT S.sid
FROM Sailors S，Reserves R
WHERE S.sid=R.sid
如果取值是sid时，可以使用DISTINCT,将重复的数据去掉，对结果的语义没有影响
如果取值是sname的话，就不可以使用DISTINCT,因为可能存在sname重名
----------------------------------------------------------------
**如果取sname的话，加入DISTINCT删除重复值并不引起混淆，应该多SELECT加入主键，示例如下**
SELECT DISTINCT S.sid,S.sname
FROM Sailors S，Reserves R
WHERE S.sid=R.sid

```

---

**例3：SELECT子句中可以使用表达式**

```sql
SELECT S.age,age1=S.age-5,2*S.age AS age2#结果属性增加age1,age2,"="来表达式，"AS"是另外一种表示，给属性起名并计算的2种方法
FROM Sailors S
WHERE S.sname LIKE 'B_%B'#支持LIKE模糊查询，_表示任意一个字符，%表示任何0到多个字符，**不同系统不一样**
```

---

**例4：预订过红船或者是绿船的水手编号**

```sql
#两种方式
SELECT S.sid
FROM Sailors S,Boats B,Reserves R
WHERE S.sid=R.sid AND R.bid=B.bid AND (B.color='RED' OR B.COLOR='GREEN')
------------------------------------------------------------------------------------------
SELECT S.sid#找出红船的水手编号
FROM Sailors S,Boats B,Reserves R
WHERE S.sid=R.sid AND R.bid=B.bid AND B.color='RED'

UNION#并操作，需要满足并兼容条件

SELECT S.sid#找出红船的绿手编号
FROM Sailors S,Boats B,Reserves R
WHERE S.sid=R.sid AND R.bid=B.bid AND B.color='GREEN'

```

**例5：同时预订了红船与绿船的水手编号**

```sql
#对boats船表与reserves预订表进行自连接，这样表的tuple会出现两次,不能简单的对例4的第一种方式简单的把OR换成AND，因为一条tuple中颜色不可能同时是红与绿
#！这种写法效率不高，逻辑上没问题
SELECT S.sid
FROM Sailors S, Boats B1,Boats B2,Reserves R1,Reserves R2
WHERE S.sid=R1.sid AND R1.bid=B1.bid 
      AND S.sid=R2.sid And R2.bid=B2.bid
      AND (B1.color='RED' AND B2.color='GREEN')
------------------------------------------------------------------------------------------
#使用集合的交集来查询
SELECT S.sid
FROM Sailors S,Boats B,Reserves R
WHERE S.sid=R.sid AND R.bid=B.bid AND B.color='RED'
INTERSECT
SELECT S.sid
FROM Sailors S,Boats B,Reserves R
WHERE S.sid=R.sid AND R.bid=B.bid AND B.color='GREEN'
------------------------------------------------------------
#第三种方法
SELECT S.sid
FROM Sailors S,Boats B,Reserves R
WHERE S.sid=R.sid AND R.bid=B.bid AND B.color='RED'
AND S.sid IN (SELECT S2.sid
             FROM Sailors S2,Boats B2,Reserves R2
             WHERE S2.sid=R2.sid AND R2.bid=B2.bid 
             AND B2.color='GREEN')
```

**例6:非关联嵌套查询,订过103号船水手的姓名**

```sql
SELECT S.sname
FROM sailors S
WHERE S.sid IN (SELECT R.sid
                FROM Reserves R
                WHERE R.bid=103)
```

**例7：关联嵌套查询，订过103号船水手的姓名**

```sql
SELECT S.sname
FROM Sailors S
WHERE EXISTS (SELECT *
             FROM Reserves R
             WHERE R.bid=103 AND S.sid=R.sid)
```

**例8：只被订了一次的船的编号**

```sql
SELECT bid
FROM Reserves R1
WHERE bid NOT IN (SELECT bid
                 FROM Reserves R2
                 WHERE R2.bid != R1.bid)#代表了被订过1次以上的船的bid
```

**例9：找出rating大于sname为Horatio的tuple**

```sql
SELECT *
FROM Sailors S
WHERE S.rating > ANY (Select S1.rating
                 FROM Sailors S1
                 WHERE S1.sname='Horatio')大于任何集合中的一个
```

**例10：找出谁预订了所有的船，名字**

否定之否定，先使用所有船减去水手订的船，如果不存在这么一个结果，说明这个水手预订了所有的船

```sql
#solution 1:
SELECT S.sname
FROM Sailors S
WHERE NOT EXISTS(#如果不存在，那么找出了结果，集合代表船员没定的船
    	(SELECT B.bid#所有的船
		 FROM Boats B)
        EXCEPT#减
        (SELECT R.bid#外面这个水手定过的船
        FROM Reserves R
        WHERE R.sid=S.sid)       
		)#得到的结果是这个选手没有定过的船
		
#solution 2:
SELECT S.sname
FROM Sailors S
WHERE NOT EXISTS (SELECT B.bid
                 FROM Boats B
                 WHERE NOT EXISTS (
                 				SELECT R.bid 
                 				FROM Reserves R
                 				WHERE R.bid=B.bid AND R.sid=S.sid))
```

#### Aggregate Operators

* COUNT(*)
* COUNT([DISTINCT] A)
* SUM([DISTINCT] A)
* AVG([DISTINCT] A)
* MAX(A)
* MIN(A)

例：

![operators](https://github.com/Python2K/Principles-Applications-Database/blob/master/images/operators.png?raw=true)



#### 两条新sql子句

* GROUP BY:grouping-list
* HAVING :group-qualification

**例：每一个rating中最年龄水手的年龄，前提（每个级别中年龄大于18岁的人员大于2个的rating才符合条件）**![groupsearch](https://github.com/Python2K/Principles-Applications-Database/blob/master/images/groupsearch.png?raw=true)

```sql
内部运作：
1、进行WHERE判断排除(where子句的作用是在对查询结果进行分组前，将不符合where条件的行去掉，即在分组之前过滤数据，条件中不能包含聚组函数，使用where条件显示特定的行。 )
2、先进行排序
3、进行HAVING 条件判断(having子句的作用是筛选满足条件的组，即在分组之后过滤数据，条件中经常包含聚组函数，使用having条件显示特定的组，也可以使用多个分组标准进行分组。)
**PS：HAVING,SELECT子句中出现的内容必须是GROUP BY 集合的子集，group by语句中select和having指定的字段必须是“分组依据字段”，其他字段若想出现在select或having中则必须包含在聚合函数中，常见的聚合函数如下表：

函数	作用	支持性
sum(列名)	求和	　　　　
max(列名)	最大值	　　　　
min(列名)	最小值	　　　　
avg(列名)	平均值	　　　　
first(列名)	第一条记录	仅Access支持
last(列名)	最后一条记录	仅Access支持
count(列名)	统计记录数	注意和count(*)的区别
```

![group](https://github.com/Python2K/Principles-Applications-Database/blob/master/images/group.png?raw=true)

**例：根据船的bid分组，计算各分组红船的个数**

```sql
#solution1
SELECT B.bid COUNT(*) AS Scount
FROM Reserves R,Boats B
WHERE R.bid=B.bid AND B.color='RED'
GROUP BY B.bid
------------------------------------------------------------------------------------------
思考：如果将B.color判断放入HAVING是否可以
回答：不可以，因为如果查询语句中有GROUP BY，那么SELECT中的属性与HAVING中的属性必须在GROUP BY的属性中存在，否则报错,GROUP BY中的属性分组单单只是分组所写的属性
#solution2
SELECT B.bid COUNT(*) AS Scount
FROM Reserves R,Boats B
WHERE R.bid=B.bid
GROUP BY B.bid,B.color
HAVING B.color='RED'
------------------------------------------------------------------------------------------
#solution3
SELECT B.bid COUNT(*) AS Scount
FROM Reserves R,Boats B
WHERE R.bid=B.bid
GROUP BY B.bid
HAVING B.bid IN (SELECT bid     #B.bid在GROUP BY子句中，符合语法检查
             	FROM Boats B
                WHERE B.color='RED')#选择出所有红船的bid

```

**例：find age of the youngest sailor with age > 18,for each rating with at least 2 sailor(of any age)**

```sql
SELECT S.rating, MIN(S.age)
FROM Sailors S
WHERE S.age > 18
GROUP BY S.rating
HAVING 1 < (SELECT COUNT(*)
           FROM SAILORS S2
           WHERE S2.rating = S.rating)
```

**例：查找平均年龄最小的级别**

```sql
SELECT Temp.rating
FROM (SELECT S.rating,AVG(S.age) AS avgage
     FROM Sailors
     GROUP BY S.rating) AS Temp#新建立了一张对照表，内含rating与avgage这两个属性
WHERE Temp.avgage=(SELECT MIN(Temp.avgage)
                  FROM Temp)


```

---

**Null Values**

空值：不知道、未知的值

注意点：当WHERE rating > 8的时候，当有rating的值为Null时，返回值是Null，那这条tuple会被过滤掉。因此在写sql时候得考虑这部分数据是否需要，如果需要，需要加上WHERE rating>8 OR rating=Null

---

#### 一些SQL的新特性

* CAST expression
* CASE expression
* Sub-query
* Outer Join
* Recursion

---

**CAST expression**

语法图：

![castexpression](https://github.com/Python2K/Principles-Applications-Database/blob/master/images/castexpression.png?raw=true)



**使用方面：**

1. 用来在做函数调用时使得形参与实参间符合类型的匹配:substr(string1,CAST(x AS Integer),CAST(x AS Integer))
2. 改变计算的精度：CAST(elevation AS Decimal(5,0))
3. 给空值赋予一个类型，让来符合计算的要求：

![castexample](https://github.com/Python2K/Principles-Applications-Database/blob/master/images/castexample.png?raw=true)

---

**CASE Expression**

```sql
Simple form:

Officers(name,status,rank,title)

SELECT name,CASE status
					WHEN 1 THEN 'Active Duty'
					WHEN 2 THEN 'Reserve'
					WHEN 3 THEN 'Special Assignment'
					WHEN 4 THEN 'Retired'
					ELSE 'Unknown'
				END AS status
FROM Officers;
#对各种属性的编码值，表进行投影，对1，2，3，4自动转换，更human

					
```

**例：当type='chain saw'时,对accidents求合，否则等于0e0，然后除以xxx，算出概率**
![Screen Shot 2018-08-14 at 3.52.18 PM](https://github.com/Python2K/Principles-Applications-Database/blob/master/images/Screen Shot 2018-08-14 at 3.52.18 PM.png?raw=true)

---

**Sub-query**

1. 查询结果只有一个值的查询，是标量子查询Scalar-query

```sql
SELECT d.deptname,d.location
FROM dept AS d
WHERE (SELECT avg(honus)
      FORM emp WHERE deptno=d.deptno)
      > (SELECT avg(salary)
        FROM emp
        WHERE deptno=d.deptno)
```

```sql
SELECT D.deptno,d.deptname,(SELECT MAX(salary)
                           FROM emp
                           WHERE deptno=d.deptno) AS maxpay)
FROM dept AS d
WHERE d.locatio='NEW YORK'
```

2. Table expression表表达式

在FROM子句中创建临时表，就是表表达式

```sql
SELECT Temp.rating
FROM (SELECT S.rating,AVG(S.age) AS avgage
     FROM Sailors
     GROUP BY S.rating) AS Temp#新建立了一张对照表，内含rating与avgage这两个属性
WHERE Temp.avgage=(SELECT MIN(Temp.avgage)
                  FROM Temp)
```

3.  common table expressin公共表表达式

当一个临时表需要多次使用的时候，每次都去重新写一次是没有效率的，这样让这张表多次使用的话，叫做公共表表达式，可以当作一个临时视图。

例：找一个部门，这个部门总收入最高

```sql
WITH PAYROLL(DEPTNO,TOTALPAY) AS (SELECT deptno,sum(salary)+sum(bonus)
                                 FROM emp
                                 GROUP BY deptno)
#WITH 创建一个临时表                                 
SELECT deptno
FROM payroll
WHERE totalpay=(SELECT max(totaly)
               FROM payroll);
```

4. Outer Join外连接

```sql
WITH
	innerjoin(name,rank,subject,enrollment) AS#新建临时表innerjoin
        (SELECT t.name,t.rank,c.subject,c.enrollment
        FROM teachers AS t,courses AS c
    	WHERE t.name=c.teacher AND c.quarter='FALL 96')
    teacher-only(name,rank) AS #新建临时表teacher-only
    	(SELECT name,rank
        FROM teachers
        EXCEPT ALL
        SELECT name,rank
        FROM innerjoin)
    course-only(subject,enrollment) AS
    	(SELECT subject,enrollment
        FROM courses)
#将多张临时表合并成一张表        
SELECT name,rank,subject,enrollment
FROM innerjoin
UNION ALL#使用CAST来符合并兼容要求
SELECT name,rank,CAST(NULL AS Varchar(20)) AS subject,CAST(NULL AS Integer) AS enrollment
FROM teacher-only
UNION ALL
SELECT CAST(NULL AS VARCHAR(20))AS name,CAST(NULL AS Varchar(20)) AS rank,subject,enrollment
FROM course-only;

    
```

5. recursion

*例1：给一张表，有名字，薪水，他的上级FedEmp(name,salary,manager)，找出在Hoover手下工作的薪水大于10万的人员（注意，直接查询只是查出Hoover的直接手下，还有层次多个上级之间的可能性）*

```sql
想办法做一张表，显示所有的hoover手下与薪水，一下子无法做出，所以作如下思考
1.先做出hoover的直接手下
2.然后查询步骤1的人员的手下
3.再查询步骤2查询的人员的手下
4.递归查询，直到没有

WITH agents(name,salary) AS
	(SELECT name,salary
	FROM FedEmp                       #初始查询，查直接手下initial query
	WHERE manager='Hoover')
	UNION ALL
	(SELECT f.name,f.salary
    FROM agents AS a,FedEmp AS f      #递归查询recursion query
    WHERE f.manager=a.name)
    
SELECT name,salary
FROM agents
WHERE salary>100000
```

*例2：经典零件搜索问题，主零件part，子零件subpart，每个主零件需要多少子零件QTY，要查询一个飞机机翼总共需要多少铆钉*

![Screen Shot 2018-08-21 at 2.58.18 PM](https://github.com/Python2K/Principles-Applications-Database/blob/master/images/Screen Shot 2018-08-21 at 2.58.18 PM.png?raw=true)

```sql
与例1思路一样，但是与例1不同的是，查询出的表都是1个零件的内容，需要做计算，才能算出多少个铆钉
所以需要做这么一张表
1.主零件的子零件一共需要多少个
2. 所有1步骤的子零件一共需要多少个子子零件
3. 递归

WITH wingpart(subpart,qty) AS
	((SELECT subpart,qty
     FROM components
     WHERE part='wing')
    UNION ALL
    (SELECT c.subpart,w.qty*c.qty
    FROM wingpart w,components c
    WHERE w.subpart=c.part))
然后分组计算就可以了
```

*例3：一张表，内有航班号，飞机起飞地，目的地，票价四个属性，现在要求纽约到旧金山最便宜价格的路么，此航线没有直达*

```sql
先要做一张表trips(destination，route,nsegs,totalcost)
destination目的地
route中转路径
nsegs中转次数
totalcost总支付

WITH trips(destination,route,nsegs,totalcost) AS
	((SELECT destination,CAST(destination AS varchar(20)),1,cost
     FROM flights
     WHERE origin='SFO')
    UNION ALL
    (SELECT f.destination,CAST(t.route||','||f.destination AS varchar(20)),t.nsegs+1,t.totalcost+f.cost
    FROM trips t,flights f
    WHERE t.destination=f.orgigin AND f.destination<>'SFO' AND f.origin<>'JFK' AND t.nsegs<=3))
    
    
SELECT route,totalcost
FROM trips
WHERE destination='JFK' AND nsegs=(SELECT min(nesgs)
                                  FROM trips
                                  WHERE destination='JFK')
```

-------------

**以上，查询语言部分完结**

---

# 

# Data Manipulation Language

* Insert: INSERT INTO 表名称 VALUES (值1, 值2,....)
* Delete: DELETE FROM 表名称 WHERE 列名称 = 值
* Update: UPDATE 表名称 SET 列名称 = 新值 WHERE 列名称 = 某值



# View in SQL

view只存储了定义，并没有真正存储表，Base table基表是真正存储在磁盘上的

* **General view普通视图：定义存储在数据库中，不存储表**

```sql
创建一个view视图
CREATE VIEW YoungSailor AS#创建了一张YoungSailor的视图，可以当成一张表来用
	SELECT sid,sname,rating
	FROM Sailors #Sailors是基表
	WHERE age<26;
当视图可以唯一对应基表的tuple时，才可以使用view update，否则无法使用
```



* **临时视图：定义也只是临时的，不存储在数据库中**

WITH语句创建的临时表就是临时视图

# Embedded SQL 嵌入式SQL

**在程序设计语言内访问数据库，需要解决以下问题**

* 如何让程序设计语言接受一条sql语句
* 如果让DBMS与应用程序交换数据
* 对于数据库系统进行操作，查询的结果是集合，集合如何传递给程序语言的变量
* 一个DBMS所支持的数据类型与程序设计语言的数据类型不完全一样

**为了解决这些问题，有以下几种解决方案：**

* embedded sql（是最基本的方式，其他的方法都是从此基础上发展出来的）
  * 最基本的方式，原理，嵌入式，预编译：此章介绍嵌入式sql，现在一般使用api或class libary
* Programming APIs
  * 数据库厂商制作的API
* Class libary 
  * 程序设计语言封装了对数据库访问的类

**usage of embedded SQL (in C)**

* sql语句直接的使用在C语言中

  * 所有嵌入在C语言中的SQL语句，全部以EXEC SQL开头，以';'结尾
  * 在语言中它是通过宿主变量的形式来在C代码与数据库之间来传递数据消息，宿主变量以EXEC SQL定义
  * 嵌入在C代码中的sql语句，可以用'':""来引用一个宿主变量的值
  * 在宿主语言中（比如C），宿主变量当普通变量来用
  * 宿主变量不能当作array或structure
  * 在嵌入式SQL里面，定义了一个特殊的宿主变量，叫作SQLCA，通过此来实现一些C程序与SQL之间交换信息，EXEC SQL INCLUDE SQLCA
  * 使用SQLCA.SQLCODE可以判断返回结果的状态
  * 在宿主语言中使用indicator(short int)表示NULL

  **例：如何定义宿主变量**

  ```c
  EXEC SQL BEGIN DECLARE SECTION;
  	char SNO[7];
  	char GIVENSNO[7];
  	char CNO[6];
  	char GIVENCNO[6];
  	float GRADE;
  	short GRADEI; /* indicator of GRADEI */
  EXEC SQL END DECLARE SECTION;
  
  ```

**executable statements可执行语句**

* Connect : 

  ```c
   EXEC SQL CONNECT:uid IDENTIFIED BY :pwd/*uid与pwd是2个宿主变量，*/
  ```

* Execute DDL or DML statements: 

```c
EXEC SQL INSERT INTO SC(SNO,CNO,GRADE)
		VALUES(:SNO,:CNO,:GRADE);

```

* Execute Query Statements:

```c
EXEC SQL SELECT GRADE
		INTO:GRADE,:GRADEI#返回值放回宿主变量中，如果为NULL，放到GRADEI这个indicator中
		FROM SC
		WHERE SNO=:GIVENSNO AND CNO=:GIVENCNO;#返回单个值,当返回值有多个tuple，不能如此操作，使用游标(cursor)操作
```

* Cursor

1. define a cursor

```c
EXEC SQL DECLARE <cursor name> CURSOR FOR
SELECT ...
FROM   ...
WHERE  ...
```

2. EXEC SQL OPEN<cursor name>

```c
EXEC SQL OPEN <cursor name> #some like open a file
```

3. fetch data from cursor

```c
EXEC SQL FETCH <cursor name>
		 INTO:hostvar1, :hostvar2, ...;
```

4. SQLCA.SQLCODE will return 100 when arriving the end of cursor

```sql
当SQLCA.SQLCODE返回100时，结果处理完了
```

5. CLOSE CURSOR <cursor name>

```sql
CLOSE CURSOR <cursor name>#释放资源
```

**Example of Query with Cursor**

```sql
......
EXEC SQL DECLARE C1 CURSOR FOR
	SELECT SNO,GRADE
	FROM SC
	WHERE CNO = :GIVENCNO;
EXEC SQL OPEN C1;
if (SQLCA.SQLCODE<0) exit(1); /*There is error in query*/
while(1){
	EXEC SQL FETCH C1 INTO :SNO,:GRADE,:GRADEI
	if (SQLCA.SQLCODE==100) break;
	 .
	 .#在此处理数据
	 .
}
EXEC SQL CLOSE C1;

```

#### Dynamic SQL动态SQL

当编译之前，需要的sql语句不知道，而是在使用中动态生成的，这就是dynamic sql

* Dynamic SQL executed directly直接可运行的动态sql

```SQL
在学生表中进行删除操作，请求操作者需要删除的条件
EXEC SQL BEGIN DECLARE SECTION;
char sqlstring[200];#定义一个宿主变量
EXEC SQL END DECLARE SECTION;
char cond[150];#定义一个普通数组
strcpy(sqlstring,"DELECE FROM STUDENT WHERE");
printf("Enter search conition:");#要求用户输入SELECT语句
scanf("%s",cond);#输入的值赋值级cond
strcat(sqlstring,cond);#拼接字符串
EXEC SQL EXECUTE IMMEDIATE:sqlstring;#立即执行
```

* Dynamic sql with dynamic parameters带参数的动态sql

```sql
EXEC SQL BEGIN DECLARE SECTION;
char sqlstring[200];
int birth_year;
EXEC SQL END DECLARE SECTION;
strcpy(sqlstring,"DELETE FROM STUDENT WHERE YEAR(BDATE)<=:y;");#前面并没有定义y，:y只是一个占位符
printf("Enter birth year for delete:")
scanf("%d",&birth_year)
EXECSQL PREPARE PURGE FROM:sqlstring;
EXECSQL EXECUTE PURGE USING:birth_year;
...
```

* Dynamic SQL for query

```sql
EXEC SQL BEGIN DECLARE SECTION;
char sqlstring[200];
char SNO[7];
float GRADE;
short GRADEI;
char GIVENCNO[6];
EXEC SQL END DECLARE SECTION;
CHAR ORDERBY[150];
STRCPY(sqlstrings,"SELECT SNO,GRADE FROM SC WHERE CNO=:c");
printf("Enter ORDER BY clause:");
scanf("%s",orderby);
strcat(sqlstring,orderby);
printf("Enter the course number");
scanf("%s", GIVENCNO);
EXEC SQL PREPARE query FROM :sqlstring;
ECEC SQL DECLARE grade_cursor CURSOR FOR query;
EXEC SQL OPEN grade_cursor USING:GIVENCNO;
```

#### Stored procedure 存储过程

可以将创建一个存储过程在数据库中，以后可以直接调用

**example of a stored procedure**

```sql
EXEC SQL
	CREATE PROCEDURE drop_student#创建一个存储过程名叫drop_student
		(IN student_no CHAR(7),#设置输入参数
         OUT message CHAR(30))#输出参数
     BEGIN ATOMIC#告诉系统作为原子操作，要么以下语句全部成功，要么全部不执行
     	DELETE FROM STUDENT
     			WHERE SNO=student_no;
     	DELETE FROM SC
     			WHERE SNO=student_no;
     	SET message=student_no || 'droped';
     END;
     
 EXEC SQL
 .
 .
 .
 CALL drop_student(...); /*class this stored procedure later*/
     
```



			





