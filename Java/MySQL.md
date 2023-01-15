# MySQL学习笔记

## 快查
<div align="center">
	<img src="https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/SQLV2Light1.jpg" width="80%">
</div>

## 待办
- ~~`P120` 关于B+树的全方位理解由来,二叉树,平衡二叉树~~..
- 联合索引`account,name,age`存在的情况下,是否有必要单独创建`account`索引
- `hash join`
- `FileSort算法`: 双路排序和单路排序


## 总览
- 基础
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/MySQL视频总览.png)

- 高级
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/Mysql高级.png)

> 持久化存储数据

- DB 数据库
- DBMS 数据库管理系统 (MySQL、Oracle、SQL server)
- SQL 结构化查询语言

> RDBMS 关系型数据库,二维表格(行列)形式存储,复杂查询,事务支持
> 非关系型数据库,键值存储/XML/JSON
> - 键值型,`redis`
> - 文档型,`MongoDB`
> - 搜索引擎型,`ES`,`Solr`
> - 列式数据库,`HBase`
> - 图形数据库(人物关系),`Neo4J`,`InfoGrid`


- 表关联关系
  - 一对一关联
  - 一对多关联
  - 多对多关联


## SQL

### SQL分类
- `DDL`(数据定义语言),操作数据库结构 CREATE,DROP,ALERT
- `DML`(数据操作语言),操作数据库记录 INSERT,DELETE,UPDATE,SELECT...
- `DCL`(数据控制),设置访问权限 GRANT,REVOKE,COMMIT,ROLLBACK,SAVEPOINT

> SQL规范,MySQL在Windows环境下大小写不敏感,在Linux环境下大小写敏感
> 所以建议统一大写或者统一小写


