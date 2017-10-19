---
layout: post
title: oracle_learn
date: 2017-10-19
tag: oracle
---

## 遇到的 oracle 日常知识点

**round**

	返回一个按照指定的小数位进行四舍五入运算后的数值
	round(123.4567,0)  返回123
	round(123.4567,1)  返回123.5

**decode**

	decode(条件,值1,返回值1,值2,返回值2,....,值n,返回值n,缺省值)
	decode(table.id,'10',table.value/10,'100',table.value/100,0)

**case when**

	case flag
	when '0' then 'false'
	when '1' then 'true'
	else 'others' end
	
	case
	when flag = '0' then 'false'
	when flag = '1' then 'true'
	else 'others' end

**to_date,to_char,to_number**

	字符串时间互转
	to_date('2017-10-19 13-55-20','yyyy-mm-dd hh24:mi:ss')
	to_char(sysdate,'yyyy-mm-dd hh24:mi:ss')
	to_number(字符串,'格式')

**substr**

	字符串截取
	substr('my name is tomsun28',0,5)返回'my na' 从左端第一个字符开始截取长度为5的字符串 
	substr('my name is tomsun28',1,5)返回'my na' 0和1都表示从第一个字符开始截取
	substr('my name is tomsun28',-1,5)返回'sun28' 表示从右端第一个字符开始截取5个字符

**`||`**

	字符串连接符 'aaaa'||'bbbb' = 'aaaabbbb'

**<>**

	不等于的标准语法，可以移植到其他平台

**execute_immediate**

	动态SQL执行语句
	execute_immediate 动态sql语句 using 绑定参数列表 returning into 输出参数列表;
	execute_immediate 'select name,salary from temp where id = 1' using id returning into p_name,p_salary;

**truncate,delete,drop**

	delete from tablename; 一条一条的删记录，会保留一个空的页
	truncate table tablename; 一次性删掉整页，不保留任何页

**nvl**

	nvl(expr1,expr2) 若expr1为null，则返回expr2，不为null则返回expr1

**merge into**

	用一个或多个数据源完成对表的数据更新与插入,有则更新,无则插入
	merge into test1 t1
	using (select userid,id from test2 where id >= 20) tw
	on (t1.userid = tw.userid)
	when matched then update set t1.id = tw.id
	when mot matched then insert values(tw.userid,tw.id);

<br>


- - -
- - -

<br>

**误删除表空间test的数据文件后，需要将该表空间删除重建**

	alter database datafile '对应数据文件地址' offline drop; 
	drop tablespace test;

**在sqlplus下执行sql文件**

	SQL>@/home/oracle/test.sql         @@表示当前目录下执行

**数据库数据插入注意编码格式**

	查看文件编码格式  file -i test.txt

**oracle启动，关闭，查看监听服务**

	lsnrctl start,lsnrctl stop,lsnrctl status

**启动关闭oracle数据库实例**

	sqlplus / nolog
	conn / as sysdba
	conn username
	startup  这里环境变量export要设置正确的数据库SID，就是启动环境变量ORACLE_SID所对应的那个数据库实例
	startdown


<br>
<br>
<br>

*转载请注明* [from tomsun28](http://usthe.com)