### 基本SELECT
- 别名 `AS`
- 去重 `DISTINCT`
- **所有运算符或列值遇到null值，运算的结果都为null**
- 空值不等于空字符串。一个空字符串的长度是 0，而一个空值的长度是空。而且，在 MySQL 里面，空值是占用空间的
- 着重号(标识保留字,关键字) (SELECT * FROM ORDER)错误  (SELECT * FROM \`ORDER\`)正确
- `from dual` 伪表查询


### 算术运算符
- `+`
- `-`
- `*`
- `/`或者`DIV` 默认结果浮点,`100/0=null`
- `%`或者`MOD`

```sql
# 101
select 100+'1' from dual;
# 100
select 100+'a' from dual;
# null
select 100+null from dual;

```


### 比较运算符

`=` `<=>` `<>` `!=` `<` `>` `<=` `>=`

- 字符串和数字比较时存在隐式转换,若转换不成功就当作0看待
- 字符串和字符串比较时按照 `ANSI` 规则比较
- 只要有 `NULL` 参与的判断 结果就是NULL
- `<=>` 安全等于,针对 `NULL` 的判断 
- 最小值运算符 语法格式为：`LEAST(值1，值2，...，值n)`
- 最大值运算符 语法格式为：`GREATEST(值1，值2，...，值n)`
- `BETWEEN AND` 运算符, 上界在前,下界在后,反之无结果

> `<=>` 安全等于可以用来对NULL进行判断。在两个操作数均为NULL时，其返回值为1,当一个操作数为NULL时，其返回值为0   
> **其他的比较运算符只要有NULL参与即为NULL**  

```sql
# 所有包含a 和e的
SELECT last_name FROM employees WHERE last_name LIKE '%a%' AND last_name LIKE '%e%'
SELECT last_name FROM employees WHERE last_name LIKE '%a%e%' OR last_name LIKE '%e%a%'

# 第二个字符是 a ,包含e的
SELECT last_name FROM employees WHERE last_name LIKE '_a%e%'
# 第二三个字符是 _a 的,转义字符
SELECT last_name FROM employees WHERE last_name LIKE '_\_a%'
# 最用同上,使用 $替代转义字符\
SELECT last_name FROM employees WHERE last_name LIKE '_$_a%' ESCAPE '$'
```

### 逻辑运算符
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/逻辑运算符.png)

> 逻辑异或（XOR）运算符是当给定的值中任意一个值为NULL时，则返回NULL；   
> 如果两个非NULL的值都是0或者都不等于0时，则返回0；如果一个值为0，另一个值不为0时，则返回1  


### 排序
> 可以使用不在SELECT列表中的列排序。   
> 在对多列进行排序的时候，首先排序的第一列必须有相同的列值，才会对第二列进行排序。如果第一列数据中所有值都是唯一的，将不再对第二列进行排序


### 分页

`LIMIT [位置偏移量,] 行数 `
`SELECT * FROM table LIMIT(PageNo - 1)*PageSize,PageSize`

> MySQL 8.0中可以使用`LIMIT 3 OFFSET 4`，意思是获取从第4条记录开始后面的3条记录，和`LIMIT4,3` 返回的结果相同

> 约束返回结果的数量可以 减少数据表的网络传输量 ，也可以 提升查询效率
> 如果我们知道返回结果只有1 条，就可以使用 LIMIT 1 ，告诉 SELECT 语句只需要返回一条记录即可。**这样的好处就是 SELECT 不需要扫描完整的表，只需要检索到一条符合条件的记录即可返回**

- SQL Server 和 Access 使用 TOP 关键字
  - `SELECT TOP 5 name, hp_max FROM heros ORDER BY hp_max DESC`
- DB2，使用 `FETCH FIRST 5 ROWS ONLY `
   -  `SELECT name, hp_max FROM heros ORDER BY hp_max DESC FETCH FIRST 5 ROWS ONLY`
- Oracle使用 `ROWNUM` 来统计
  - `SELECT rownum,last_name,salary FROM employees WHERE rownum < 5 ORDER BY salary DESC`


## 多表查询

### 等值连接,非等值连接

```sql
# 等值连接
SELECT employees.last_name, departments.department_name,employees.department_id FROM employees, departments WHERE employees.department_id = departments.department_id;

# 非等值连接
SELECT e.last_name,e.salary,j.grade_level FROM employees e,job_grades j WHERE e.salary BETWEEN j.lowest_sal AND j.highest_sal;
```

### 自连接,非自连接

```sql
# 自连接
SELECT emp.employee_id,emp.last_name,mgr.employee_id,mgr.last_name FROM employees emp,employees mgr
WHERE emp.manager_id = mgr.employee_id

```

### 内连接(两表根据条件的交集)


### 外连接(并集)
- 左外`LEFT OUTER JOIN` `LEFT JOIN`
- 右外`RIGHT OUTER JOIN` `RIGHT JOIN`
- 全外**MySQL不支持FULL JOIN，但是可以用 LEFT JOIN UNION RIGHT JOIN 代替**

### 使用(+)创建连接,
- `SQL92` 中采用`(+)`代表`从表`所在的位置。即左或右外连接中，`(+)` 表示哪个是从表

```sql 
# 全外连接,或者使用 上面的LEFT JOIN UNION RIGHT JOIN
select * from emp,dept where epm.deptno(+) = dept.dpetno(+)

# 左外
select * from emp,dept where epm.deptno = dept.dpetno(+)

# 右外
select * from emp,dept where epm.deptno(+) = dept.dpetno

```

### 7种SQL JOIN 实现
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/sql-join.png)


### SQL99新特性

- `SQL99` 自然连接 `NATURAL JOIN`
- `SQL99` USING 指定数据表里的**同名字段**进行等值连接。但是只能配合JOIN一起使用

```sql
# sql92
SELECT employee_id,last_name,department_name
FROM employees e JOIN departments d
ON e.`department_id` = d.`department_id`
AND e.`manager_id` = d.`manager_id`;

# 等效的SQL99自然连接
SELECT employee_id,last_name,department_name
FROM employees e NATURAL JOIN departments d;

# USING SQL99 连接
SELECT employee_id,last_name,department_name
FROM employees e JOIN departments d
USING (department_id);

```

##　函数
> MySQL提供的内置函数从 实现的功能角度 可以分为`数值函数`、`字符串函数`、`日期和时间函数`、`流程控制函数`、`加密与解密函数`、`获取MySQL信息函数`、`聚合函数`等     
> 内置函数分为两类： 单行函数 、 聚合函数（或分组函数）
### 单行函数

#### 基本函数

<!-- ![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/mysql基本函数.png) -->

| 函数                | 用法                                                         |
| ------------------- | ------------------------------------------------------------ |
| ABS(x)              | 返回x的绝对值                                                |
| SIGN(X)             | 返回X的符号。正数返回1，负数返回-1，0返回0                   |
| PI()                | 返回圆周率的值                                               |
| CEIL(x)，CEILING(x) | 返回大于或等于某个值的最小整数                               |
| FLOOR(x)            | 返回小于或等于某个值的最大整数                               |
| LEAST(e1,e2,e3…)    | 返回列表中的最小值                                           |
| GREATEST(e1,e2,e3…) | 返回列表中的最大值                                           |
| MOD(x,y)            | 返回X除以Y后的余数                                           |
| RAND()              | 返回0~1的随机值                                              |
| RAND(x)             | 返回0~1的随机值，其中x的值用作种子值，相同的X值会产生相同的随机数 |
| ROUND(x)            | 返回一个对x的值进行四舍五入后，最接近于X的整数               |
| ROUND(x,y)          | 返回一个对x的值进行四舍五入后最接近X的值，并保留到小数点后面Y位 |
| TRUNCATE(x,y)       | 返回数字x截断为y位小数的结果                                 |
| SQRT(x)             | 返回x的平方根。当X的值为负数时，返回NULL                     |


#### 字符串函数
> 注意：MySQL中，字符串的位置是从1开始的。

| 函数                              | 用法                                                         |
| --------------------------------- | ------------------------------------------------------------ |
| ASCII(S)                          | 返回字符串S中的第一个字符的ASCII码值                         |
| CHAR_LENGTH(s)                    | 返回字符串s的字符数。作用与CHARACTER_LENGTH(s)相同           |
| LENGTH(s)                         | 返回字符串s的字节数，和字符集有关                            |
| CONCAT(s1,s2,......,sn)           | 连接s1,s2,......,sn为一个字符串                              |
| CONCAT_WS(x, s1,s2,......,sn)     | 同CONCAT(s1,s2,...)函数，但是每个字符串之间要加上x           |
| INSERT(str, idx, len, replacestr) | 将字符串str从第idx位置开始，len个字符长的子串替换为字符串replacestr |
| REPLACE(str, a, b)                | 用字符串b替换字符串str中所有出现的字符串a                    |
| UPPER(s) 或 UCASE(s)              | 将字符串s的所有字母转成大写字母                              |
| LOWER(s)  或LCASE(s)              | 将字符串s的所有字母转成小写字母                              |
| LEFT(str,n)                       | 返回字符串str最左边的n个字符                                 |
| RIGHT(str,n)                      | 返回字符串str最右边的n个字符                                 |
| LPAD(str, len, pad)               | 用字符串pad对str最左边进行填充，直到str的长度为len个字符     |
| RPAD(str ,len, pad)               | 用字符串pad对str最右边进行填充，直到str的长度为len个字符     |
| LTRIM(s)                          | 去掉字符串s左侧的空格                                        |
| RTRIM(s)                          | 去掉字符串s右侧的空格                                        |
| TRIM(s)                           | 去掉字符串s开始与结尾的空格                                  |
| TRIM(s1 FROM s)                   | 去掉字符串s开始与结尾的s1                                    |
| TRIM(LEADING s1 FROM s)           | 去掉字符串s开始处的s1                                        |
| TRIM(TRAILING s1 FROM s)          | 去掉字符串s结尾处的s1                                        |
| REPEAT(str, n)                    | 返回str重复n次的结果                                         |
| SPACE(n)                          | 返回n个空格                                                  |
| STRCMP(s1,s2)                     | 比较字符串s1,s2的ASCII码值的大小                             |
| SUBSTR(s,index,len)               | 返回从字符串s的index位置其len个字符，作用与SUBSTRING(s,n,len)、MID(s,n,len)相同 |
| LOCATE(substr,str)                | 返回字符串substr在字符串str中首次出现的位置，作用于POSITION(substr IN str)、INSTR(str,substr)相同。未找到，返回0 |
| ELT(m,s1,s2,…,sn)                 | 返回指定位置的字符串，如果m=1，则返回s1，如果m=2，则返回s2，如果m=n，则返回sn |
| FIELD(s,s1,s2,…,sn)               | 返回字符串s在字符串列表中第一次出现的位置                    |
| FIND_IN_SET(s1,s2)                | 返回字符串s1在字符串s2中出现的位置。其中，字符串s2是一个以逗号分隔的字符串 |
| REVERSE(s)                        | 返回s反转后的字符串                                          |
| NULLIF(value1,value2)             | 比较两个字符串，如果value1与value2相等，则返回NULL，否则返回value1 |



#### 时间函数  

| `获取日期、时间`函数                                 | 用法                           |
| ------------------------------------------------------------ | ------------------------------ |
| **CURDATE()** ，CURRENT_DATE()                               | 返回当前日期，只包含年、月、日 |
| **CURTIME()** ， CURRENT_TIME()                              | 返回当前时间，只包含时、分、秒 |
| **NOW()** / SYSDATE() / CURRENT_TIMESTAMP() / LOCALTIME() / LOCALTIMESTAMP() | 返回当前系统日期和时间         |
| UTC_DATE()                                                   | 返回UTC（世界标准时间）日期    |
| UTC_TIME()                                                   | 返回UTC（世界标准时间）时间    |

| `日期与时间戳的转换` 函数    | 用法                                                         |
| ------------------------ | ------------------------------------------------------------ |
| UNIX_TIMESTAMP()         | 以UNIX时间戳的形式返回当前时间。SELECT UNIX_TIMESTAMP() ->1634348884 |
| UNIX_TIMESTAMP(date)     | 将时间date以UNIX时间戳的形式返回。                           |
| FROM_UNIXTIME(timestamp) | 将UNIX时间戳的时间转换为普通格式的时间                       |


| `年 月 日 星期` 函数                     | 用法                                            |
| ---------------------------------------- | ----------------------------------------------- |
| YEAR(date) / MONTH(date) / DAY(date)     | 返回具体的日期值                                |
| HOUR(time) / MINUTE(time) / SECOND(time) | 返回具体的时间值                                |
| MONTHNAME(date)                          | 返回月份：January，...                          |
| DAYNAME(date)                            | 返回星期几：MONDAY，TUESDAY.....SUNDAY          |
| WEEKDAY(date)                            | 返回周几，注意，周1是0，周2是1，。。。周日是6   |
| QUARTER(date)                            | 返回日期对应的季度，范围为1～4                  |
| WEEK(date) ， WEEKOFYEAR(date)           | 返回一年中的第几周                              |
| DAYOFYEAR(date)                          | 返回日期是一年中的第几天                        |
| DAYOFMONTH(date)                         | 返回日期位于所在月份的第几天                    |
| DAYOFWEEK(date)                          | 返回周几，注意：周日是1，周一是2，。。。周六是7 |

| `日期的操作`函数         | 用法                                       |
| ----------------------- | ------------------------------------------ |
| EXTRACT(type FROM date) | 返回指定日期中特定的部分，type指定返回的值 |


```sql
# 161109
select EXTRACT(HOUR_SECOND FROM NOW())
# 202209
select EXTRACT(YEAR_MONTH FROM NOW())
```

| `时间和秒钟转换`函数                 | 用法                                                         |
| -------------------- | ------------------------------------------------------------ |
| TIME_TO_SEC(time)    | 将 time 转化为秒并返回结果值。转化的公式为：`小时*3600+分钟*60+秒` |
| SEC_TO_TIME(seconds) | 将 seconds 描述转化为包含小时、分钟和秒的时间                |

```sql
SELECT TIME_TO_SEC('2021-10-21 23:32:12');
# 23*3600+32*60+12
```


| 函数                                                         | 用法                                           |
| ------------------------------------------------------------ | ---------------------------------------------- |
| DATE_ADD(datetime, INTERVAL  expr type)，ADDDATE(date,INTERVAL expr type) | 返回与给定日期时间相差INTERVAL时间段的日期时间 |
| DATE_SUB(date,INTERVAL expr type)，SUBDATE(date,INTERVAL expr type) | 返回与date相差INTERVAL时间间隔的日期           |

```sql
SELECT DATE_ADD(NOW(), INTERVAL 1 DAY) AS col1, -- 加一天
DATE_ADD('2021-10-21 23:32:12',INTERVAL 1 SECOND) AS col2,-- 加一秒
ADDDATE('2021-10-21 23:32:12',INTERVAL 1 SECOND) AS col3, -- 加一秒
DATE_ADD('2021-10-21 23:32:12',INTERVAL '1_1' MINUTE_SECOND) AS col4,  -- 加一分钟一秒 2021-10-21 23:33:13
DATE_ADD(NOW(), INTERVAL -1 YEAR) AS col5, -- 可以是负数 减去一年
DATE_ADD(NOW(), INTERVAL '1_1' YEAR_MONTH) AS col6 -- 需要单引号 加一年一月
FROM DUAL;
```

| 函数                         | 用法                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| ADDTIME(time1,time2)         | 返回time1加上time2的时间。当time2为一个数字时，代表的是`秒`，可以为负数 |
| SUBTIME(time1,time2)         | 返回time1减去time2后的时间。当time2为一个数字时，代表的是`秒`，可以为负数 |
| DATEDIFF(date1,date2)        | 返回date1 - date2的日期间隔天数                              |
| TIMEDIFF(time1, time2)       | 返回time1 - time2的时间间隔                                  |
| FROM_DAYS(N)                 | 返回从0000年1月1日起，N天以后的日期                          |
| TO_DAYS(date)                | 返回日期date距离0000年1月1日的天数                           |
| LAST_DAY(date)               | 返回date所在月份的最后一天的日期                             |
| MAKEDATE(year,n)             | 针对给定年份与所在年份中的天数返回一个日期                   |
| MAKETIME(hour,minute,second) | 将给定的小时、分钟和秒组合成时间并返回                       |
| PERIOD_ADD(time,n)           | 返回time加上n后的时间                                        |



|`日期格式化` 函数                              | 用法                                       |
| --------------------------------- | ------------------------------------------ |
| DATE_FORMAT(date,fmt)             | 按照字符串fmt格式化日期date值              |
| TIME_FORMAT(time,fmt)             | 按照字符串fmt格式化时间time值              |
| GET_FORMAT(date_type,format_type) | 返回日期字符串的显示格式                   |
| STR_TO_DATE(str, fmt)             | 按照字符串fmt对str进行解析，解析为一个日期 |

上述`非GET_FORMAT`函数中fmt参数常用的格式符：

| 格式符 | 说明                                                        | 格式符 | 说明                                                        |
| ------ | ----------------------------------------------------------- | ------ | ----------------------------------------------------------- |
| %Y     | 4位数字表示年份                                             | %y     | 表示两位数字表示年份                                        |
| %M     | 月名表示月份（January,....）                                | %m     | 两位数字表示月份（01,02,03。。。）                          |
| %b     | 缩写的月名（Jan.，Feb.，....）                              | %c     | 数字表示月份（1,2,3,...）                                   |
| %D     | 英文后缀表示月中的天数（1st,2nd,3rd,...）                   | %d     | 两位数字表示月中的天数(01,02...)                            |
| %e     | 数字形式表示月中的天数（1,2,3,4,5.....）                    |        |                                                             |
| %H     | 两位数字表示小数，24小时制（01,02..）                       | %h和%I | 两位数字表示小时，12小时制（01,02..）                       |
| %k     | 数字形式的小时，24小时制(1,2,3)                             | %l     | 数字形式表示小时，12小时制（1,2,3,4....）                   |
| %i     | 两位数字表示分钟（00,01,02）                                | %S和%s | 两位数字表示秒(00,01,02...)                                 |
| %W     | 一周中的星期名称（Sunday...）                               | %a     | 一周中的星期缩写（Sun.，Mon.,Tues.，..）                    |
| %w     | 以数字表示周中的天数(0=Sunday,1=Monday....)                 |        |                                                             |
| %j     | 以3位数字表示年中的天数(001,002...)                         | %U     | 以数字表示年中的第几周，（1,2,3。。）其中Sunday为周中第一天 |
| %u     | 以数字表示年中的第几周，（1,2,3。。）其中Monday为周中第一天 |        |                                                             |
| %T     | 24小时制                                                    | %r     | 12小时制                                                    |
| %p     | AM或PM                                                      | %%     | 表示%                                                       |


`GET_FORMAT`函数中fmt参数常用的格式符,**返回的是格式**  

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/格式化日期GET_FORMAT.png)

```sql
SELECT DATE_FORMAT(NOW(), '%H:%i:%s'); # 22:57:34

--- 格式化日期参考
SELECT STR_TO_DATE('09/01/2009','%m/%d/%Y')
FROM DUAL;

SELECT STR_TO_DATE('20140422154706','%Y%m%d%H%i%s')
FROM DUAL;

SELECT GET_FORMAT(DATE, 'USA'); # %m.%d.%Y

SELECT DATE_FORMAT(NOW(),GET_FORMAT(DATE,'USA')) FROM DUAL; # 09.27.2022

--字符串解析为日期 可以自己拼接格式
SELECT STR_TO_DATE('2020-01-01 00:00:00','%Y-%m-%d');

```

#### 流程处理函数

流程处理函数可以根据不同的条件，执行不同的处理流程，可以在SQL语句中实现不同的条件选择。MySQL中的流程处理函数主要包括IF()、IFNULL()和CASE()函数。

| 函数                                                         | 用法                                            |
| ------------------------------------------------------------ | ----------------------------------------------- |
| IF(value,value1,value2)                                      | 如果value的值为TRUE，返回value1，否则返回value2 |
| IFNULL(value1, value2)                                       | 如果value1不为NULL，返回value1，否则返回value2  |
| CASE WHEN 条件1 THEN 结果1 WHEN 条件2 THEN 结果2 .... [ELSE resultn] END | 相当于Java的if...else if...else...              |
| CASE  expr WHEN 常量值1 THEN 值1 WHEN 常量值1 THEN 值1 .... [ELSE 值n] END | 相当于Java的switch...case...                    |



#### 加密与解密函数

加密与解密函数主要用于对数据库中的数据进行加密和解密处理，以防止数据被他人窃取。这些函数在保证数据库安全时非常有用。

| 函数                        | 用法                                                         |
| --------------------------- | ------------------------------------------------------------ |
| PASSWORD(str)               | 返回字符串str的加密版本，41位长的字符串。加密结果`不可逆`，常用于用户的密码加密 |
| MD5(str)                    | 返回字符串str的md5加密后的值，也是一种加密方式。若参数为NULL，则会返回NULL |
| SHA(str)                    | 从原明文密码str计算并返回加密后的密码字符串，当参数为NULL时，返回NULL。`SHA加密算法比MD5更加安全`。 |
| ENCODE(value,password_seed) | 返回使用password_seed作为加密密码加密value                   |
| DECODE(value,password_seed) | 返回使用password_seed作为加密密码解密value                   |

####  MySQL信息函数

MySQL中内置了一些可以查询MySQL信息的函数，这些函数主要用于帮助数据库开发或运维人员更好地对数据库进行维护工作。

| 函数                                                  | 用法                                                     |
| ----------------------------------------------------- | -------------------------------------------------------- |
| VERSION()                                             | 返回当前MySQL的版本号                                    |
| CONNECTION_ID()                                       | 返回当前MySQL服务器的连接数                              |
| DATABASE()，SCHEMA()                                  | 返回MySQL命令行当前所在的数据库                          |
| USER()，CURRENT_USER()、SYSTEM_USER()，SESSION_USER() | 返回当前连接MySQL的用户名，返回结果格式为“主机名@用户名” |
| CHARSET(value)                                        | 返回字符串value自变量的字符集                            |
| COLLATION(value)                                      | 返回字符串value的比较规则                                |

#### 其他函数

MySQL中有些函数无法对其进行具体的分类，但是这些函数在MySQL的开发和运维过程中也是不容忽视的。

| 函数                           | 用法                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| FORMAT(value,n)                | 返回对数字value进行格式化后的结果数据。n表示`四舍五入`后保留到小数点后n位,如果n的值小于或者等于0，则只保留整数部分 |
| CONV(value,from,to)            | 将value的值进行不同进制之间的转换                            |
| INET_ATON(ipvalue)             | 将以点分隔的IP地址转化为一个数字                             |
| INET_NTOA(value)               | 将数字形式的IP地址转化为以点分隔的IP地址                     |
| BENCHMARK(n,expr)              | 将表达式expr重复执行n次。用于测试MySQL处理expr表达式所耗费的时间 |
| CONVERT(value USING char_code) | 将value所使用的字符编码修改为char_code                       |


### 多行(聚合)函数
- `AVG`
- `SUM`
- `MAX,MIN` 适用于,数值,字符串,日期时间类型
- `COUNT`

> `AVG`等函数会自动**忽略null值**,在统计所有时要注意 `SELECT AVG(IFNUll(commission_pct,0))`     
> `AVG` `SUM` 面对字符串返回值为零    


> `COUNT()`
> 用 count(*),count(1),count(列名)谁好呢?     
> 对于`MyISAM引擎`的表是没有区别的。这种引擎内部有一计数器在维护着行数。      
> Innodb引擎的表用count(*),count(1)直接读行数，复杂度是O(n)，但好于具体的count(列名)     
> 不要使用 `count(列名)`来替代 `count(*)`,`count(*)` 是 SQL92 定义的标准统计行数的语法，跟数据库无关，跟 NULL 和非 NULL 无关。    
> count(*)会统计值为 NULL 的行，而 **count(列名)不会统计此列为 NULL 值的行**


> `GROUP BY`
> 在SELECT列表中所有未包含在组函数中的列都应该包含在 `GROUP BY`子句中,包含在`GROUP BY` 子句中的列不必包含在SELECT 列表中   
> 即：**select 中的直接列都应该在group by中存在**   

```sql
--需求：查询各个department_id,job_id的平均工资
SELECT department_id,job_id,AVG(salary)
FROM employees
GROUP BY  department_id,job_id;

--错误的 department_id,job_id 必须放在group by中
SELECT department_id,job_id,AVG(salary)
FROM employees
GROUP BY department_id;
```

> `GROUP BY` 中使用 `WITH ROLLUP` 汇总列,应用对应函数,`AVG` 会生成 所有列的平均值,`SUM` 生成所有列之和...
> 当使用ROLLUP时，不能同时使用ORDER BY子句进行结果排序，即ROLLUP和ORDER BY是互相排斥的 `实测MySQL8可以`

- `HAVING` 过滤数据,结合`GROUP BY`
  - 行为已经被分组
  - 使用了聚合函数
  - 最好和GROUP BY一起使用,并声明在GROUP BY 的后面

```sql
-- 错误示例 查找工资大于10000 的部门 会报错
SELECT department_id,MAX(salary)
FROM employees
WHERE MAX(salary )> 10000
GROUP BY department_id

-- 修改版
SELECT department_id,MAX(salary)
FROM employees
GROUP BY department_id
HAVING MAX(salary )> 10000

-- 附带条件 ,可以使用WHERE
SELECT department_id,MAX(salary)
FROM employees
GROUP BY department_id
HAVING MAX(salary )> 10000 and department_id in (10,20,30,40)
```
> **WHERE和HAVING**
> 过滤条件中有聚合函数时,必须使用HAVING  
> 没有聚合函数最好使用WHERE  
> 非聚合函数情况下, `WHERE` 的效率是高于 `HAVING` 的,WHERE 是先筛选后连接，而 HAVING 是先连接后筛选  

> 关键字顺序 `SELECT ... FROM ... WHERE ... GROUP BY ... HAVING ... ORDER BY ... LIMIT..`      
> SELECT 语句的执行顺序 `FROM -> WHERE -> GROUP BY -> HAVING -> SELECT 的字段 -> DISTINCT -> ORDER BY -> LIMIT`   

## 子查询
> 子查询要包含在括号内   
> 将子查询放在比较条件的右侧   
> 单行操作符对应单行子查询,      
> 多行操作符对应多行子查询,返回多行`IN` `ANY` `ALL` `SOME`  

| 单行子查询操作符 | 含义                     |
| ------ | ------------------------ |
| =      | equal to                 |
| >      | greater than             |
| >=     | greater than or equal to |
| <      | less than                |
| <=     | less than or equal to    |
| <>     | not equal to             |

| 多行子查询操作符 | 含义                                                         |
| ------ | ------------------------------------------------------------ |
| IN     | 等于列表中的**任意一个**                                     |
| ANY    | 需要和单行比较操作符一起使用，和子查询返回的**某一个**值比较 |
| ALL    | 需要和单行比较操作符一起使用，和子查询返回的**所有**值比较   |
| SOME   | 实际上是ANY的别名，作用相同，一般常使用ANY                   |

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/mysql子查询.png)


```sql
-- 单行示例 返回job_id与141号员工相同，salary比143号员工多的员工姓名，job_id和工资
SELECT last_name, job_id, salary
FROM   employees
WHERE  job_id =  
                (SELECT job_id
                 FROM   employees
                 WHERE  employee_id = 141)
AND    salary >
                (SELECT salary
                 FROM   employees
                 WHERE  employee_id = 143);
```

```sql

-- 多行示例 查询平均工资最低的部门id
SELECT
	department_id
FROM
	employees
GROUP BY
	department_id
HAVING
	AVG( salary ) = ( SELECT MIN( avg_sal ) FROM ( SELECT AVG( salary ) avg_sal FROM employees GROUP BY department_id ) dept_avg_sal )
```

### 相关子查询

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/mysql相关子查询.png)

```sql
-- 使用相关子查询批量更新
UPDATE employees e
SET department_name = (SELECT department_name
           FROM  departments d
           WHERE e.department_id = d.department_id);

-- 查询员工的id,salary,按照department_name 排序
SELECT employee_id,salary
FROM employees e
ORDER BY (
 SELECT department_name
 FROM departments d
 WHERE e.`department_id` = d.`department_id`
);
```

## 数据库的创建、修改、删除

### 创建

```sql
CREATE DATABASE 数据库名;

-- MySQL 8默认为utf-8  7默认为gbk
CREATE DATABASE 数据库名 CHARACTER SET 字符集;
CREATE DATABASE mytest2 CHARACTER SET 'gbk';

CREATE DATABASE IF NOT EXISTS 数据库名;
CREATE DATABASE IF NOT EXISTS mytest3 CHARACTER SET 'utf8';
```

> DATABASE 不能改名。一些可视化工具可以改名，它是建新库，把所有表复制到新库，再删旧库完成的

- 使用

```sql
-- 有一个S，代表多个数据库
SHOW DATABASES;

-- 使用的一个 mysql 中的全局函数
SELECT DATABASE();

SHOW TABLES FROM 数据库名;

-- 更改数据库字符集
ALTER DATABASE 数据库名 CHARACTER SET 字符集;
ALTER DATABASE mytest2 CHARACTER SET 'utf8';

-- 创建表结构 
CREATE TABLE dept(
  -- int类型，自增
deptno INT(2) AUTO_INCREMENT,
dname VARCHAR(14),
loc VARCHAR(13),
salary DOUBLE,
 -- 日期类型
birthday DATE,
  -- 主键
  PRIMARY KEY (deptno)
);

--根据已有表创建表
CREATE TABLE emp1 AS SELECT * FROM employees; -- 创建的emp1和employees 结构数据相同 

CREATE TABLE emp2 AS SELECT * FROM employees WHERE 1=2; -- 创建的emp2是空表  

-- 带条件创建
CREATE TABLE dept80
AS 
SELECT  employee_id, last_name, salary*12 ANNSAL, hire_date
FROM    employees
WHERE   department_id = 80;

-- dept80 表结构 
CREATE TABLE `dept80` (
  `employee_id` int(11) NOT NULL DEFAULT '0',
  `last_name` varchar(25) CHARACTER SET utf8 NOT NULL,
  `ANNSAL` double(19,2) DEFAULT NULL,
  `hire_date` date NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;  

```


###  修改
#### 追加列 
`ALTER TABLE 表名 ADD 【COLUMN】 字段名 字段类型 【FIRST|AFTER 字段名】` 

```sql
-- 默认添加到表中的最后一个字段的位置
ALTER TABLE myemp1
ADD salary DOUBLE(10,2); 

-- 添加到第一个位置
ALTER TABLE myemp1
ADD phone_number VARCHAR(20) FIRST;

-- 加到某个字段后
ALTER TABLE myemp1
ADD email VARCHAR(45) AFTER emp_name;

```

#### 修改列 
可以修改数据类型，长度、默认值和位置   
`ALTER TABLE 表名 MODIFY 【COLUMN】 字段名1 字段类型 【DEFAULT 默认值】【FIRST|AFTER 字段名2】`

```sql
ALTER TABLE	dept80
MODIFY last_name VARCHAR(30); 

ALTER TABLE	dept80
MODIFY salary double(9,2) default 1000;
```

#### 重命名一个列
`ALTER TABLE 表名 CHANGE 【column】 列名 新列名 新数据类型;`

```sql
-- 将department_name 改为 dept_name varchar(15)
ALTER TABLE  dept80
CHANGE department_name dept_name varchar(15); 
```

#### 删除列
`ALTER TABLE 表名 DROP 【COLUMN】字段名`

```sql
ALTER TABLE  dept80
DROP COLUMN  job_id; 
```

### 重命名表

```sql
-- RENAME
RENAME TABLE emp
TO myemp;
```

###  删除

```sql
DROP DATABASE 数据库名;
DROP DATABASE IF EXISTS 数据库名;

-- 删除所有数据
DELETE FROM emp2;
-- 回滚,可以恢复删除的数据
ROLLBACK;

-- 不可回滚删除
TRUNCATE TABLE detail_dept;
```
> `TRUNCATE TABLE`语句会删除表中所有的数据,释放表的存储空间,**不能回滚**  


### MySQL8新特性—DDL的原子化

> 在MySQL 8.0版本中，InnoDB表的DDL支持事务完整性，即 DDL操作要么成功要么回滚 。DDL操作回滚日志写入到data dictionary数据字典表mysql.innodb_ddl_log（该表是隐藏的表，通过show tables无法看到）中，用于回滚操作

- 执行命令 删除两个表 (book1,book2),book2存在book1 不存在

```bash
# 5.7下删除报错,但是会删除book2
# 若是8.0 删除报错,但是不会删除book2,操作报错会自动回滚
mysql> DROP TABLE book1,book2;
ERROR 1051 (42S02): Unknown table 'mytest.book2'
```


## 数据的CURD

### 插入　

```sql
-- 按表所有列的默认顺序插入 
INSERT INTO 表名
VALUES (value1,value2,....);

-- 按照指定列插入
INSERT INTO 表名(column1 [, column2, …, columnn])
VALUES (value1 [,value2, …, valuen]);

-- 批量插入
INSERT INTO table_name
VALUES
(value1 [,value2, …, valuen]),
(value1 [,value2, …, valuen]),
……
(value1 [,value2, …, valuen]);

-- 查询和添加 将查询结果插入到表中
INSERT INTO 目标表名
(tar_column1 [, tar_column2, …, tar_columnn])
SELECT
(src_column1 [, src_column2, …, src_columnn])
FROM 源表名
[WHERE condition]
```

### 更新

```sql

UPDATE copy_emp
SET  department_id = 110;

-- 更新多列
UPDATE table_name
SET column1=value1, column2=value2, … , column=valuen
[WHERE condition]

```

### 删除

```sql
DELETE FROM table_name [WHERE <condition>];

-- 全部删除
DELETE FROM  copy_emp;
```

### MySQL新特性:计算列

> 某一列的值是通过别的列计算得来的。例如，a列值为1、b列值为2，c列不需要手动插入，定义a+b的结果为c的值，那么c就是计算列，是通过别的列计算得来的    
> 新增的a b 就会自动生成c,修改a b的值也会更新c

```sql
CREATE TABLE tb1(
id INT,
a INT,
b INT,
c INT GENERATED ALWAYS AS (a + b) VIRTUAL
);
```

## 数据类型
| 类型             | 类型举例                                                     |
| ---------------- | ------------------------------------------------------------ |
| 整数类型         | TINYINT、SMALLINT、MEDIUMINT、INT(或INTEGER)、BIGINT         |
| 浮点类型         | FLOAT、DOUBLE                                                |
| 定点数类型       | DECIMAL                                                      |
| 位类型           | BIT                                                          |
| 日期时间类型     | YEAR、TIME、DATE、DATETIME、TIMESTAMP                        |
| 文本字符串类型   | CHAR、VARCHAR、TINYTEXT、TEXT、MEDIUMTEXT、LONGTEXT          |
| 枚举类型         | ENUM                                                         |
| 集合类型         | SET                                                          |
| 二进制字符串类型 | BINARY、VARBINARY、TINYBLOB、BLOB、MEDIUMBLOB、LONGBLOB      |
| JSON类型         | JSON对象、JSON数组                                           |
| 空间数据类型     | 单值类型：GEOMETRY、POINT、LINESTRING、POLYGON；<br/>集合类型：MULTIPOINT、MULTILINESTRING、MULTIPOLYGON、GEOMETRYCOLLECTION |

常见数据类型的属性，如下：

| MySQL关键字        | 含义                     |
| ------------------ | ------------------------ |
| NULL               | 数据列可包含NULL值       |
| NOT NULL           | 数据列不允许包含NULL值   |
| DEFAULT            | 默认值                   |
| PRIMARY KEY        | 主键                     |
| AUTO_INCREMENT     | 自动递增，适用于整数类型 |
| UNSIGNED           | 无符号                   |
| CHARACTER SET name | 指定一个字符集           |

### 整型
| **整数类型** | **字节** | 有符号数取值范围                         | 无符号数取值范围       |
| ------------ | -------- | ---------------------------------------- | ---------------------- |
| TINYINT      | 1        | -128~127                                 | 0~255                  |
| SMALLINT     | 2        | -32768~32767                             | 0~65535                |
| MEDIUMINT    | 3        | -8388608~8388607                         | 0~16777215             |
| INT、INTEGER | 4        | -2147483648~2147483647                   | 0~4294967295           |
| BIGINT       | 8        | -9223372036854775808~9223372036854775807 | 0~18446744073709551615 |



> int(M)中 `M` 表示可选属性(显示宽度),M 的取值范围是(0,255),例如 int(5),当数据宽度小于5位的时候在数字前面需要用字符填满宽度。该项功能需要配合`ZEROFILL`使用   
> `UNSIGNED` 无符号类型（非负），所有的整数类型都有一个可选的属性`UNSIGNED`（无符号属性），无符号整数类型的最小取值为0,int类型默认显示宽度为int(11)，无符号int类型默认显示宽度为int(10)     
> `ZEROFILL` 0填充,（如果某列是ZEROFILL，那么MySQL会自动为当前列添加`UNSIGNED`属性），如果指定了`ZEROFILL`只是表示不够M位时，用0在左边填充    
> **注意:** M 的值跟 int(M)所占多少存储空间并无关系, int(3)、int(4)、int(8) 在磁盘上都是占用 4 bytes 的存储空间   
> **int(M)，必须和UNSIGNED ZEROFILL一起使用才有意义**,如果整数值超过M位，就按照实际位数存储 `显示宽度与类型可以存储的值范围无关`。只是无须再用字符 0 进行填充     
> **从MySQL 8.0.17开始，整数数据类型不推荐使用显示宽度属性。**

### 浮点

- 单精度 FLOAT
- 双精度 DOUBLE

```sql
CREATE TABLE test_double1(
f1 FLOAT,
f2 FLOAT(5,2),
f3 DOUBLE,
f4 DOUBLE(5,2)
);
```
> `FLOAT(M,D)` 或 `DOUBLE(M,D)` 。这里，`M`称为 `精度` ，`D`称为 `标度` 。(M,D)中 M=整数位+小数位，D=小数位。 D<=M<=255，0<=D<=30
> 定义为 `FLOAT(5,2)`的一个列可以显示为`-999.99-999.99` 5位2位小数。如果超过这个范围会报错

> 从MySQL 8.0.17开始，FLOAT(M,D) 和DOUBLE(M,D)用法在官方文档中已经明确不推荐使用，将来可能被移除。另外，关于浮点型FLOAT和DOUBLE的UNSIGNED也不推荐使用(不影响分配空间)，将来也可能被移除

> 不推荐使用浮点,浮点数是不准确的,避免使用 `=` 来判断两个浮点数是否相等

### 定点数 DECIMAL

| 数据类型                 | 字节数  | 含义               |
| ------------------------ | ------- | ------------------ |
| DECIMAL(M,D),DEC,NUMERIC | M+2字节 | 有效范围由M和D决定 |


### 位类型 BIT

BIT类型中存储的是二进制值，类似010110。

| 二进制字符串类型 | 长度 | 长度范围     | 占用空间            |
| ---------------- | ---- | ------------ | ------------------- |
| BIT(M)           | M    | 1 <= M <= 64 | 约为(M + 7)/8个字节 |

```sql
CREATE TABLE test_bit1(
f1 BIT,
f2 BIT(5), # 5位
f3 BIT(64) # 64位
);
```

> BIT类型，如果没有指定(M)，默认是1位。这个1位，表示只能存1位的二进制值。这里(M)是表示二进制的位数，位数最小值为1，最大值为64    
> 使用 select 查询结果时,结果使用16进制展示

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/MySQLbit1.png)

### 时间日期类型
- `YEAR`类型通常用来表示年
- `DATE`类型通常用来表示年、月、日
- `TIME`类型通常用来表示时、分、秒
- `DATETIME`类型通常用来表示年、月、日、时、分、秒
- `TIMESTAMP`类型通常用来表示带时区的年、月、日、时、分、秒

| 类型      | 名称     | 字节 | 日期格式            | 最小值                  | 最大值                 |
| --------- | -------- | ---- | ------------------- | ----------------------- | ---------------------- |
| YEAR      | 年       | 1    | YYYY或YY            | 1901                    | 2155                   |
| TIME      | 时间     | 3    | HH:MM:SS            | -838:59:59              | 838:59:59              |
| DATE      | 日期     | 3    | YYYY-MM-DD          | 1000-01-01              | 9999-12-03             |
| DATETIME  | 日期时间 | 8    | YYYY-MM-DD HH:MM:SS | 1000-01-01 00:00:00     | 9999-12-31 23:59:59    |
| TIMESTAMP | 日期时间 | 4    | YYYY-MM-DD HH:MM:SS | 1970-01-01 00:00:00 UTC | 2038-01-19 03:14:07UTC |



### 文本字符串类型

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/MySQL文本类型.png)

- `char`

> CHAR(M) 类型一般需要预先定义字符串长度。如果不指定(M)，则表示长度默认是1个字符。
> 如果保存时，数据的实际长度比CHAR类型声明的长度小，则会在 右侧填充 空格以达到指定的长度。当MySQL检索CHAR类型的数据时，**CHAR类型的字段会去除尾部的空格**。
> 定义CHAR类型字段时，声明的字段长度即为CHAR类型字段所占的存储空间的字节数

- `varchar`

> VARCHAR(M) 定义时， 必须指定 长度M，否则报错
> 实际长度范围 `21845 = 65535/3` (一个汉字占三个字节)

| 类型       | 特点     | 空间上       | 时间上 | 适用场景             |
| ---------- | -------- | ------------ | ------ | -------------------- |
| CHAR(M)    | 固定长度 | 浪费存储空间 | 效率高 | 存储不大，速度要求高 |
| VARCHAR(M) | 可变长度 | 节省存储空间 | 效率低 | 非CHAR的情况         |


- `TEXT`

| 文本字符串类型 | 特点               | 长度 | 长度范围                         | 占用的存储空间 |
| -------------- | ------------------ | ---- | -------------------------------- | -------------- |
| TINYTEXT       | 小文本、可变长度   | L    | 0 <= L <= 255                    | L + 2 个字节   |
| TEXT           | 文本、可变长度     | L    | 0 <= L <= 65535                  | L + 2 个字节   |
| MEDIUMTEXT     | 中等文本、可变长度 | L    | 0 <= L <= 16777215               | L + 3 个字节   |
| LONGTEXT       | 大文本、可变长度   | L    | 0 <= L<= 4294967295（相当于4GB） | L + 4 个字节   |

> TEXT文本类型，可以存比较大的文本段，搜索速度稍慢，因此如果不是特别大的内容，建议使用CHAR，VARCHAR来代替。还有TEXT类型不用加默认值，加了也没用。而且text和blob类型的数据删除后容易导致“空洞”，使得文件碎片比较多，所以频繁使用的表不建议包含TEXT类型字段，建议单独分出去，**单独用一个表**

### ENUM


| 文本字符串类型 | 长度 | 长度范围        | 占用的存储空间 |
| -------------- | ---- | --------------- | -------------- |
| ENUM           | L    | 1 <= L <= 65535 | 1或2个字节     |

- 当ENUM类型包含1～255个成员时，需要1个字节的存储空间；
- 当ENUM类型包含256～65535个成员时，需要2个字节的存储空间。
- ENUM类型的成员个数的上限为65535个。

```sql
CREATE TABLE test_enum(
season ENUM('春','夏','秋','冬','unknow')
);

INSERT INTO test_enum
VALUES('春'),('秋');

-- 忽略大小写
INSERT INTO test_enum
VALUES('UNKNOW');

-- 允许按照角标的方式获取指定索引位置的枚举值
INSERT INTO test_enum
VALUES('1'),(3);
```

### SET

```sql
CREATE TABLE test_set(
s SET ('A', 'B', 'C')
);
-- 插入重复的SET类型成员时，MySQL会自动删除重复的成员
INSERT INTO test_set (s) VALUES ('A,B,C,A');

-- 向SET类型的字段插入SET成员中不存在的值时，MySQL会抛出错误。
INSERT INTO test_set (s) VALUES ('A,B,C,D');

```

### 二进制字符串类型
| 二进制字符串类型 | 特点     | 值的长度             | 占用空间  |
| ---------------- | -------- | -------------------- | --------- |
| BINARY(M)        | 固定长度 | M （0 <= M <= 255）  | M个字节   |
| VARBINARY(M)     | 可变长度 | M（0 <= M <= 65535） | M+1个字节 |

BINARY和VARBINARY类似于CHAR和VARCHAR，只是它们存储的是二进制字符串   
VARBINARY类型必须指定(M)，否则报错  

```sql
CREATE TABLE test_binary1(
f1 BINARY,
f2 BINARY(3),
# f3 VARBINARY,
f4 VARBINARY(10)
);
```


### BLOB
BLOB是一个二进制大对象，可以容纳可变数量的数据,MySQL中的BLOB类型包括 `TINYBLOB`、`BLOB`、`MEDIUMBLOB`和`LONGBLOB` 4种类型，它们可容纳值的最大长度不同。可以存储一个二进制的大对象，比如`图片`、`音频`和`视频`等。

### JSON
在MySQL 5.7中，就已经支持JSON数据类型。在MySQL 8.x版本中，JSON类型提供了可以进行自动验证的JSON文档和优化的存储结构，使得在MySQL中存储和读取JSON类型的数据更加方便和高效。
创建数据表，表中包含一个JSON类型的字段 js 

```sql
CREATE TABLE test_json(
js json
);

-- 插入
INSERT INTO test_json (js) 
VALUES ('{"name":"songhk", "age":18, "address":{"province":"beijing", "city":"beijing"}}');

-- 当需要检索JSON类型的字段中数据的某个具体值时，可以使用“->”和“->>”符号。
SELECT js -> '$.name' AS NAME,js -> '$.age' AS age ,js -> '$.address.province' AS province, js -> '$.address.city' AS city
 FROM test_json;

--结果
+----------+------+-----------+-----------+
| NAME     | age  | province  | city      |
+----------+------+-----------+-----------+
| "songhk" | 18   | "beijing" | "beijing" |
+----------+------+-----------+-----------+
1 row in set (0.00 sec)
```

### 空间类型


## 约束

- `NOT NULL` 非空约束，规定某个字段不能为空
- `UNIQUE` 唯一约束，规定某个字段在整个表中是唯一的
- `PRIMARY KEY` 主键(非空且唯一)约束
- `FOREIGN KEY` 外键约束
- `DEFAULT` 默认值约束
- `CHECK` 检查约束(mysql中无效)


### NOT NULL 非空
- 默认，所有的类型的值都可以是NULL，包括INT、FLOAT等数据类型
- 非空约束只能出现在表对象的列上，只能某个列单独限定非空，不能组合非空
- 一个表可以有很多列都分别限定了非空
- 空字符串''不等于NULL，0也不等于NULL 

```sql

CREATE TABLE emp(
id INT(10) NOT NULL,
NAME VARCHAR(20) NOT NULL,
sex CHAR NULL
);

-- 建表后修改(修改若字段中数据有null值则修改失败报错 )
alter table 表名称 modify 字段名 数据类型 not null;
alter table student modify sname varchar(20) not null;

-- 删除非空约束
ALTER TABLE emp
MODIFY sex VARCHAR(30) NULL;

ALTER TABLE emp
MODIFY NAME VARCHAR(15) DEFAULT 'abc' NULL;
```


### UNIQUE 唯一

- 同一个表可以有多个唯一约束。
- 唯一约束可以是某一个列的值唯一，也可以多个列组合的值唯一。
- 唯一性约束允许列值为空。
- 在创建唯一约束的时候，如果不给唯一约束命名，就默认和列名相同。
- MySQL会给唯一约束的列上默认创建一个唯一索引

```sql
CREATE TABLE test2(
id INT UNIQUE, --列级约束
last_name VARCHAR(15) ,
email VARCHAR(25),
salary DECIMAL(10,2),
--表级约束  uk_test2_email 是约束名 
CONSTRAINT uk_test2_email UNIQUE(email)
);

-- unique,key 二者相同作用 
create table student(
	sid int,
    sname varchar(20),
    tel char(11) unique,
    cardid char(18) unique key
);

create table student_course(
    id int,
	sid int,
    cid int,
    score int,
    unique key(sid,cid)  --复合唯一
);
```
#### 删除唯一约束

- 添加唯一性约束的列上也会自动创建唯一索引。
- 删除唯一约束只能通过删除唯一索引的方式删除。
- 删除时需要指定唯一索引名，唯一索引名就和唯一约束名一样。
- 如果创建唯一约束时未指定名称，如果是单列，就默认和列名相同；如果是组合列，那么默认和()中排在第一个的列名相同。也可以自定义唯一性约束名。

```sql
ALTER TABLE USER 
DROP INDEX uk_name_pwd;
```

### PRIMARY KEY 主键约束

- 主键约束相当于**唯一约束+非空约束**的组合，主键约束列不允许重复，也不允许出现空值      
- 如果是多列组合的复合主键约束，那么这些列都不允许为空值，并且组合的值不允许重复   
- **MySQL的主键名总是PRIMARY**，就算自己命名了主键约束名也没用。  
- 当创建主键约束时，系统默认会在所在的列或列组合上建立对应的**主键索引**（能够根据主键查询的，就根据主键查询，效率更高）。如果删除主键约束了，主键约束对应的索引就自动删除了。 

```sql
CREATE TABLE temp(
	id INT PRIMARY KEY,
    NAME VARCHAR(20)
);

-- 列约束
CREATE TABLE emp4(
id INT PRIMARY KEY AUTO_INCREMENT ,
NAME VARCHAR(20)
);

-- 表约束
CREATE TABLE emp5(
id INT NOT NULL AUTO_INCREMENT,
NAME VARCHAR(20),
pwd VARCHAR(15),
CONSTRAINT emp5_id_pk PRIMARY KEY(id)
);

--复合主键
CREATE TABLE emp6(
id INT NOT NULL,
NAME VARCHAR(20),
pwd VARCHAR(15),
CONSTRAINT emp7_pk PRIMARY KEY(NAME,pwd)
);

-- 建表后增加主键
ALTER TABLE student ADD PRIMARY KEY (sid);

ALTER TABLE emp5 ADD PRIMARY KEY(NAME,pwd);

-- 删除主键
ALTER TABLE emp5 DROP PRIMARY KEY;
```

### AUTO_INCREMENT 自增约束

- 一个表最多只能有一个自增长列
- 增长列约束的列必须是键列（主键列，唯一键列）
- 自增约束的列的数据类型必须是整数类型

>  `AUTO_INCREMENT` 8.0之前自增的标识维护在内存中,若重启mysql 就会重置为表中最大的id+1    
> MySQL 8.0将自增主键的计数器持久化到 `redolog`重做日志 中。每次计数器发生改变，都会将其写入重做日志中。如果数据库重启，InnoDB会根据重做日志中的信息来初始化计数器的内存值

```sql
CREATE TABLE employee (
  eid INT PRIMARY KEY AUTO_INCREMENT,
  ename VARCHAR (20)
) ;

-- 建表后增加自增约束
ALTER TABLE employee MODIFY eid INT AUTO_INCREMENT;

-- 删除自增约束,相当于更新字段描述 去除掉 AUTO_INCREMENT 
ALTER TABLE employee MODIFY eid INT;


```

###  FOREIGN KEY 外键约束

- 限制某个表某个字段的引用完整性
- 插入时外键对应的数据必须存在才能插入成功
- 删表时，先删从表（或先删除外键约束），再删除主表

```sql
CREATE TABLE dept (
  --主表
  did INT PRIMARY KEY,
  --部门编号
  dname VARCHAR (50) #部门名称
) ;

CREATE TABLE emp (
  --从表
  eid INT PRIMARY KEY,
  --员工编号
  ename VARCHAR (5),
  --员工姓名
  deptid INT,
  --员工所在的部门
  FOREIGN KEY (deptid) REFERENCES dept (did) -- 在从表中指定外键约束
  --emp表的deptid和和dept表的did的数据类型一致，意义都是表示部门的编号
) ;

--说明：
--（1）主表dept必须先创建成功，然后才能创建emp表，指定外键成功。
--（2）删除表时，先删除从表emp，再删除主表dept

```

> 创建完成可以看到  

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/MSYQL外键.png)

#### 外键约束等级

* `Cascade方式`：在父表上update/delete记录时，同步update/delete掉子表的匹配记录 

* `Set null方式`：在父表上update/delete记录时，将子表上匹配记录的列设为null，但是要注意子表的外键列不能为not null  

* `No action方式`：如果子表中有匹配的记录，则不允许对父表对应候选键进行update/delete操作  

* `Restrict方式`：同no action， 都是立即检查外键约束

* `Set default方式`（在可视化工具SQLyog中可能显示空白）：父表有变更时，子表将外键列设置成一个默认的值，但Innodb不能识别

如果没有指定等级，就相当于Restrict方式。

对于外键约束，最好是采用: `ON UPDATE CASCADE ON DELETE RESTRICT` 的方式。


### CHECK 约束

- 检查某个字段的值是否符号xx要求，一般指的是值的范围   
- MySQL5.7 可以使用check约束，但check约束对数据验证没有任何作用。添加数据时，没有任何错误或警告    
- 但是**MySQL 8.0中可以使用check约束了**   

```sql
-- 限制 sal的值
CREATE TABLE tests10 (id int,lastname VARCHAR(15),sal DECIMAL(10,2) CHECK (sal >200))

-- 限制性别
CREATE TABLE employee (
  eid INT PRIMARY KEY,
  ename VARCHAR (5),
  gender CHAR CHECK ('男' OR '女')
) ;


```

### DEFAULT 默认约束

> 设置默认值避免`NULL`,在插入数据时，如果此字段没有显式赋值，则赋值为默认值  

```sql
CREATE TABLE employee (
  eid INT PRIMARY KEY,
   ename VARCHAR (20) NOT NULL,
   gender CHAR DEFAULT '男',
   tel CHAR(11) NOT NULL DEFAULT '' -- 默认是空字符串
) ;

-- 增加默认约束
ALTER TABLE employee MODIFY gender CHAR DEFAULT '男';
--取消 默认约束
ALTER TABLE employee MODIFY gender CHAR;

```

## 视图

- 视图是一种 `虚拟表` ，本身是 `不具有数据` 的，占用很少的内存空间，它是 SQL 中的一个重要概念
- 视图建立在已有表的基础上, 视图赖以建立的这些表称为基表
- 可以将视图理解为存储起来的 `SELECT` 语句

```sql
CREATE VIEW empvu80
AS
SELECT employee_id, last_name, salary
FROM  employees
WHERE  department_id = 80; 

-- 多表联合视图
CREATE VIEW emp_dept
AS 
SELECT ename,dname
FROM t_employee LEFT JOIN t_department
ON t_employee.did = t_department.did;

```

- 可以创建单表视图
- 可以创建多表联合视图
- 可以基于视图创建视图

> MySQL支持使用 `INSERT` 、 `UPDATE` 和 `DELETE` 语句对视图中的数据进行插入、更新和删除操作。当视图中的数据发生变化时，数据表中的数据也会发生变化，反之亦然


![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/MySQL视图不可更新情况.png)


- 修改视图
  - 使用CREATE OR REPLACE VIEW 子句修改视图
  - ALTER VIEW

- 删除视图
  - `DROP VIEW IF EXISTS 视图名称`


## 存储过程与存储函数

> 存储过程和函数能够将复杂的SQL逻辑封装在一起，应用程序无须关注存储过程和函数内部复杂的SQL逻辑，而只需要简单地调用存储过程和函数即可

### 存储过程

五种类型
- 没有参数（无参数无返回）
- 仅仅带 IN 类型（有参数无返回）
- 仅仅带 OUT 类型（无参数有返回）
- 既带 IN 又带 OUT（有参数有返回）
- 带 INOUT（有参数有返回）

```sql
# 简单实现

DELIMITER //
CREATE PROCEDURE select_all_data2()
BEGIN
  SELECT * from emp;
END //
DELIMITER;

#调用
CALL select_all_data()

# 带返回值的
CREATE PROCEDURE show_min_salary(OUT ms DOUBLE)
BEGIN
 select min(salary) INTO ms from emp;
END
# 写入用户变量中
CALL show_min_salary(@ms)
# 调用显示变量
SELECT @ms


# 带入参
CREATE PROCEDURE show_someone_salary(IN epname VARCHAR(20))
BEGIN
	SELECT * from emp WHERE last_name = epname;
END


CALL show_someone_salary('King')

# 入参+返回值
create PROCEDURE show_someone_salary2(in empname VARCHAR(20),OUT empsa DECIMAL(10,2))
BEGIN
	SELECT salary INTO empsa from emp
 WHERE last_name = empname;
END

# 返回多个结果会报错
call show_someone_salary2('Abel',@sal)

SELECT @sal

# 入参和返回值同一个参数
CREATE PROCEDURE show_mgr_name ( INOUT empname VARCHAR ( 25 ) )
BEGIN
SELECT
	last_name INTO empname
FROM
	emp
WHERE
	employee_id = ( SELECT manager_id FROM emp WHERE last_name = empname );
END

SET @empname := 'Olson';
CALL show_mgr_name(@empname)

SELECT @empname

```


### 存储函数

> MySQL支持自定义函数，定义好之后，调用方式与调用MySQL预定义的系统函数一样`LENGTH、SUBSTR、CONCAT...`

```sql
# 基础调用
CREATE FUNCTION email_by_name()
# 返回值类型
RETURNS VARCHAR(25)
# 函数的约束
DETERMINISTIC
CONTAINS SQL

BEGIN
# 函数体,函数体中肯定有 RETURN 语句
return (SELECT email FROM emp WHERE last_name='Allan');
END

# 调用
SELECT email_by_name()


# 入参调用
CREATE FUNCTION email_by_id(emp_id int)

returns VARCHAR(25)
DETERMINISTIC
CONTAINS SQL
BEGIN
return (SELECT email FROM  WHERE employee_id=emp_id);
END

SELECT email_by_id(employee_id) FROM emp

```

### 对比
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/mysql存储过程函数对比.png)

## 变量

### 系统变量
> 系统变量分为**全局系统变量**（需要添加 global 关键字）以及**会话系统变量**（需要添加 session 关键字），
> 有时也把全局系统变量简称为全局变量，有时也把会话系统变量称为local变量。如果不写，**默认会话级别**
> 全局系统变量针对于所有会话（连接）有效
> 会话系统变量仅针对于当前会话（连接）有效,会话期间，当前会话对某个会话系统变量值的修改，不会影响其他会话同一个会话系统变量的值

```sql
# 全局
SHOW GLOBAL VARIABLES;
# 会话
SHOW SESSION VARIABLES;
# 查询指定变量
SELECT @@GLOBAL.MAX_CONNECTIONS
SELECT @@SESSION.***

#为某个系统变量赋值
#方式1：
SET @@global.变量名=变量值;
#方式2：
SET GLOBAL 变量名=变量值;

#为某个会话变量赋值
#方式1：
SET @@session.变量名=变量值;
#方式2：
SET SESSION 变量名=变量值;
```

### 用户变量

- 会话用户变量

```sql
#方式1：“=”或“:=”
SET @用户变量 = 值;
SET @用户变量 := 值;
#方式2：“:=” 或 INTO关键字
SELECT @用户变量 := 表达式 [FROM 等子句];
SELECT 表达式 INTO @用户变量 [FROM 等子句];
```

- 局部变量(只在存储过程和函数中使用,只在 BEGIN 和 END 语句块中有效)可以使用 `DECLARE` 语句定义一个局部变量,必须在首行

```sql
DECLARE 变量名 类型 [default 值];  # 如果没有DEFAULT子句，初始值为NULL

# 赋值
SET 变量名=值;
SET 变量名:=值;
SELECT 字段名或表达式 INTO 变量名 FROM 表;

```

### (异常)定义条件和处理程序

```bash
mysql> CALL UpdateDataNoCondition();
ERROR 1048 (23000): Column 'email' cannot be null
```

> `1048` 为 `MySQL_error_code` , `23000` 为 `sqlstate_value`
> `MySQL_error_code` 是数值类型错误代码
> `sqlstate_value` 是长度为5的字符串类型错误代码


> **定义条件**
> 定义条件就是给MySQL中的错误码命名，这有助于存储的程序代码更清晰。它将一个`错误名字`和指定的`错误条件` 关联起来。这个名字可以随后被用在定义处理程序的 DECLARE HANDLER 语句中

```sql
DECLARE 错误名称 CONDITION FOR 错误码（或错误条件）

#命名错误 为 command_not_allowed
#使用MySQL_error_code
DECLARE command_not_allowed CONDITION FOR 1148;
#使用sqlstate_value
DECLARE command_not_allowed CONDITION FOR SQLSTATE '42000';
```

> **处理程序**
> 可以为SQL执行过程中发生的某种类型的错误定义特殊的处理程序

```sql
DECLARE 处理方式 HANDLER FOR 错误类型 处理语句
```
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/mysql处理程序.png)

```sql
#方法1：捕获sqlstate_value
DECLARE CONTINUE HANDLER FOR SQLSTATE '42S02' SET @info = 'NO_SUCH_TABLE';

#方法2：捕获mysql_error_value
DECLARE CONTINUE HANDLER FOR 1146 SET @info = 'NO_SUCH_TABLE';

#方法3：先定义条件，再调用,先定义错误名,再根据错误名捕获
DECLARE no_such_table CONDITION FOR 1146;
DECLARE CONTINUE HANDLER FOR NO_SUCH_TABLE SET @info = 'NO_SUCH_TABLE';

```
### 流程控制
> MySQL流程控制主要有3类,只能用于存储程序

- 条件判断:IF 语句和 CASE 语句
- 循环:LOOP、WHILE 和 REPEAT 语句
- 跳转:ITERATE 和 LEAVE 语句


#### IF

```sql
# IF公式
IF 表达式1 THEN 操作1
ELSEIF 表达式2 THEN 操作2……
ELSE 操作N
END IF

# 存储过程
DELIMITER //
CREATE PROCEDURE update_sal(IN emp_id INT)
BEGIN
# 定义变量
	DECLARE emp_sal DOUBLE;
	DECLARE emp_date DOUBLE;
  # 查询变量赋值
	SELECT salary INTO emp_sal FROM emp WHERE employee_id = emp_id;

	SELECT DATEDIFF(CURRENT_DATE(),hire_date)/365 INTO emp_date FROM emp WHERE employee_id = emp_id;
#条件判断
	IF emp_sal <8000 AND emp_date >5
		THEN
			UPDATE emp SET salary = salary+500 WHERE employee_id = emp_id;
	END IF;
END //
DELIMITER ;
```

#### CASE

```sql
#情况一：类似于switch
CASE 表达式
WHEN 值1 THEN 结果1或语句1(如果是语句，需要加分号)
WHEN 值2 THEN 结果2或语句2(如果是语句，需要加分号)
...
ELSE 结果n或语句n(如果是语句，需要加分号)
END [case]（如果是放在begin end中需要加上case，如果放在select后面不需要）


#情况二：类似于多重if
CASE
WHEN 条件1 THEN 结果1或语句1(如果是语句，需要加分号)
WHEN 条件2 THEN 结果2或语句2(如果是语句，需要加分号)
...
ELSE 结果n或语句n(如果是语句，需要加分号)
END [case]（如果是放在begin end中需要加上case，如果放在select后面不需要）
```

#### LOOP

```sql
# loop_label 循环名,用于结束循环,
[loop_label:] LOOP
循环执行的语句
END LOOP [loop_label]
```

```sql
# 示例
DELIMITER //
CREATE PROCEDURE test_loop(IN emp_id INT)
BEGIN
	DECLARE num INT DEFAULT 1;
	loop_label:LOOP
		SET num  =num +1;
    # 退出循环
		IF num >=10 THEN LEAVE loop_label;
		END IF;
	END LOOP loop_label;

	SELECT num;

END //
DELIMITER ;

CALL test_loop(1);
```

#### WHILE

> WHILE语句创建一个带条件判断的循环过程。WHILE在执行语句执行时，先对指定的表达式进行判断，如果为真，就执行循环内的语句，否则退出循环

```sql

[while_label:] WHILE 循环条件  DO
循环体
END WHILE [while_label];

#示例 i值小于10时，将重复执行循环过程
DELIMITER //
CREATE PROCEDURE test_while()
BEGIN
DECLARE i INT DEFAULT 0;
WHILE i < 10 DO
SET i = i + 1;
END WHILE;
SELECT i;
END //
DELIMITER ;
#调用
CALL test_while();

```
#### REPEAT

> REPEAT 语句创建一个带条件判断的循环过程。与WHILE循环不同的是，REPEAT 循环首先会执行一次循环，然后在 UNTIL 中进行表达式的判断，如果满足条件就退出，即 END REPEAT

```sql
[repeat_label:] REPEAT
　　　　循环体的语句
UNTIL 结束循环的条件表达式
END REPEAT [repeat_label]
```

#### LEAVE

> LEAVE语句：可以用在循环语句内，或者以 BEGIN 和 END 包裹起来的程序体内，表示跳出循环或者跳出程序体的操作。如果你有面向过程的编程语言的使用经验，你可以把 LEAVE 理解为 break


#### ITERATE
> ITERATE 语句：只能用在循环语句（LOOP、REPEAT和WHILE语句）内，表示重新开始循环，将执行顺序转到语句段开头处。
> 可以把 ITERATE 理解为 continue，意思为“再次循环”

```sql
DELIMITER //
CREATE PROCEDURE test_iterate ()
BEGIN
  DECLARE num INT DEFAULT 0 ;
  my_loop :
  LOOP
    SET num = num + 1 ;
    IF num < 10
    # 再次循环
    THEN ITERATE my_loop ;
    ELSEIF num > 15
    #退出循环
    THEN LEAVE my_loop ;
    END IF ;
    SELECT 'test' ;
  END LOOP my_loop ;
END //

DELIMITER ;
```
### 游标
> 虽然我们也可以通过筛选条件 WHERE 和 HAVING，或者是限定返回记录的关键字 LIMIT 返回一条记录，
> 但是，却无法在结果集中像指针一样，向前定位一条记录、向后定位一条记录，逐条读取 结果集中的数据,或者是 随意定位到某一条记录 ，并对记录的数据进行处理

示例:创建存储过程“get_count_by_limit_total_salary()”，声明IN参数 limit_total_salary，DOUBLE类型；声明
OUT参数total_count，INT类型。函数的功能可以实现累加薪资最高的几个员工的薪资值，直到薪资总和
达到limit_total_salary参数的值，返回累加的人数给total_count

```sql
DELIMITER //
CREATE PROCEDURE get_count_salary (
  IN limit_total DOUBLE,
  OUT total_count INT
)
BEGIN
  # 记录累加工资总和
  DECLARE sum_sal DOUBLE DEFAULT 0.0 ;
  DECLARE emp_sal DOUBLE ;
  DECLARE emp_count INT DEFAULT 0 ;
  # 声明游标(查询所有工资并排序)
  DECLARE emp_cursor CURSOR FOR
  SELECT
    salary
  FROM
    emp
  ORDER BY salary DESC ;

  #打开游标
  OPEN emp_cursor ;
  REPEAT
    #取一条记录
    FETCH emp_cursor INTO emp_sal ;
    SET sum_sal = sum_sal + emp_sal ;
    SET emp_count = emp_count + 1 ;
    UNTIL sum_sal >= limit_total
  END REPEAT ;
  SET total_count = emp_count ;
  # 关闭游标
  CLOSE emp_cursor ;
END //
DELIMITER ;

# 调用
CALL get_count_salary(200000,@total_count);
SELECT @total_count
```


### MySQL 8.0的新特性—全局变量的持久化
> 在MySQL数据库中，全局变量可以通过SET GLOBAL语句来设置。例如，设置服务器语句超时的限制，可以通过设置系统变量max_execution_time来实现

```sql
SET GLOBAL MAX_EXECUTION_TIME=2000;
```

> 使用SET GLOBAL语句设置的变量值只会临时生效,数据库重启后,服务器又会从MySQL配置文件中读取变量的默认值。MySQL 8.0版本新增了 `SET PERSIST` 命令

```sql
SET PERSIST global max_connections = 1000;
```
> MySQL会将该命令的配置保存到数据目录下的 mysqld-auto.cnf 文件中，下次启动时会读取该文件，**用其中的配置来覆盖默认的配置文件**


## 触发器
> 触发器是由事件来触发某个操作，这些事件包括 `INSERT、UPDATE、DELETE` 事件。
> 所谓事件就是指用户的动作或者触发某项行为。如果定义了触发程序，当数据库执行这些语句时候，就相当于事件发生了，就会自动激发触发器执行相应的操作
> 只有直接对表执行操作语句时才会激活触发器,如: 外键的关联删除无法触发

```sql
CREATE TRIGGER 触发器名称
# 语句块之前还是之后  监听触发动作 是插入 更新 删除
{BEFORE|AFTER} {INSERT|UPDATE|DELETE} ON 表名
FOR EACH ROW
触发器执行的语句块;
```

```sql
DELIMITER //
#触发器名称为 before_insert
CREATE TRIGGER before_insert
# 在表 test_trigger insert 之前触发
BEFORE INSERT ON test_trigger
FOR EACH ROW
BEGIN
# 触发器执行内容
	INSERT INTO test_trigger_log(t_log)
	VALUES('before inserts...');

END //

DELIMITER ;
# 插入数据前会自动在log表中写入
INSERT INTO test_trigger(t_note)
VALUES('Tom....')
```

## MySQL8 特性

- 更简便的NoSQL支持
- 更好的索引
- 更完善的JSON支持
- 安全和账户管理
- InnoDB的改进和优化
- 数据字典
- 字符集支持，默认改为utf8mb4
- 优化器增强，索引
- 公用表表达式(替换子查询)
- 窗口函数
- 正则表达式
- 内部临时表
- 日志记录
- 备份锁
- 增强的MySQL复制

###  窗口函数

> 窗口函数的作用类似于在查询中对数据进行分组，不同的是，分组操作会把分组的结果聚合成一条记录，而窗口函数是将结果置于每一条数据记录中

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/MySQL窗口函数.png)

### 公用表表达式

> 公用表表达式（或通用表表达式）简称为CTE（Common Table Expressions）。CTE是一个命名的临时结果集，作用范围是当前语句。CTE可以理解成一个可以复用的子查询，当然跟子查询还是有点区别的，CTE可以引用其他CTE，但子查询不能引用其他子查询

- 普通公用表表达式
- 递归公用表表达式

```sql
WITH emp_dept_id
AS (SELECT DISTINCT department_id FROM employees)
SELECT *
FROM departments d JOIN emp_dept_id e
ON d.department_id = e.department_id
```

# 高级篇
## 字符集
 
> 8.0 默认字符集为 `utf8mb4`    
> 5.7 默认字符集为 `utf-8`        
> 通过修改 `/etc/my.cnf` 文件,修改编码方式,修改后只能对新创建的数据库生效      
> 也可以通过 `alert table` 修改数据库的字符集      

- 8默认字符集

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/MySQL80字符集.png)

- 5.7默认字符集

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/MySQL5.7字符集.png)

### 各个级别的字符集

- 服务器级别
- 数据库级别
- 表级别
- 列级别

```sql
#查看字符集
show variables like 'character%';
```

### 字符集的关系

> `utf8` 字符集表示一个字符需要1-4个字节    
> `utf8mb3` 是阉割过的 utf8 使用1-3个字节表示字符   
> `utf8mb4` 是全面的utf8字符集,使用1-4个字节表示字符,可以存储`emoji表情`     
> MySQL中 `utf8=utf8mb3`    



## SQL 大小写

-  关键字和函数名称全部大写
-  数据库,表,表别名,字段名,字段别名全部小写
-  SQL语句必须以分号结尾


## 存储引擎对比

| 对比项         | **MyISAM**                                               | **InnoDB**                                                   |
| -------------- | -------------------------------------------------------- | ------------------------------------------------------------ |
| 外键           | 不支持                                                   | 支持                                                         |
| 事务           | 不支持                                                   | 支持                                                         |
| 行表锁         | 表锁，即使操作一条记录也会锁住整个表，不适合高并发的操作 | 行锁，操作时只锁某一行，不对其它行有影响，适合高并发的操作   |
| 缓存           | 只缓存索引，不缓存真实数据                               | 不仅缓存索引还要缓存真实数据，对内存要求较高，而且内存大小对性能有决定性的影响 |
| 自带系统表使用 | Y                                                        | N                                                            |
| 关注点         | 性能：节省资源、消耗少、简单业务                         | 事务：并发写、事务、更大资源                                 |
| 默认安装       | Y                                                        | Y                                                            |
| 默认使用       | N                                                        | Y                                                            |

<!-- ![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/innodb对比.png) -->


## 索引

> 索引是存储引擎中用于快速找到数据记录的一种**数据结构**,好比书的目录   
> MySQL中进行数据查找时,首先查看查询条件是否命中某条索引,符合则通过索引查找相关数据,如果不符合则需要全表扫描    
> 建立索引的目的是为了减少磁盘I/O的次数,加快查询速率   
> 索引是在存储引擎中实现的,因此每种存储引擎不一定完全相同,所有存储引擎每个表最少支持16个索引    

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/二叉搜索树.png)

索引的优点：
- 降低数据库的IO成本
- 唯一索引可以保证数据的唯一性
- 加速表和表之间的连接
- 使用分组和排序时,可以显著减少分组排序时间，降低CPU消耗

索引的缺点：
- 创建和维护索引需要耗时，数据越大耗时越长
- 索引需要占据磁盘空间
- 索引会降低更新表速度


### 设计索引

- 1 基础数据页

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/MySQL索引基础数据页.png)

> 基础数据页代表基本数据的存放关系

- 2 目录记录页

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/MySQL目录记录.png)

> 提取目录项作为一个数据页后实现如下图

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/MySQL目录记录提取.png)

> 目录项记录 的 record_type 值是1，而 普通用户记录 的 record_type 值是0

- 3 提取目录记录页(高级目录)

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/MySQL树.png)


> 上面的结构可以整合为图如下

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/MySQLb加树.png)

> 注意:数据之间是单向链表结构,数据页和数据页之间是双向链表结构

这个数据结构，它的名称是 `B+树`,一个 `B+树` 一般不超过4层,最下面存放真正数据的称为 `0层`,上面为 `1层,2层...`
若 `B+树`有4层，最多能存放 1000×1000×1000×100=1000,0000,0000 条记录


### 常见索引概念
> 索引按照物理实现方式,可以分为2种,`聚簇(聚集)`和`非聚簇`索引,`非聚簇`索引也称为`二级索引`或者`辅助索引`

#### 聚簇索引
> 聚簇索引并不是一种单独的索引类型,而是一种数据存储方式(所有的用户记录都存储在了叶子节点)

- 对于MySQL数据库目前**只有InnoDB数据引擎支持聚簇索引**
- 每个表只能有一个聚簇索引,一般为**主键**
- 如果没有定义主键,会选择非空的唯一索引代替,如果没有这样的索引,**就会隐式定义一个主键来作为聚簇索引**
- 为了充分利用聚簇索引的特性,所以**尽量保证主键有序**,不要使用uuid等无序字符串作为主键

> 优点:
> - 数据访问更快
> - 对于主键的排序查找和范围查找速度很快
> - 节省IO操作

> 缺点:
> - 插入速度严重依赖于插入顺序
> - 更新主键的代价很高
> - 二级索引访问需要进行两次索引查找

#### 二级索引(非聚簇索引)
> 一个表中`只能有一个聚簇索引`,但是可以`有多个非聚簇索引`,使用非主键列作为索引,称为非聚簇索引        
> 依据对应列构建B+树,与聚簇索引不同的是,在叶子节点中存储的**不是主键和所有值**,而是`该列和主键的值`    


![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/非聚簇索引.png)

> 查询值的过程:先**根据非聚簇索引找到聚簇索引再找到对应数据**,这种操作称为`回表`

> 防止非主键数据相同无法判断,他们的父页节点也会携带主键的最小值(c2列都为1的情况)

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/真正的二级索引构建.png)

#### 聚簇和非聚簇对比

- 聚簇索引的叶子节点存储的是数据记录,非聚簇索引叶子节点存储的是数据位置(主键)
- 一个表只能有一个聚簇索引,可以有多个非聚簇索引
- 使用聚簇索引时,数据的查询效率高,但是对数据进行插入,删除,更新等操作,效率会比非聚簇索引低

#### 联合索引

我们也可以同时以多个列的大小作为排序规则，也就是同时为**多个列建立索引**，比方说我们想让B+树按照 c2和c3列 的大小进行排序，这个包含两层含义：
- 先把各个记录和页按照c2列进行排序。
- 在记录的c2列相同的情况下，采用c3列进行排序
注意:以c2和c3列的大小为排序规则建立的B+树称为联合索引,本质上也是一个二级索引。它的意思与分别为c2和c3列分别建立索引的表述是不同的

不同点如下：
- 建立 联合索引 只会建立如上图一样的`1棵B+树`。
- 为c2和c3列分别建立索引会分别以c2和c3列的大小为排序规则建立`2棵B+树`


#### 总结

**根页面万年不动**
- 自动创建聚簇索引时,会为索引创建一个根页面
- 插入数据时,将数据记录到根页面
- 根页面空间用完,此时会将根页面中所有记录复制到一个新分配的页`a`, 对这个页进行页分裂,得到页`b`,新插入的数据会进入到`a`或`b`中,而**根页面升级为目录页**  

**二级索引的内节点的目录项记录的内容实际上是由三个部分构成的**,防止值重复无法判断插入位置
- 索引列的值
- 主键值
- 页号

**一个页面最少可以存储2条记录**


### MyISAM中的索引方案
| 索引/存储引擎 | MyISAM | InnoDB | Memory |
| ------------- | ------ | ------ | ------ |
| B-Tree索引    | 支持   | 支持   | 支持   |

即使多个存储引擎支持同一种类型的索引，但是他们的实现原理也是不同的。Innodb和MyISAM默认的索引是Btree索引；而Memory默认的索引是Hash索引。   
MyISAM引擎使用B+Tree作为索引结构，叶子节点的data域存放的是`数据记录的地址`   
**MyISAM的索引方式都是“非聚簇”的，与InnoDB包含1个聚簇索引是不同的**  

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/MyISAM引擎索引.jpg)   

**小结两种引擎中索引的区别：**

① 在InnoDB存储引擎中，我们只需要根据主键值对`聚簇索引`进行一次查找就能找到对应的记录，而在`MyISAM`中却需要进行一次`回表`(通过索引获取的地址去获取数据的过程)操作，意味着MyISAM中建立的索引相当于全部都是`二级索引`。 

> `InnoDB`中的回表是指二级索引获取到主键后去聚簇索引中查询数据的过程     
> `MyISAM`中的回表是指索引获取到数据地址后去磁盘获取数据的过程   

② InnoDB的数据文件本身就是索引文件，而MyISAM索引文件和数据文件是`分离的`，索引文件仅保存数据记录的地址。

③ InnoDB的非聚簇索引data域存储相应记录`主键的值`，而MyISAM索引记录的是`地址`。换句话说，InnoDB的所有非聚簇索引都引用主键作为data域。

④ MyISAM的回表操作是十分`快速`的，因为是拿着地址偏移量直接到文件中取数据的，反观InnoDB是通过获取主键之后再去聚簇索引里找记录，虽然说也不慢，但还是比不上直接用地址去访问。

⑤ InnoDB要求表`必须有主键`（`MyISAM可以没有`）。如果没有显式指定，则MySQL系统会自动选择一个可以非空且唯一标识数据记录的列作为主键。如果不存在这种列，则MySQL自动为InnoDB表生成一个隐含字段作为主键，这个字段长度为6个字节，类型为长整型。



### 索引的代价
- 空间:每个B+树的每个节点都是一个数据页,每个页默认`16KB`
- 时间:每次对表进行`增删改`时,都需要去修改各个B+树索引,增、删、改操作可能会对节点和记录的排序造成破坏，所以存储引擎需要额外的时间进行一些`记录移位`，`页面分裂`、`页面回收`等操作来维护好节点和记录的排序  


### 数据结构选择的合理性

#### HASH索引
- hash索引仅能满足 `=` `<>` 和 `IN`查询,如果进行范围查询,时间复杂度会退化为 `O(n)` 
- hash数据的存储是没有顺序的,若使用`ORDER BY`的情况下,还需对数据重新排序
- 对于联合索引情况下,hash值是将联合索引键合并后来计算,无法对单个键或几个索引进行查询
- 大数据量下,或者重复数据较多(年龄/性别...),hash碰撞过多效率就会下降 

| 索引/存储引擎 | MyISAM | InnoDB | Memory |
| ------------- | ------ | ------ | ------ |
| HASH索引    | 不支持   | 不支持   | 支持   |

> Redis 存储核心就是Hash表     
> 对于经常使用等值判断的情况,Hash索引是个不错的选择   
> InnoDB不支持Hash索引,但是提供`自适应Hash索引(Adaptive Hash Index)`,若一条数据经常被访问,满足一定条件时就会将数据页地址存放到Hash表中   


#### 二叉搜索树  
- 一个节点只能有两个子节点
- 左子节点 < 本节点,右子节点 > 本节点  

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/标准二叉搜索.png) 

存在特殊情况,若数据顺序为 (5,22,23,34,77,89,91) 那么构造出来的二叉搜索树则为

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/偏离二叉搜索树.png)   

这样的搜索树的效率就会大大下降,如果搜索 91 的话 和`链表`无差 

#### 平衡二叉树 
基于`二叉树退化成链表`的问题,提出了 `平衡二叉搜索树 Balanced Binary Tree` 又称为AVL树     
平衡二叉树首先需要符合二叉查找树,其次必须满足任何节点的两个子树的高度最大差为1    

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/平衡二叉树的缺点.png)  

#### B树  
`Balance Tree` 即 多路平衡查找树,简写为`B-Tree`  
**在非叶子(最底层)节点上也存放对应数据(而不是像B+树存放目录,所有真实数据都存在叶子节点)**   

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/B树示例.jpg)

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/完整的B树.png)  

#### B+树  
`B+`树由`B树`和`索引顺序访问方法(ISAM)`演化而来


### 索引的分类

- `普通索引`
- `唯一索引`
- `主键索引`
- `全文索引`
- 单列索引
- 多列(联合)索引
- 空间索引


### 创建索引

- `UNIQUE`、`FULLTEXT`和`SPATIAL`为可选参数，分别表示唯一索引、全文索引和空间索引；
- `INDEX`与`KEY`为同义词，两者的作用相同，用来指定创建索引；
- `index_name`指定索引的名称，为可选参数，如果不指定，那么MySQL默认col_name为索引名；
- `col_name`为需要创建索引的字段列，该列必须从数据表中定义的多个列中选择；
- `length`为可选参数，表示索引的长度，只有字符串类型的字段才能指定索引长度；
- `ASC`或`DESC`指定升序或者降序的索引值存储。

```sql
ALTER TABLE table_name 
ADD [UNIQUE | FULLTEXT | SPATIAL] [INDEX | KEY] [index_name] (col_name[length],...) [ASC | DESC]
```

- 隐式创建
  - `主键`,`唯一UNIQUE字段`,`外键`会默认创建索引
- 显式创建:直接创建索引

```sql
#查看索引
SHOW INDEX FROM table_name

# 创建普通索引
CREATE TABLE book(
book_id INT ,
book_name VARCHAR(100),
authors VARCHAR(100),
info VARCHAR(100) ,
comment VARCHAR(100),
year_publication YEAR,
INDEX(year_publication)
);

# 创建唯一索引,唯一索引不可重复,可为NULL
CREATE TABLE test1(
id INT NOT NULL,
name varchar(30) NOT NULL,
UNIQUE INDEX uk_idx_id(id)
);

# 创建单列索引 前20个字符
CREATE TABLE test2(
id INT NOT NULL,
name CHAR(50) NULL,
INDEX single_idx_name(name(20))
);

# 创建组合索引
CREATE TABLE test3(
id INT(11) NOT NULL,
name CHAR(30) NOT NULL,
age INT(11) NOT NULL,
info VARCHAR(255),
INDEX multi_idx(id,name,age)
);

# 创建索引
ALERT TABLE table_name ADD INDEX index_name(col_name);
CREATE INDEX index_name ON table_name(col_name);

# 唯一索引
ALERT TABLE table_name ADD UNIQUE INDEX index_name(col_name);
CREATE UNIQUE INDEX index_name ON table_name(col_name);

# 删除索引
ALTER TABLE table_name DROP INDEX index_name;
DROP INDEX index_name ON table_name;

```

### 8.0的降序索引,隐藏索引
8.0之前默认是升序索引,即使指定索引是降序(DESC)也无效,8.0后开始支持降序索引,降序索引针对某些特定情况会更高效(譬如 create_time DESC的情况)

`隐藏索引`,若显式的删除索引,后续再添加索引,对于数据量比较大的表的话会消耗过多的资源,操作成本高
若不想使用某个索引,可以将其设置为`隐藏索引`,查询优化器不在使用这个索引
需要注意的是:主键(或者被当作主键的)无法被设置为隐藏索引

> 当索引被隐藏时，它的内容仍然是和正常索引一样实时更新的。如果一个索引需要长期被隐藏，那么可以将其删除，因为索引的存在会影响插入、更新和删除的性能。


```sql
# 创建隐藏索引
CREATE TABLE test3(
id INT(11) NOT NULL,
name CHAR(30) NOT NULL,
age INT(11) NOT NULL,
info VARCHAR(255),
INDEX age_ind(age) invisible
);

# 添加隐藏索引
ALTER TABLE test3 ADD UNIQUE INDEX age_ind(age) invisible;

CREATE INDEX age_ind invisible ON test3(age);
#切换成隐藏索引
ALTER TABLE tablename ALTER INDEX index_name INVISIBLE;
#切换成非隐藏索引
ALTER TABLE tablename ALTER INDEX index_name VISIBLE;
```


### 适合创建索引的11种情况

- 字段的数值有唯一性的限制
  - 业务上具有唯一特性的字段,即使是组合字段,最好也建成唯一索引
- 频繁作为`WHERE`查询条件的字段
- 经常`GROUP BY` `ORDER BY`的列
  - 同时使用二者需要保证联合索引的顺序`ORDER BY`的列在前,`GROUP BY`的列在后
- `UPDATE``DELETE`的`WHERE`条件列
  - 若更新的字段是非索引字段,效率提升会更明显
- `DISTINCT`字段需要创建索引
- 多表`JOIN`连接操作时,创建索引注意事项
  - 连接表数尽量不要超过3张
  - 对`WHERE`条件创建索引
  - 对用于`连接的字段`创建索引,并且该字段在多表种的`类型必须一致`
- 使用列的类型小的创建索引
  - 优先级`TINYINT`>`MEDIUMINT`>`INT`>`BIGINT`
- 使用字符串前缀创建索引
  - 对于过长的字符串,截取字符串前面一部分内容建立索引即`前缀索引`指定索引长度
  - `alert table shop_table add index_name(col_name(12)); `只取前12个字符作为前缀索引
  - `阿里`在varchar字段上建立索引时，`必须`指定索引长度，根据实际文本区分度决定索引长度
  - 一般对字符串类型数据，长度为 20 的索引，区分度会 高达90% 以上
- 区分度高(散列性高,不重复度)的列适合作为索引
- 使用最频繁的列放到联合索引的左侧
- 在多个字段都要创建索引的情况下，`联合索引优于单值索引`

> 实际工作中,建议单张表中`索引数量不超过6个`,索引越多占据磁盘空间越大,索引会影响插入,删除,更新操作的性能



### 不适合创建索引的情况
- 在where中使用不到的字段
- 数据量小的表最好不要使用索引
- 有大量重复数据的列上不要建立索引,如:性别
- 避免对经常更新的表创建过多的索引
- 不建议用无序的值作为索引(参考不用uuid之类做主键)
- 及时删除不再使用或者很少使用的索引
- 不要定义冗余或重复的索引











## InnoDB的数据存储结构

### 页的概述
InnoDB 将数据划分为若干个页,InnoDB 中页的**大小**默认为 `16KB`

以 `页` 作为磁盘和内存之间交互的`基本单位`,也就是一次最少重磁盘中读取 `16KB` 的内容到内存中,一次最少把内存中的`16KB`内容刷新到磁盘中,在数据库中,不论读几行数据,都需要将其所在的`页` 进行加载,**数据库管理存储空间的基本单位是页,数据库IO操作最小单位是页**,一个页中可以存储多个行记录

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/InnoDB存储结构.png)

- 页a,页b...这些页通过`双向链表`关联,每个数据页的记录按照主键值从小到大组成一个`单项链表`
- 64个连续页组成一个`区`,一个区的大小是 `64*16KB=1MB`
- 多个`区`组成一个`段`,常见有`数据段`,`索引段`,`回滚段`
- 各个段构成表空间,`表空间`是一个逻辑容器,表空间从管理上可以划分为`系统表空间`,`用户表空间`,`撤销表空间`,`临时表空间`等


### 页的内部结构
页的类型划分:
- `数据页(B+树节点)`
- `系统页` System Page
- `Undo页` undo Log Page
- `事务数据页` Transaction system Page

> 数据页的`16KB`存储空间被划分为七个部分,分别是 文件头(File Header),页头(Page Header),最大最小记录,用户记录(User Records),空闲空间(Free Space),页目录(Page Directory)和文件尾(File Tailer)

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/页结构.png)

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/页的七部分.png)

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/数据页之间的关系.png)
#### FileHeader,FileTrailer
文件头部,总计`38字节`

> - `FIL_PAGE_OFFSET`4字节
> 每个页的页号


> - `FIL_PAGE_TYPE`2字节
> 前页的类型


> - `FIL_PAGE_PREV ` `FIL_PAGE_NEXT` 各4字节
> `FIL_PAGE_PREV` 和 `FIL_PAGE_NEXT` 就分别代表本页的上一个和下一个页的页号


> - `FIL_PAGE_SPACE_OR_CHKSUM`4字节
> 当前页面的校验和,`Header` 和 `Trailer` 都有校验和,通过这个值校验整页数据的完整性


> - `FIL_PAGE_LSN`8字节
> 日志序列号,页面被最后修改时对应的日志序列位置,也是为了校验页的完整性的,若`Header` 和 `Trailer` 的 `LSN` 值校验失败,说明同步过程出现了问题


`FileTrailer`只有一个`FIL_PAGE_END_LSN`共8字节,其中前4字节的 `CHKSUM`校验和,4字节的`LSN`与头部相同,保证数据完整性

#### FreeSpace,UserRecords,Infimum,Supremum
- `FreeSpace` 空闲空间

> - `UserRecords` 用户记录
> 一开始并没有`UserRecords` 部分,当插入一条记录时,就会从`FreeSpace`部分,申请划分一个记录大小的空间到`UserRecords`
> `UserRecords`中的记录按照指定`行格式`排列,形成`单链表`

- `Infimum,Supremum` 最大最小记录,每个数据页都存在两个虚拟的行记录用来限定记录的边界


#### PageDirectory,PageHeader
> 为了方便查找,将所有记录分成几个组,包括最小记录和最大记录,但是不包括已经删除的记录
> 第一组只有最小记录,最后一组是包含最大记录在内的1-8条记录,其余按照4-8条一组进行分组
> 每组最后一条记录的头信息会存储改组一共有多少条记录,写入`n_owned`
> `pageDirectory`页目录用来存储每组最后一条记录的地址偏移量,每个地址偏移量称为`槽slot`,每个槽指向不同组的最后一条记录

`PageHeader` 可以记录数据插入的方向记录数等信息


### InnoDB行格式

> MySQL5.7 和MySQL8 中的默认行格式都是`Dynamic`

#### COMPACT

行格式示意图
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/compact行格式信息.png)

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/记录头信息.png)

- `变长字段长度列表`,若列长度小于255字节,用1字节表示;若大于255字节,用2字节表示;存储变长字段的长度信息,存储顺序和字段顺序相反
例:两个变长varchar字段在表结构中顺序是`a(10)`,`b(15)`,那么变长字段长度列表中存储的长度顺序就是15,10

- `NULL标志位`指示了该行数据中是否有null值,有则用1表示;
例:字段a,b,c,a是主键,b是null,c非null,那么行格式中null值列表为`01`,因为a肯定不是null(非空约束会忽略),b是0,c是1;

- `记录头信息`固定占用5字节(40位);

记录头信息详情
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/compact记录头信息.png)

> `delete_flag` 占用一个二进制位,`1`为被删除,`0`为没有被删除
> 被删除的记录不会立即从磁盘中移除,只是打上一个删除标记,所有被删掉的记录都会组成垃圾链表,这个链表中的记录占用的空间称为`可重用空间`,之后插入新纪录可能会覆盖这些空间
> `min_rec_flag`只有在存储`目录项记录`的页中的`主键最小目录项记录`值为1,其他值为0
> `record_type` 0表示普通记录,1表示B+树非叶节点记录,2表示最小记录,3表示最大记录(实际显示为2进制表示)
> `heap_no`表示当前记录在本页中的位置,从`2`开始,MySQL自动插入两个记录占位`0``1`分别为最小和最大记录
> `n_owned`分组后最后一条记录会存储该组中有多少条记录,其他记录为`0`,参考`PageDirectory`
> `next_record`地址偏移量(往后移动多少个字节是下一条记录)

- `真实数据`,真实数据中除了定义的字段外,还有三个`隐藏列`,分别是`row_id`(若没有主键就会增加该列),`transaction_id`,`roll_pointer`

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/隐藏列.png)


#### Dynamic,Compressed
> 这两种行格式对于存放`BLOB`中的数据采用了完全的`行溢出`的方式,在数据页中只存放20个字节的指针,实际的数据都存放在`OffPage`中
> `Compact`和`Redundant`两种格式会存放`768`个前缀字节,剩余部分存储在其他页中
> `Compressed`行记录格式会将存储在其中的行数据以`zlib`的算法进行压缩,对于`BLOB``TEXT``VARCHAR`这类大长度类型数据能够进行有效存储


`行溢出`:一个页大小一般是`16KB`,`16384字节`,VARCHAR类型的列最多可以存储`65535-2`个字节,这样一页存放不了一条记录称为`行溢出`

注:`65535-2` VARCHAR最多是`65535`,其中包含`2字节变长字段长度`+`1字节的非空标识`,实际可用`65535-3`字节
若字段标识为`NOT NULL`就会省略掉`1字节的非空标识`,实际可用`65535-2`字节

#### Redundant
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/Redundant行格式.png)

### 区、段、碎片区
P127
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/页的空间关系.png)


## 性能分析工具

### 分析流程和优化步骤
<div>
	<img src="https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/数据库性能分析过程1.png" width="70%" height=50%>
</div>

### 查看系统性能参数

```sql
# 查看(全局|当前会话)的系统性能参数,执行频率
SHOW [GLOBAL| SESSION] STATUS LIKE '参数';

```

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/常用MySQL性能参数.png)

### 使用last_query_cost

`SHOW STATUS LIKE 'last_query_cost';` 可以看到上一次查询使用到的成本(读取的页数量)

### 定位慢查询(慢查询日志)
慢查询日志,用来记录MySQL中`响应时间超过阈值`的语句,即运行时间超过`long_query_time`值的SQL,就会被记录到慢查询日志中
`long_query_time`默认值是`10`S
默认情况下,MySQL数据库没有开启慢查询日志,**若不是调优需要,一般不开启该参数**

#### 开启慢查询日志参数

```sql
# 查看慢查询日志是否开启
SHOW VARIABLES LIKE '%slow_query_log';

# 开启慢查询日志
set global slow_query_log='ON';

# 设置慢查询响应时间阈值,为了方便测试设置为1s
#测试发现：设置global的方式对当前session的long_query_time失效。对新连接的客户端有效。所以可以一并执行下述语句
set global long_query_time = 1;
show global variables like '%long_query_time%';

set long_query_time=1;
show variables like '%long_query_time%';

```

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/慢查询日志位置.png)

> 除了上述条件,还有`min_examined_row_limit`查询扫描过的最少记录数,若某个查询扫描过的记录数大于等于这个变量的值,并且查询执行时间超过`long_query_time`,那么这个查询就会被记录到慢查询日志中
> 默认值为`0`,表示只要查询执行时间超过`10s`,哪怕一个记录都没有扫描过,也会被记录到慢查询日志中

#### 慢查询分析mysqldumpslow

`mysqldumpslow --help` 查看mysqldumpslow的帮助信息

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/mysqldumpslow参数.png)

示例:查看前五条sql `mysqldumpslow -s t -t 5 /var/lib/mysql/atguigu01-slow.log`

```bash
#得到返回记录集最多的10个SQL
mysqldumpslow -s r -t 10 /var/lib/mysql/atguigu-slow.log
#得到访问次数最多的10个SQL
mysqldumpslow -s c -t 10 /var/lib/mysql/atguigu-slow.log
#得到按照时间排序的前10条里面含有左连接的查询语句
mysqldumpslow -s t -t 10 -g "left join" /var/lib/mysql/atguigu-slow.log
#另外建议在使用这些命令时结合 | 和more 使用 ，否则有可能出现爆屏情况
mysqldumpslow -s r -t 10 /var/lib/mysql/atguigu-slow.log | more
```

#### 关闭慢查询日志

```
[mysqld]
slow_query_log=OFF

# 或者注释掉配置项
[mysqld]
#slow_query_log =OFF

```

临时关闭可以使用`SET GLOBAL slow_query_log=off;`

### show profile

```sql
#查看状态
show variables like 'profiling';
#开启
set profiling = 'ON';

# 查看查询成本
show profiles;

```
这个已经基本弃用


### EXPLAIN 分析
可以使用`EXPLAIN` 或`DESCRIBE`工具针对SQL语句进行分析,二者使用方法相同,结果也是相同的

MySQL 5.6.3以前只能 `EXPLAIN SELECT`;MYSQL 5.6.3以后就可以 `EXPLAIN SELECT，UPDATE，DELETE`
在5.7以前的版本中，想要显示 `partitions` 需要使用 `explain partitions` 命令；想要显示 `filtered` 需要使用 `explain extended` 命令。在5.7版本后，默认`explain`直接显示`partitions`和`filtered`中的信息。
`EXPLAIN SELECT * FROM student_info;`执行EXPLAIN时的语句并不会对数据库中的数据产生影响,例如 `UPDATE，DELETE`操作

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/explain属性.png)

| 列名          | 描述                                                     |
| ------------- | -------------------------------------------------------- |
| id            | 在一个大的查询语句中每个SELECT关键字都对应一个`唯一的id` |
| select_type   | SELECT关键字对应的那个查询的类型                         |
| table         | 表名                                                     |
| partitions    | 匹配的分区信息                                           |
| type          | 针对单表的访问方法                                       |
| possible_keys | 可能用到的索引                                           |
| key           | 实际上使用的索引                                         |
| key_len       | 实际使用到的索引长度                                     |
| ref           | 当使用索引列等值查询时，与索引列进行等值匹配的对象信息   |
| rows          | 预估的需要读取的记录条数                                 |
| filtered      | 某个表经过搜索条件过滤后剩余记录条数的百分比             |
| Extra         | 一些额外的信息                                           |

#### table
  - 查询的每一行记录都对应一个单表,若是关联查询就会出现多行记录对应多个表
  - EXPLAIN语句输出的每条记录都对应着某个单表的访问方法
#### id
  - 一个查询语句中每个select都对应一个唯一的id
  - 查询优化器可能对涉及子查询的语句进行重写,转变为多表查询,导致`explain`结果id只有一个
  - 使用`union` 查询会生成临时表,`explain`结果有三条记录,而`union all`不会生成临时表
  - id若相同,可以认为是一组,从上往下顺序执行
  - id值越大,优先级越高,越先执行
  - 每个id表示一趟独立查询
#### select_type
  - 每个`select`都代表一个小查询语句,每个小查询都有一个`select_type`代表其在整个查询中的角色
  - `SIMPLE`类型:查询语句中不包含`UNION`或子查询
  - `PRIMARY`类型:`UNION`的左边或者子查询(未被查询器优化)的最外层
  - `UNION`类型:出现在`UNION`关键字后的查询标记为`UNION`类型
  - `UNION RESULT`类型:`UNION`关键字生成的临时表的类型
  - `SUBQUERY` 相关子查询
  - `DEPENDENT SUBQUERY`
  - `DEPENDENT UNION`
  - `DERIVED`派生表对应的子查询
  - `MATERIALIZED`
#### partitions
  - 匹配的分区信息,非分区表,该项为null
#### type
  - 代表MySQL对某个表的`执行查询时的访问方法`,又称访问类型
  - `system`,`const`,`eq_ref`,`ref`,`fulltext`,`ref_or_null`,`index_merge`,`unique_subquery`,`index_subquery`,`range`,`index`,`ALL`
  - 至少要达到 range 级别，要求是 ref 级别，最好是 consts级别。
#### possible_keys和key
  - `possible_keys`对某个表执行`单表查询时可能使用到的索引`
  - `key`执行查询`真正使用的索引`
#### key_len
  - 实际使用的索引长度(字节数),是否充分利用了索引,值越大越好,主要针对联合索引
#### ref
  - 当使用索引列等值查询时,于索引列进行等值匹配的对象信息,包括常量,表的列,函数等
#### rows
  - 预估需要读取的记录条数
#### filtered
  - 某个表经过搜索条件过滤后剩余记录条数的百分比
#### Extra ⚠
  - 一些额外信息来`理解MySQL如何执行给定的查询语句`
  - `No tables used` 不使用表 `select 1`
  - `Impossible WHERE`where条件永远false(条件不生效) `where 1!=1`
  - `Using where` where条件生效(where是索引列不会显示)
  - `no matching min/max row`使用函数`min/max`并没有符合where条件的记录
  - `Using index` 查询列表和条件只包含某个索引列,会免去回表操作,高效率👍
  - `Using index condition` 索引条件下推
  - `Using join buffer`
  - `Using filesort`如对非索引列进行排序,低效率👎
  - `Using temporary`使用临时表,去重操作会出现(非索引列),低效率👎



### MySQL监控分析视图-sys schema

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/schema视图摘要.png)

```sql
#1. 查询冗余索引
select * from sys.schema_redundant_indexes;
#2. 查询未使用过的索引
select * from sys.schema_unused_indexes;
#3. 查询索引的使用情况
select index_name,rows_selected,rows_inserted,rows_updated,rows_deleted
from sys.schema_index_statistics where table_schema='dbname' ;

# 1. 查询表的访问量
select table_schema,table_name,sum(io_read_requests+io_write_requests) as io from
sys.schema_table_statistics group by table_schema,table_name order by io desc;
# 2. 查询占用bufferpool较多的表
select object_schema,object_name,allocated,data
from sys.innodb_buffer_stats_by_table order by allocated limit 10;
# 3. 查看表的全表扫描情况
select * from sys.statements_with_full_table_scans where db='dbname';


#1. 监控SQL执行的频率
select db,exec_count,query from sys.statement_analysis
order by exec_count desc;
#2. 监控使用了排序的SQL
select db,exec_count,first_seen,last_seen,query
from sys.statements_with_sorting limit 1;
#3. 监控使用了临时表或者磁盘临时表的SQL
select db,exec_count,tmp_tables,tmp_disk_tables,query
from sys.statement_analysis where tmp_tables>0 or tmp_disk_tables >0
order by (tmp_tables+tmp_disk_tables) desc;


#1. 查看消耗磁盘IO的文件
select file,avg_read,avg_write,avg_read+avg_write as avg_io
from sys.io_global_by_file_by_bytes order by avg_read  limit 10;

#1. 行锁阻塞情况
select * from sys.innodb_lock_waits;

```

## 💥索引优化和查询优化
- 索引失效,没有充分利用索引
- 关键查询太多
- 服务器参数调优
- 分库分表

> 大多数情况下默认采用`B+树`来构建索引,若是空间类型索引使用`R-树`,并且`MEMORY`表还支持`hash索引`
> 是否最终使用索引都是由优化器决定,根据SQL开销,决定是否使用索引,与数据库版本,数据量等都有关系

### 索引失效的情况　　

```sql
# 表结构例子
CREATE TABLE `student` (
 `id` INT(11) NOT NULL AUTO_INCREMENT,
 `stuno` INT NOT NULL ,
 `name` VARCHAR(20) DEFAULT NULL,
 `age` INT(3) DEFAULT NULL,
 `classId` INT(11) DEFAULT NULL,
 PRIMARY KEY (`id`)
 #CONSTRAINT `fk_class_id` FOREIGN KEY (`classId`) REFERENCES `t_class` (`id`)
) ENGINE=INNODB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

# 索引例子 3个

CREATE INDEX idx_age ON student(age);
CREATE INDEX idx_age_classid ON student(age,classId);
CREATE INDEX idx_age_classid_name ON student(age,classId,NAME);

```

- 优先全值匹配,优先使用最多个的联合索引,能用三个的不用两个的

```sql
# 这个语句优先使用idx_age_classid_name索引,此时另外两个索引失效
EXPLAIN SELECT SQL_NO_CACHE * FROM student WHERE age=30 AND classId=4 AND NAME = 'abcd';
```

- 左前缀法则,必须按照联合索引的顺序调用,用了第一个(最左边)才能用第二个,联合使用第`2,3`和`1,3`个都无效
  - 最左无法匹配,无法使用该联合索引

```sql
# 删除idx_age 和 idx_age_classid 的前提下
# 这个SQL 都无法使用 idx_age_classid_name 联合索引
EXPLAIN SELECT SQL_NO_CACHE * FROM student WHERE student.classid=1 AND student.name = 'abcd';

```

- 主键插入顺序

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/主键插入顺序.png)

- 计算,函数,类型转换(自动或手动)导致索引失效

```sql
# 计算和函数无法使用索引的情况
CREATE INDEX idx_name ON student(NAME);
CREATE INDEX idx_sno ON student(stuno);
#可以使用索引
EXPLAIN SELECT SQL_NO_CACHE * FROM student WHERE student.name LIKE 'abc%';
EXPLAIN SELECT SQL_NO_CACHE id, stuno, NAME FROM student WHERE stuno = 900000;
#无法使用索引
EXPLAIN SELECT id, stuno, NAME FROM student WHERE SUBSTRING(NAME, 1,3)='abc';
EXPLAIN SELECT SQL_NO_CACHE id, stuno, NAME FROM student WHERE stuno+1 = 900001;
EXPLAIN SELECT SQL_NO_CACHE * FROM student WHERE LEFT(student.name,3) = 'abc';

# 类型转换索引失效情况 int 转varchar
# 无效
EXPLAIN SELECT SQL_NO_CACHE * FROM student WHERE NAME = 123;
# 有效
EXPLAIN SELECT SQL_NO_CACHE * FROM student WHERE NAME = '123';
```

- 范围条件右边的列索引失效

```sql
# 1号索引 注意顺序
CREATE INDEX idx_age_classId_name ON student(age,classId,NAME);
# 2号索引 注意顺序
CREATE INDEX idx_age_name_cid ON student(age,NAME,classId);

# 能完全使用2号索引,无法使用全部的1号索引,最后一个 AND student.name = 'abc' 失效
EXPLAIN SELECT SQL_NO_CACHE * FROM student
WHERE student.age=30 AND student.classId>20 AND student.name = 'abc' ;

# 可以完全2号索引,无法使用全部的1号索引
EXPLAIN SELECT SQL_NO_CACHE * FROM student
WHERE student.age=30 AND student.name = 'abc' AND student.classId>20;
```

> 有两个点,and的顺序优化器会自动选择最佳顺序,所以针对1号,2号索引两个sql是等效的
> 重点是索引的顺序,要把范围选择放在最后就能使用全部联合索引
> 包括 `<` `>` `<=` `>=` `between`等

- 不等于`!=` 或者`<>`索引失效
- `IS NULL`可以使用索引,`IS NOT NULL` `NOT LIKE`无法使用索引
  - 针对`NOT NULL`可以设置默认值0或者空字符串进行优化
- `like '%abc'` like以通配符开头,索引失效
  - `阿里开发`严禁左模糊,全模糊
- `OR` 前后存在任意`非索引`列,索引失效
- 数据库和表的字符集要统一,最好统一为`utf8mb4`

> - 单列索引尽量选择针对当前query过滤性更好的索引
> - 组合索引,滤性更好的索引越靠前越好
> - 组合索引,尽量包含当前query更多where条件
> - 组合索引,将可能出现范围查询的字段放在索引最后


### 关联查询优化

#### 左连接
左连接关键在于被驱动表(右边的表)的关联字段是否有索引,并且驱动表(左边的表)和被驱动表的关联字段类型要相同(避免类型转换导致索引失效)

```sql
CREATE INDEX X ON `type`(card);
CREATE INDEX Y ON book(card);
# 下面的语句都只会 使用book表的索引Y
EXPLAIN SELECT SQL_NO_CACHE * FROM `type` LEFT JOIN book ON type.card = book.card;
```

#### 内连接
- 对于内连接来说,查询优化器可以决定谁作为驱动表,谁作为被驱动表(类似`AND`连接)
  - 优先选择关联条件带索引的表作为`被驱动表`(右边)
  - 优先选择数据量小的作为驱动表(小表驱动大表)
  - 索引的优先级高于数据量大小的比较

> 连接查询驱动表(左边)不变,遍历驱动表的记录,去被驱动表中进行查询,所以被驱动表的索引才是影响性能的关键

#### JOIN原理

- 驱动表和被驱动表
  - 驱动表是主表,被驱动表是从表
  - 对于`内连接`来说,主表和从表是由查询优化器决定的
  - 对于`外连接`来说,一般情况下左边的作为主表,查询优化器也会影响
- `Simple Nested-Loop Join`简单嵌套循环连接(全表*全表)
- `Index Nested-Loop Join`索引嵌套循环连接 (被驱动表关联字段添加索引)
- `Block Nested-Loop Join`块嵌套循环连接 (将驱动表部分数据列加入缓存)`8.0`被废弃

结论:
- 小结果集驱动大结果集(限制条件加持后的结果集)
- 为被驱动表匹配的条件添加索引
- 需要JOIN 的字段，数据类型保持绝对一致
- LEFT JOIN 时，选择小表作为驱动表， 大表作为被驱动表 。减少外层循环的次数
- INNER JOIN 时，MySQL会自动将 小结果集的表选为驱动表 。选择相信MySQL优化策略
- 增加`join buffer size`的大小,缓存更多数据
- 减少驱动表不必要的字段查询

> 从`8.0.18`开始,默认使用`hash join`,是针对`大数据集连接`时的常用方式



### 子查询优化

子查询效率比较低:
- 会创建临时表,消耗资源
- 临时表过大时查询效率下降,无法为其创建索引

> 尽量不要使用`NOT IN` `NOT EXITS` 用 `LEFT JOIN ON WHERE xxx IS NULL`代替

### 排序(ORDER BY)优化
MySQL中支持两种排序方式,分别是`FileSort`和`Index`,`Index`效率更高,`FileSort`尽量避免

> SQL 中,可以在`WHERE`子句和`ORDER BY`子句中使用索引,前者是为了避免`全表扫描`,后者是为了避免`FileSort`排序

```sql
CREATE  INDEX idx_age_classid_name ON student (age,classid,NAME);

#不限制,索引失效
EXPLAIN  SELECT SQL_NO_CACHE * FROM student ORDER BY age,classid;

#增加limit过滤条件，使用上索引了。
EXPLAIN  SELECT SQL_NO_CACHE * FROM student ORDER BY age,classid LIMIT 10;
```

- order by不限制条目数,索引失效
- order by时顺序错误，索引失效
- order by时规则不一致, 索引失效 （顺序错，不索引；方向反(desc)，不索引）

> 所有排序都是在条件过滤之后进行的,若使用条件过滤后只剩下小部分数据,就不需要使用排序索引,可以获得很高的查询效率
> 或者,同等条件下优先选择过滤条件的索引

### 分组(GROUP BY)优化

- group by 使用索引的原则几乎跟order by一致 ，group by 即使没有过滤条件用到索引，也可以直接使用索引。
- group by 先排序再分组，遵照索引建的最佳左前缀法则
- 当无法使用索引列，增大 max_length_for_sort_data 和 sort_buffer_size 参数的设置
- where效率高于having，能写在where限定的条件就不要写在having中了
- 减少使用order by，和业务沟通能不排序就不排序，**或将排序放到程序端去做**。`order by、groupby、distinct`这些语句较为耗费CPU，数据库的CPU资源是极其宝贵的。
- 包含了`order by、group by、distinct`这些查询的语句，where条件过滤出来的结果集请保持在1000行以内，否则SQL会很慢。


### 分页 limit 优化

- 在索引上完成分页排序操作,最后根据主键关联回原表查询其他列内容
- 对于自增id的表直接对id进行判断

### 覆盖索引
`索引列+主键` 包含 `SELECT 到 FROM`之间查询的列    
例:索引列为`c1,c2`那么若`select c1,c2 from table` 就不会进行回表操作,直接查询索引返回数据,包括`主键id`,这种情况称为覆盖索引    
同时`select c1 from table` `select id,c1,c2 from table` 也是

> 一个索引包含了满足查询结果的数据就叫覆盖索引

几个特殊案例来理解,针对之前`索引失效问题`

```sql
# 创建索引
CREATE INDEX idx_age_name ON student (age,NAME);
# 无法使用索引需要回表
EXPLAIN SELECT * FROM student WHERE age <> 20;
# 可以使用索引,不需要回表
EXPLAIN SELECT age,NAME FROM student WHERE age <> 20;

# 无法使用索引
EXPLAIN SELECT * FROM student WHERE NAME LIKE '%abc';
# 可以使用索引
EXPLAIN SELECT id,age FROM student WHERE NAME LIKE '%abc';
```

优点: 避免回表;可以将`随机IO`变为`顺序IO`加快效率    
缺点: 索引字段的维护的代价

### 索引下推ICP
`Index Condition Pushdown(ICP)` 是MySQL5.6中的新特性,`Extra`显示为`Using index condition`
索引下推真正发挥作用的地方在于:**针对失效索引的生效情况**   

```sql
# 联合索引(`zipcode`,`lastname`,`firstname`)

EXPLAIN SELECT * FROM people
WHERE zipcode='000001'
# 应该失效,但是可以使用索引下推,在回表之前进行过滤
AND lastname LIKE '%张%'
AND address LIKE '%北京市%';

# 下面这种虽然也是索引下推,但是并没有发挥真正索引下推的作用,实际上索引没有失效,理解
EXPLAIN SELECT * FROM people
WHERE zipcode='000001'
AND lastname LIKE '张%'
AND firstname LIKE '三%';
```

设置索引下推是否开启,默认开启:   
`SET optimizer_switch = 'index_condition_pushdown=on/off';`    
`SET  @@optimizer_switch='index_condition_pushdown=on/off';` 

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/索引下推使用条件.png)

### MRR(Multi-Range Read) 优化
`MySQL5.6` 开始支持`MRR`默认是开启状态, `MRR` 优化的目的是为了减少磁盘的随机访问,并且将随机访问转化为顺序访问  
是否启用`MRR`可以通过参数 `optimizer_switch` 中的属性 `mrr`和`mrr_cost_based`来进行配置   
`MRR`优化可适用于`range` `ref` `eq_ref`类型的查询

`SET  @@optimizer_switch='mrr=on,mrr_cost_based=on';`   

若设置为 `mrr=on,mrr_cost_based=on` 启用优化,`mrr_cost_based`标记表示是否通过`cost based(基于成本消耗)` 的方式来选择是否启用`MRR`    
若设置为 `mrr=on,mrr_cost_based=off` 总是启用优化    



### 其他查询优化策略

#### EXISTS和IN
核心标准要符合`小表驱动大表`

```sql
#适合A大B小 由B驱动A
SELECT * FROM A WHERE CC IN (SELECT CC FROM B)

#适合B大A小 由A驱动B
SELECT * FROM A WHERE EXISTS (SELECT CC FROM B WHERE B.CC=A.CC)

```

#### COUNT(*)与COUNT(字段)

`COUNT(*)` `COUNT(1)` `COUNT(字段)`比较

> `COUNT(*)` `COUNT(1)` 无区别,自动选择占用空间更小的二级索引,`MyISAM`引擎下复杂度为`O(1)`,因为有`row_count`值直接存储记录数,`InnoDB`下为`O(n)`需要全表扫描   
> `COUNT(字段)`要尽量采用二级索引,不要采用聚簇索引

#### SELECT*
- 无法使用覆盖索引
- 查询所有列名,耗时

#### LIMIT 1
针对的是会扫描全表的 SQL 语句，如果你可以确定结果集只有一条，那么加上 `LIMIT 1` 的时候，当找到一条结果的时候就不会继续扫描了，这样会加快查询速度。
如果数据表已经对字段建立了`唯一索引`，那么可以通过索引进行查询，不会`全表扫描`的话，就不需要加上 `LIMIT 1` 了


## 数据库设计

### 淘宝数据库设计参考
- `非核心业务`:对应表的主键自增ID，如告警、日志、监控等信息。
- `核心业务`:主键设计至少应该是全局唯一且是单调递增。全局唯一保证在各系统之间都是唯一的，单调递增是希望插入时不影响数据库性能。


### 范式
在关系型数据库中，关于数据表设计的基本原则、规则就称为`范式`。可以理解为，`一张数据表的设计结构需要满足的某种设计标准的级别` 。要想设计一个结构合理的关系型数据库，必须满足一定的范式,`Normal Form`简称`NF`
目前关系型数据库有六种常见范式，按照范式级别，从低到高分别是：第一范式（1NF）、第二范式（2NF）、第三范式（3NF）、巴斯-科德范式（BCNF）、第四范式(4NF）和第五范式（5NF，又称完美范式）
一般情况下,最高遵循到`BCNF`,`3NF`即可

- `超键`:能唯一标识元组的属性集叫做超键
- `候选键`:如果超键不包括多余的属性,那么这个超键就是候选键
- `主键`:用户可以从候选键中选择一个作为主键
- `外键`:R1表中的某个属性集是R2表的主键,这个属性集就叫做R1的外键
- `主属性`:包含在任一候选键中的属性称为主属性
- `非主属性`:与主属性相对,值得是不包含任何一个候选键的属性

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/键示例.png)


![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/范式关系.png)


- 第一范式
  - 确保数据表中的每个值字段都具有原子性,不能再进行拆分
  - 例:手机号码字段不能同时存两个手机号码,而应该是两条数据
  - 例:user_info 拆分为 username,password,address 等
- 第二范式
  - 满足数据表中每一条记录都是可唯一标识的,而且所有非主键字段必须完全依赖主键,不能只依赖主键的一部分
- 第三范式
  - 要求所有数据表中的非主键字段和主键字段直接相关,非主键字段不能依赖其他非主键字段
- 巴斯范式(三+)
  - 第三范式前提下,只有一个候选键或者每个候选键都是单属性,那么就是巴斯范式
- 第四范式
- 第五范式



### 反范式化

> 完全按照范式设计,可能会导致大量的关联查询,可以通过增加`冗余字段`来提高数据库性能
> 当`冗余信息有价值或者能大幅提高查询效率`的时候,可以采用反范式化


劣势:
多余存储信息,导致空间浪费
同步修改问题,更新频繁性能消耗
对于小数据量下,反范式没有性能优势


### ER模型
ER模型三要素: 实体,属性,关系

- `实体`: 数据对象,使用`矩形`表示,`强实体`不依赖其他实体的实体,`弱实体`
- `属性`:实体的特性,使用`椭圆形`表示
- `关系`:实体之间的关系,使用`菱形`表示

> 实体和属性的区分,可独立存在的是实体,不可再分的是属性

#### 细化举例

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/ER图简易理解.png)




### 优化策略
P159 P160

#### 硬件优化 
**服务器的硬件性能直接决定着MySQL数据库的性能。**硬件的性能瓶颈直接决定MySQL数据库的运行速度和效率。针对性能瓶颈提高硬件配置，可以提高MySQL数据库查询、更新的速度。  


#### 优化MySql参数
- `innodb_buffer_pool_size`：这个参数是Mysql数据库最重要的参数之一，表示InnoDB类型的`表和索引的最大缓存`。它不仅仅缓存`索引数据`，还会缓存`表的数据`。这个值越大，查询的速度就会越快。但是这个值太大会影响操作系统的性能。
- `key_buffer_size`：表示`索引缓冲区的大小`。索引缓冲区是所有的`线程共享`。增加索引缓冲区可以得到更好处理的索引（对所有读和多重写）。当然，这个值不是越大越好，它的大小取决于内存的大小。如果这个值太大，就会导致操作系统频繁换页，也会降低系统性能。对于内存在`4GB`左右的服务器该参数可设置为`256M`或`384M`。
- `table_cache`：表示`同时打开的表的个数`。这个值越大，能够同时打开的表的个数越多。物理内存越大，设置就越大。默认为2402，调到512-1024最佳。这个值不是越大越好，因为同时打开的表太多会影响操作系统的性能。
- `query_cache_size`：表示`查询缓冲区的大小`。可以通过在MySQL控制台观察，如果Qcache_lowmem_prunes的值非常大，则表明经常出现缓冲不够的情况，就要增加Query_cache_size的值；如果Qcache_hits的值非常大，则表明查询缓冲使用非常频繁，如果该值较小反而会影响效率，那么可以考虑不用查询缓存；Qcache_free_blocks，如果该值非常大，则表明缓冲区中碎片很多。MySQL8.0之后失效。该参数需要和query_cache_type配合使用。
- `query_cache_type`的值是0时，所有的查询都不使用查询缓存区。但是query_cache_type=0并不会导致MySQL释放query_cache_size所配置的缓存区内存。
  - 当query_cache_type=1时，所有的查询都将使用查询缓存区，除非在查询语句中指定`SQL_NO_CACHE`，如SELECT SQL_NO_CACHE * FROM tbl_name。 
  - 当query_cache_type=2时，只有在查询语句中使用`SQL_CACHE`关键字，查询才会使用查询缓存区。使用查询缓存区可以提高查询的速度，这种方式只适用于修改操作少且经常执行相同的查询操作的情况。
- `sort_buffer_size`：表示每个`需要进行排序的线程分配的缓冲区的大小`。增加这个参数的值可以提高`ORDER BY`或`GROUP BY`操作的速度。默认数值是2 097 144字节（约2MB）。对于内存在4GB左右的服务器推荐设置为6-8M，如果有100个连接，那么实际分配的总共排序缓冲区大小为100 × 6 ＝ 600MB。 
- `join_buffer_size = 8M`：表示`联合查询操作所能使用的缓冲区大小`，和sort_buffer_size一样，该参数对应的分配内存也是每个连接独享。
- `read_buffer_size`：表示`每个线程连续扫描时为扫描的每个表分配的缓冲区的大小（字节）`。当线程从表中连续读取记录时需要用到这个缓冲区。SET SESSION read_buffer_size=n可以临时设置该参数的值。默认为64K，可以设置为4M。 
- `innodb_flush_log_at_trx_commit`：表示`何时将缓冲区的数据写入日志文件`，并且将日志文件写入磁盘中。该参数对于innoDB引擎非常重要。该参数有3个值，分别为0、1和2。该参数的默认值为1。
  - 值为`0`时，表示`每秒1次`的频率将数据写入日志文件并将日志文件写入磁盘。每个事务的commit并不会触发前面的任何操作。该模式速度最快，但不太安全，mysqld进程的崩溃会导致上一秒钟所有事务数据的丢失。
  - 值为`1`时，表示`每次提交事务时`将数据写入日志文件并将日志文件写入磁盘进行同步。该模式是最安全的，但也是最慢的一种方式。因为每次事务提交或事务外的指令都需要把日志写入（flush）硬盘。
  - 值为`2`时，表示`每次提交事务时`将数据写入日志文件，`每隔1秒`将日志文件写入磁盘。该模式速度较快，也比0安全，只有在操作系统崩溃或者系统断电的情况下，上一秒钟所有事务数据才可能丢失。
- `innodb_log_buffer_size`：这是 InnoDB 存储引擎的`事务日志所使用的缓冲区`。为了提高性能，也是先将信息写入 Innodb Log Buffer 中，当满足 innodb_flush_log_trx_commit 参数所设置的相应条件（或者日志缓冲区写满）之后，才会将日志写到文件（或者同步到磁盘）中。
- `max_connections`：表示 允许连接到MySQL数据库的最大数量 ，默认值是 151 。如果状态变量connection_errors_max_connections 不为零，并且一直增长，则说明不断有连接请求因数据库连接数已达到允许最大值而失败，这是可以考虑增大max_connections 的值。在Linux 平台下，性能好的服务器，支持 500-1000 个连接不是难事，需要根据服务器性能进行评估设定。这个连接数 不是越大 越好 ，因为这些连接会浪费内存的资源。过多的连接可能会导致MySQL服务器僵死。
- `back_log`：用于`控制MySQL监听TCP端口时设置的积压请求栈大小`。如果MySql的连接数达到max_connections时，新来的请求将会被存在堆栈中，以等待某一连接释放资源，该堆栈的数量即back_log，如果等待连接的数量超过back_log，将不被授予连接资源，将会报错。5.6.6 版本之前默认值为 50 ， 之后的版本默认为 50 + （max_connections / 5）， 对于Linux系统推荐设置为小于512的整数，但最大不超过900。如果需要数据库在较短的时间内处理大量连接请求， 可以考虑适当增大back_log 的值。
- `thread_cache_size`：`线程池缓存线程数量的大小`，当客户端断开连接后将当前线程缓存起来，当在接到新的连接请求时快速响应无需创建新的线程 。这尤其对那些使用短连接的应用程序来说可以极大的提高创建连接的效率。那么为了提高性能可以增大该参数的值。默认为60，可以设置为120。
- `wait_timeout`：指定`一个请求的最大连接时间`，对于4GB左右内存的服务器可以设置为5-10。 
- `interactive_timeout`：表示服务器在关闭连接前等待行动的秒数。



## 事务
一组操作单元，使数据从一种状态变换到另一种状态
原则：保证所有事务都作为一个单元来执行，一个事务中有多个操作时，要么所有操作都提交、并物理保存，要么全部放弃，回滚为最初状态


### 事务的ACID特性
- 原子性 `atomicity`
  - 事务是一个原子，事务内的操作要么都成功，要么都失败
- 一致性 `consistency`
  - 事务执行前后，数据从一个`合法性状态` 变换到另一个`合法性状态`（转账：有人钱变多，必须有人钱变少、余额200，不能转出300）
- 隔离性`isolation`
  - 一个事务的执行不能被其他事务干扰，事务内部操作和其他事务是隔离的
- 持久性 `durability`
  - 一旦事务提交，造成的修改是永久的
  - 持久性是通过`事务日志`来保证的，包括`重做日志` `回滚日志`

> 总结：原子性是基础，隔离性是手段，一致性是约束，持久性是目的


### 事务的状态
- 活动的
- 部分提交的
- 失败的
- 中止的
- 提交的

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/事务的状态.png)


### 显式事务
1. `START TRANSACTION` 或者 `BEGIN` 显式的开启一个事务

```bash
mysql> BEGIN;
#或者
mysql> START TRANSACTION;
```

`START TRANSACTION `语句后可以使用修饰符,`READ ONLY` 只读事务,`READ WRITE`读写事务,`WITH CONSISTENT SNAPSHOT`启动一致性读

2. 一系列事务中的操作（主要是DML，不含DDL）
3. 提交事务`COMMIT`

```bash
# 提交事务。当提交事务后，对数据库的修改是永久性的。
mysql> COMMIT;
# 回滚事务。即撤销正在进行的所有没有提交的修改
mysql> ROLLBACK;
# 将事务回滚到某个保存点。
mysql> ROLLBACK TO [SAVEPOINT]
```

> 保存点:中间快照,回滚到流程中某一个位置

```sql
BEGIN;
UPDATE user3 SET balance = balance - 100 WHERE NAME = '张三';

UPDATE user3 SET balance = balance - 100 WHERE NAME = '张三';

SAVEPOINT s1;#设置保存点

UPDATE user3 SET balance = balance + 1 WHERE NAME = '张三';

ROLLBACK TO s1; #回滚到保存点 最后的+1操作失效
```

### 隐式事务
MySQL中有一个系统变量 `autocommit` ,代表自动提交事务,默认情况下执行一个DML操作都会自动提交事务

关闭事务
- 显式的的使用 START TRANSACTION 或者 BEGIN 语句开启一个事务。这样在本次事务提交或者回滚前会暂时关闭掉自动提交的功能。
- 把系统变量 autocommit 的值设置为 OFF ，

```sql
SET autocommit = OFF;
#或
SET autocommit = 0;
```

隐式提交事务的情况:
- DDL
- 隐式使用或者修改数据库中的表`ALERT USER` `CREATE USER`...
- 事务控制或者关于锁定的语句(上述的开启事务语句)
- autocommit 由 off 改为on时
- ...


### 事务隔离级别

#### 数据并发问题
- 脏写
  - SessionA修改了另一个未提交事务SessionB修改过的数据，那就意味着发生了脏写
- 脏读
  - SessionA读取了已经被SessionB更新但还没有被提交的字段。之后若SessionB回滚，SessionA读取的内容就是临时且无效的
- 不可重复读
  - SessionA读取了一个字段，然后SessionB更新了该字段。之后SessionA再次读取同一个字段，值就不同了。那就意味着发生了不可重复读。
- 幻读
  - SessionA从一个表中读取了一个字段,然后SessionB在该表中插入了一些新的行。之后,如果SessionA再次读取同一个表,就会多出几行。那就意味着发生了幻读

四种问题严重性排序： `脏写>脏读>不可重复读>幻读`


#### 四种隔离级别 
- `READ UNCOMMITTED`读未提交，在该隔离级别，所有事务都可以看到其他未提交事务的执行结果。不能避免脏读、不可重复读、幻读。
- `READ COMMITTED`读已提交，它满足了隔离的简单定义：一个事务只能看见已经提交事务所做的改变。这是大多数数据库系统的默认隔离级别（但不是MySQL默认的）。可以避免脏读，但不可重复读、幻读问题仍然存在。
- `REPEATABLE READ`可重复读，事务A在读到一条数据之后，此时事务B对该数据进行了修改并提交，那么事务A再读该数据，读到的还是原来的内容。可以避免脏读、不可重复读，但幻读问题仍然存在。这是`MySQL的默认隔离级别`。
- `SERIALIZABLE`可串行化，确保事务可以从一个表中读取相同的行。在这个事务持续期间，禁止其他事务对该表执行插入、更新和删除操作。所有的并发问题都可以避免，但性能十分低下。能避免脏读、不可重复读和幻读。

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/隔离级别.png)

> 隔离级别越高性能越差  
> `脏写`一定要避免,四种隔离级别下都可以避免脏读   

#### 查看/设置 默认隔离级别
`5.7.20`之前使用   
` SHOW VARIABLES LIKE 'tx_isolation';`   

`5.7.20`之后使用   
`SHOW VARIABLES LIKE 'transaction_isolation';`

通用使用  
`SELECT @@transaction_isolation;`

设置隔离级别

```sql
SET [GLOBAL|SESSION] TRANSACTION ISOLATION LEVEL 隔离级别; 
#或者(注意隔离级别格式的不同)
SET [GLOBAL|SESSION] TRANSACTION_ISOLATION = '隔离级别'
# GLOBAL 全局
# SESSION 当前会话
#其中，隔离级别格式：
> READ UNCOMMITTED/READ-UNCOMMITTED
> READ COMMITTED/READ-COMMITTED
> REPEATABLE READ/REPATABLE-READ
> SERIALIZABLE
```

### 事务日志

事务的4种特性: `原子性`,`一致性`,`隔离性`和`持久性`,其中`隔离性`由**锁机制**实现,而剩余的三种特性由事务的`redo日志`和`undo日志`来保证  

- `REDO LOG`重做日志,提供再写入操作,恢复提交事务修改的页操作,保证`持久性`
  - 记录物理操作，页号，偏移量信息
- `UNDO LOG`回滚日志,回滚行记录到某个特性版本,保证`原子性` `一致性`
  - 记录逻辑操作,执行Insert会生成一条Delete日志(记录每个修改操作的`逆操作`)


#### REDO LOG
InnoDB采用的机制是：先写日志(redo log),再写磁盘,只有日志写入成功,才算事务提交成功   

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/redolog.png)  

- redo日志降低了刷盘频率
- redo日志占用的空间非常小
- redo日志是顺序写入磁盘的
- 事务执行过程中redo log不断记录



#### REDO LOG 的组成 
- `重做日志的缓冲 (redo log buffer)` ，保存在内存中，是易失的
  - `redo log buffer` 默认大小为`16MB`,通过命令`show variables like '%innodb_log_bufer_size%'`进行查看
- `重做日志文件 (redo log file)` ，保存在硬盘中，是持久的
  - 默认存储在数据目录中,分别为两个`ib_logfile0` `ib_logfile1` 默认已经占用空间,所以看到的文件比较大

#### REDO LOG 流转过程
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/redo流转.png)

刷盘策略:`redo log `不是直接写入磁盘的,`Innodb引擎会先写入redo log buffer` 之后再以一定的频率刷入到真正的`redo log file`中   
InnoDB给出 `innodb_flush_log_at_trx_commit` 参数，该参数控制 commit提交事务时，如何将 `redo log buffer` 中的日志刷新到` redo log f`ile 中

- 设置为`0`：表示每次事务提交时不进行刷盘操作。每隔1s写入pagecache   
- 设置为`1`：表示每次事务提交时都将进行同步，**刷盘**操作(默认值) 保证写入(持久化)   
- 设置为`2`：表示每次事务提交时都只把 redo log buffer 内容写入page cache`文件系统缓存`，不进行同步。由os自己决定什么时候同步到磁盘文件,效率高

> 区别: `0` 完全固定刷新频率1s1次可能丢失1s数据,   
> `1`每次事务提交都会进行同步刷盘,保证事务执行完肯定持久化,   
> `2`是0和1的中间态,只保证写入系统缓存,持久化操作由系统调配

#### REDO LOG Buffer过程

`Mini-Transaction` 简称 `mtr`一个语句包含一组 `mtr`,一个`mtr`包含若干条redo日志

#### Redo Log File相关参数
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/redolog参数.png)


#### Undo 日志 

`redo log`是**事务持久性**的保证,`undo log`是**事务原子性**的保证,在事务中 `更新数据`的前置操作是写入一个`undo log`     
`undo log`操作也会产生 `redo log` 

#### Undo 日志的作用
- `回滚数据`,用户将数据库逻辑恢复为原来的样子,理解为数据恢复,但是底层数据结构或者数据页发生了变化
- `MVCC 多版本并发控制`, 在InnoDB引擎中MVCC的实现是通过undo来完成  

#### 后续待完善

p172

#### Redo 和 Undo log加持总结

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/redolog和undolog流程.png)


## 锁

事务的`隔离性`由锁来实现   
`锁`是计算机协调多个进程或者线程`并发访问某一资源`的机制,`锁`为实现MySQL的各个隔离级别提供了保证  

并发事务访问情况：
- 读-读,并发读无影响
- 写-写,容易发生脏写,脏写不允许发生
- 读-写,可能发生脏读,不可重复读,幻读   

并发问题的解决方案:
- 读操作利用多版本并发控制`MVCC`,写操作进行加锁
- 读、写操作都采用加锁的方式


### 操作类型-读(共享)锁/写(排他)锁 
InnoDB实现了两种标准的行级锁   

- `读锁`默认为`共享锁`,也可以添加X锁成为排他锁 `S Lock` 兼容 S锁,不兼容X锁
- `写锁`即`排他锁` `X Lock` 不兼容 S锁和 X锁

```sql
# S锁
SELECT ... FROM ... LOCK IN SHARE MODE;

#8.0之后新增的S锁的写法
SELECT ... FROM ... FOR SHARE;

# X锁
SELECT ... FROM ... FOR UPDATE;
```

> 5.7及之前若使用X锁,获取不到锁就会一直进行等待,直到`innodb_lock_wait_timeout`超时   
> 8.0后可以在SQL中添加后缀 `NOWAIT` `SKIP LOCKED`跳过锁等待或被锁定行   
> `NOWAIT`会立即报错返回   
> `SKIP LOCKED` 会立即返回未被锁定的行   

### 操作粒度- 表锁,页锁,行锁  

#### 表锁
表锁一般使用在`MyISAM`引擎中
- 表共享锁`Table Read Lock` S锁
- 表独占锁`Table Write Lock` X锁

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/MyISAM读写锁.png)

##### 意向锁
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/意向锁.png)


> 意向锁要解决的问题: 若我们给某一行数据加上了排他锁,数据库会自动给更大一级的空间,如数据页或者数据表加上意向锁,告诉其他人这个数据页或数据表已经有人上过排他锁了,这样当有人想要获取数据表排他锁的时候,只需了解是否已经有人获取了这个数据表的意向排他锁即可    

- 意向锁是为了协调行锁和表锁的关系,支持多粒度(表锁和行锁)共存
- 意向锁是一种 不与行锁冲突的表锁
- 表明某个事务正在某些行有锁或者该事务准备去持有锁  

> 个人理解,`意向锁` 表明该表有行锁存在,省去判断行锁  

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/意向锁对比.png)


##### 自增锁 AUTO-INC锁
表锁


##### 元数据锁 MDL锁

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/意向锁对比.png)



#### 行锁

- 记录锁`Record Lock`
  - 一个事务获取了一条记录的S记录锁后，其他事务也可以获取S锁，但是不能获取X锁
  - 一个事务获取了一条记录的X锁后，其他事务不能获取S锁和X锁


- 间隙锁`Gap Lock`  

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/间隙锁.png)

gap锁的提出仅仅是为了防止插入幻影记录而提出的

- 临键锁 
  - 记录锁+间隙锁`(3,8]`  

```sql
begin;
select * from student where id <=8 and id > 3 for update;
```

- 插入意向锁 

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/插入意向锁.png)


#### 页锁

> `页锁`就是在页的粒度上进行锁定，锁定的数据资源比行锁要多，因为一个页中可以有多个行记录。页锁的开销介于`表锁`和`行锁`之间，会出现`死锁`。锁定粒度介于`表锁`和`行锁`之间，并发度一般


#### 锁升级
锁数量有限，锁会占用内存空间，某个层级锁数量过多时就会进行锁升级，`锁升级`就是用更大粒度的锁替代多个更小粒度的锁    



### 加锁方式 隐式锁,显式锁

#### 隐式锁  

`隐式锁`来保护插入记录还未提交事务的数据不被其他事务访问  

#### 显式锁

正常的加锁方式,称为显式锁 

### 全局锁

> `全局锁`就是对`整个数据库实例`加锁。当你需要让整个库处于只读状态的时候，可以使用这个命令，之后其他线程的以下语句会被阻塞：数据更新语句（数据的增删改）、数据定义语句（包括建表、修改表结构等）和更新类事务的提交语句。全局锁的典型使用 场景 是：做 `全库逻辑备份`  



### 死锁 
当出现死锁以后，有`两种策略`:   

- 一种策略是，直接进入等待，直到超时。这个超时时间可以通过参数 `innodb_lock_wait_timeout`来设置
- 发起死锁检测，发现死锁后，主动回滚死锁链条中的某一个事务（将持有最少行级排他锁的事务进行回滚），让其他事务得以继续执行。将参数 `innodb_deadlock_detect` 设置为on ，表示开启这个逻辑


## MVCC 多版本并发控制 

MVCC `Multiversion Concurrency Control` 通过数据行的多个版本管理来实现数据库的并发控制,
为了提高数据库并发性能,不加锁来处理`读写冲突`,可以理解为一种乐观锁.MySQL中只有InnoDB引擎支持MVCC


### 快照读和当前读
`快照读` 读取的是快照数据,不加锁的简单select语句都是`快照读`,快照读实现基于MVCC,避免加锁操作,提高并发性能  
**快照读,不一定是最新数据,可能是之前的历史版本**  

> 在`SERIALIZABLE`可串行化的隔离级别下,快照读会退化成当前读


`当前读` 读取的是最新的数据,读取时要保证其他并发事务不能修改当前记录,读取时会对记录进行加锁,加锁的select 或者 增删改语句都会进行当前读 


### 隔离级别的理解
`REPEATABLE READ`可重复读 是默认的隔离级别,解决了`脏读`和`不可重复读`问题,在`MVCC`加持下也解决了`幻读`的问题


### 隐藏字段,Undo Log版本链

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/Undo隐藏字段.png)

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/undo版本链.png)

### READ VIEW
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/readView.png)

使用 `READ COMMITTED` 和 `REPEATABLE READ` 隔离级别的事务，都必须保证读到 已经提交了的 事务修改
过的记录。假如另一个事务已经修改了记录但是尚未提交，是不能直接读取最新版本的记录的，核心问
题就是需要判断一下版本链中的哪个版本是当前事务可见的，这是ReadView要解决的主要问题。 

**readview 内容**  
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/readview内容.png)

**readview 规则**  
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/readView规则.png)



## 日志 

MySQL有不同类型的日志文件，用来存储不同类型的日志，分为`二进制日志`、`错误日志` 、`通用查询日志`
和 `慢查询日志`，这也是常用的4种。MySQL 8又新增两种支持的日志：`中继日志`和 `数据定义语句日志` 

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/6类日志.png)

> 除二进制日志外,其他日志都是`文本文件`,可以直接访问  

- 缺陷
  - 日志功能会 `降低MySQL数据库的性能`
  - 日志会 `占用大量的磁盘空间` 

### 通用查询日志

通用查询日志用来 `记录用户的所有操作` ，包括启动和关闭MySQL服务、所有用户的连接开始时间和截止时间、发给 MySQL 数据库服务器的所有 SQL 指令等。当我们的数据发生异常时，查看通用查询日志，还原操作时的具体场景

**配置:** 

```ini
# 通过my.cnf/my.ini进行配置
[mysqld]
general_log=ON
general_log_file=[path[filename]] #日志文件所在目录路径，filename为日志文件名

```

```sql
#临时配置
#查看状态
SHOW VARIABLES LIKE 'general_log%';

SET GLOBAL general_log=on;  # 开启通用查询日志

SET GLOBAL general_log_file=’path/filename’; # 设置日志文件保存位置

SET GLOBAL general_log=off;  # 关闭通用查询日志

```


### 错误日志
错误日志是默认开启的,而且无法关闭,错误日志默认在MySQL数据库的数据文件夹下,名称为`mysqld.log`   
若要修改文件名需要在 `my.cnf/my.ini`中进行配置

```ini
[mysqld]
log-error=[path/[filename]] #path为日志文件所在的目录路径，filename为日志文件名
```

```sql
# 查询错误日志的存储路径
SHOW VARIABLES LIKE 'log_err%';
```

### 二进制日志(bin log)

`binlog` 即 `binary log`,记录了数据库所有执行的DDL 和DML等**数据库更新事件**的语句,但是不包含没有修改任何数据的语句(如:select,show)  
它以事件的形式记录并保存在二进制文件中,通过这些信息,我们可以再现数据更新操作的全过程

主要应用场景:
- `数据恢复`,数据库意外停止可以通过二进制日志文件进行恢复  
- `数据复制`,master将二进制日志传递给slaves,来达到数据同步

#### 查看默认情况  

```sql
show variables like '%log_bin%';
```

`5.7`版本中是默认关闭的   
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/5.7binlog默认配置.png)
 
`8.0`版本中是默认开启的   
![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/8.0binlog默认配置.png)


#### 日志参数设置
修改MySQL的 `my.cnf` 或 `my.ini` 文件可以设置二进制日志的相关参数  

```ini
[mysqld]
#启用二进制日志
log-bin=atguigu-bin
binlog_expire_logs_seconds=600
max_binlog_size=100M

# 设置带文件夹的bin-log日志存放目录
[mysqld]
log-bin="/var/lib/mysql/binlog/atguigu-bin"

```

#### 查看日志
当MySQL创建二进制日志文件时，先创建一个以“filename”为名称、以“.index”为后缀的文件，再创建一个以“filename”为名称、以“.000001”为后缀的文件

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/binlog查看日志.png)

下面命令将行事件以 伪SQL的形式 表现出来   
`mysqlbinlog -v "/var/lib/mysql/binlog/atguigu-bin.000002"`


#### 写入机制
> binlog的写入时机也非常简单，事务执行过程中，先把日志写到 `binlog cache` ，事务提交的时候，再把binlog cache写到binlog文件中。因为一个事务的binlog不能被拆开，无论这个事务多大，也要确保一次性写入，所以系统会给每个线程分配一个块内存作为`binlog cache`




### 中继日志(relay log)

**中继日志只在主从服务器架构的从服务器上存在**。从服务器为了与主服务器保持一致，要从主服务器读取二进制日志的内容，并且把读取到的信息写入 本地的日志文件 中，这个从服务器本地的日志文件就叫中继日志 。然后，从服务器读取中继日志，并根据中继日志的内容对从服务器的数据进行更新，完成主从服务器的 `数据同步` 。

搭建好主从服务器之后，中继日志默认会保存在从服务器的数据目录下。文件名的格式是： `从服务器名 -relay-bin.序号` 。中继日志还有一个索引文件： `从服务器名-relay-bin.index`，用来定位当前正在使用的中继日志

## 主从复制 

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/mysql主从复制.png)


主要场景：  
- 读写分离(针对读多写少的场景，主机写入,从机读取,涉及到数据一致性的问题)
- 数据备份(从机相当于备份 )
- 高可用性，宕机恢复


### 原理
`slave`会从`master`读取`binlog` 来进行数据同步,基于`binlog`来进行数据同步,在主从复制过程中,会基于`3个线程来操作`,一个主库线程,两个从库线程    

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/主从复制原理1.png)

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/主从复制原理2.png)


**步骤1**： Master 将写操作记录到二进制日志（ binlog ）。    
**步骤2**： Slave 将 Master 的binary log events拷贝到它的中继日志（ relay log ）；   
**步骤3**： Slave 重做中继日志中的事件，将改变应用到自己的数据库中。 MySQL复制是异步的且串行化的，而且重启后从 接入点 开始复制。

### 主从环境搭建

待实践并完善 

### 同步数据一致性问题

**主从同步的要求：**  
- 读库和写库的数据一致
- 写数据必须写到写库
- 读数据必须到读库（不一定）

> 主从同步的日志是二进制日志，它是一个文件，在进行`网络传输`的过程中就一定会存在`主从延迟`，这样就可能造成用户在从库上读取到的数据不是最新的数据，也就是主从同步中的`数据不一致问题`

主备延迟最直接的表现是，从库消费中继日志（relay log）的速度，比主库生产binlog的速度要慢。造成原因:    
- 从库的机器性能比主库要差
- 从库的压力大
- 复杂事务执行

如: 大批量的删除数据和插入数据    

### 如何减少主从延迟
1. 降低多线程大事务并发的概率，优化业务逻辑
2. 优化SQL，避免慢SQL， 减少批量操作 ，建议写脚本以update-sleep这样的形式完成。
3. 提高从库机器的配置 ，减少主库写binlog和从库读binlog的效率差。
4. 尽量采用 短的链路 ，也就是主库和从库服务器的距离尽量要短，提升端口带宽，减少binlog传输的网络延时。
5. 实时性要求的业务读强制走主库，从库只做灾备，备份


### 解决一致性问题
- 异步复制  

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/异步复制.png)


- 半同步复制

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/异步复制.png)

- 组复制 

**异步复制和半同步复制都无法最终保证数据的一致性问题**，半同步复制是通过判断从库响应的个数来决定是否返回给客户端，虽然数据一致性相比于异步复制有提升，但仍然无法满足对数据一致性要求高的场景，比如金融领域。`MGR` 很好地弥补了这两种复制模式的不足。  

`MGR（MySQL Group Replication）` 是 MySQL 在 5.7.17 版本中推出的一种新的数据复制技术，这种复制技术是基于 `Paxos` 协议的状态机复制


## 数据库备份

物理备份：备份数据文件，转储数据库物理文件到某一目录。物理备份恢复速度比较快，但占用空间比较大，MySQL中可以用 xtrabackup 工具来进行物理备份。    

逻辑备份：对数据库对象利用工具进行导出工作，汇总入备份文件内。逻辑备份恢复速度慢，但占用空间小，更灵活。MySQL 中常用的逻辑备份工具为 `mysqldump` 。逻辑备份就是 备份sql语句 ，在恢复的时候执行备份的sql语句实现数据库数据的重现。  

### mysqldump实现逻辑备份 

#### 备份文件存储在当前目录下
```bash 
mysqldump -uroot -p atguigu>atguigu.sql 
mysqldump -uroot -p atguigudb1 > /var/lib/mysql/atguigu.sql
```
#### 备份全部数据库

```
# 使用 --all-databases 或 -A 参数
mysqldump -uroot -pxxxxxx --all-databases > all_database.sql
mysqldump -uroot -pxxxxxx -A > all_database.sql
```

#### 备份部分数据库 

```
# --databases 或 -B 参数
mysqldump -uroot -p --databases atguigu atguigu12 >two_database.sql

mysqldump -uroot -p -B atguigu atguigu12 > two_database.sql
```

#### 备份部分表

```
# 备份atguigu 库中的book表  
mysqldump -uroot -p atguigu book> book.sql

# 备份atguigu 库中的book,account表
mysqldump -uroot -p atguigu book account > 2_tables_bak.sql

# 备份student表中id小于10的数据
mysqldump -uroot -p atguigu student --where="id < 10 " > student_part_id10_low_bak.sql
```

#### 排除部份表

```
mysqldump -uroot -p atguigu --ignore-table=atguigu.student > no_stu_bak.sql

```

#### 只备份结构/只备份数据

只备份结构的话可以使用 `--no-data` 简写为 `-d` 选项；    
只备份数据可以使用 `--no-create-info` 简写为`-t` 选项   

```
# 仅结构
mysqldump -uroot -p atguigu --no-data > atguigu_no_data_bak.sql  

# 仅数据
mysqldump -uroot -p atguigu --no-create-info > atguigu_no_create_info_bak.sql
```

#### 备份中包含存储过程、函数、事件

mysqldump 备份默认是**不包含存储过程，自定义函数及事件的**。可以使用 `--routines` 或 `-R` 选项来备份存储过程及函数，使用 `--events` 或 `-E` 参数来备份事件

```
mysqldump -uroot -p -R -E --databases atguigu > fun_atguigu_bak.sql  
```


### mysql命令恢复数据

#### 单库备份中恢复单库

```
#sql中包含建表语句
mysql -uroot -p < atguigu.sql

# sql 中不包含建表语句
mysql -uroot -p atguigu4< atguigu.sql

```

#### 全量备份恢复

```
mysql –u root –p < all.sql

mysql -uroot -pxxxxxx < all.sql
```

### 物理备份和恢复  
备份: 直接将MySQL中的数据库文件复制出来。这种方法最简单，速度也最快     
恢复:将备份的数据库数据拷贝到数据目录下，并重启MySQL服务器   

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/mysql数据文件位置.png)  


## 参考资料
> - [尚硅谷MySQL](https://www.bilibili.com/video/BV1iq4y1u7vj)
> - [快查图片地址](https://github.com/ABZ-Aaron/CheatSheets)



















