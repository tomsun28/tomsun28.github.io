---
layout: post
title: linux_learn
date: 2017-10-19
tag: linux
---

## 遇到的linux日常知识点

**文件**

	source file 执行文件
	mv /test1/file1 /test2/file2 将file1复制到test2并将文件名改为file2
	mv /test1/file1 /test2/      将file1移动到test2目录下
	mv test.{txt,js}             修改test.txt为test.js
	mv test1 test2               修改test1名为test2
	file -i test.txt             查看文件的编码格式
	windows上编写的文件上传至linux,Windows默认编码GBK,linux为UTF-8,需转码 #iconv -f GBK -t UTF-8 test.txt -o test.txt

<br>

	unzip test.zip                     解压zip文件
	zip filename.zip test              压缩zip文件
	tar zxvf test.tar                  解压tar文件
	tar czvf filename.tar test         压缩tar文件
	tar zxvf test.tar.gz               解压tar.gz文件
	tar zcvf filename.tar.gz test      压缩tar.gz文件
	gunzip test.gz / gzip -d test.gz   解压gz文件
	gzip filename.gz test              压缩gz文件

<br>

**远程终端**

	export DISPLAY=显示终端的IP地址:0.0   在远程终端xbowser上显示
	export LANG=en_US                   在远程终端上显示防止出现乱码
	rz -be                         文件的远程传输

<br>

**用户,组**

	groupadd -g 502 groupname
	useradd  -u 502 -g groupname username
	usermod -a -G groupname username 将一个已有用户添加到一个已有用户组，使得该用户组成为该用户的附加组 -a 代表append 
	usermod -g username groupname 将username的主要用户组改为groupname
	gpasswd -d username groupname 将一个用户从某个组中删除，需要保证group不是用户的主组


<br>

**权限**

	ls -l temp  查看temp文件权限详细
	chown -R oracle temp 将temp整个文件夹授权给oracle用户
	chmod -R 444 filename 修改文件权限
	444 r--r--r--
	600 rw-------
	644 rw-r--r--
	666 rw-rw-rw-
	700 rwx------
	744 rwxr--r--
	755 rwxr-xr-x 
	777 rwxrwxrwx 
	chmod o+w temp 表示给其他人授予写temp文件的权限
	chmod go-rw temp 表示删除所有者所在群组和其他人对temp文件的读写权限
	+为增加,-为删除
	u代表所有者(user) g代表所有者所在的组群(group) o代表其他人,但不是u和g (other) a代表全部的人,也就是包括u,g和o
	
	r表示文件可以被读(read) w表示文件可以被写(write) x表示文件可以被执行



<br>

**重启**

	reboot,shutdown -r now

<br>

**查看端口占用情况**

	netstat -apn
	netstat -apn|grep portnums

<br>

**查找**

	find / -name test*

<br>

**环境变量**

	普通用户想用DB2用户的指令：更改普通用户所对应的环境变量PATH，在~/.bash_profile的PATH中添加db2对应的启动路径
	eg：
	. /home/db2inst1/sqllib/db2profile
	export PATH=$ORACLE_HOME/bin:$ORACLE_HOME/OPatch:/bin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/X11R6/bin:$HOME/bin:/home/db2inst1/sqllib/bin:/home/db2inst1/sqllib/adm:/home/db2inst1/sqllib/misc;

**FTP**

	ftp remote_path  //连接到远程IP
	ftp> status      //显示当前ftp状态
	ftp> get filename //复制远程的文件到本地
	ftp> put filename //复制本地文件到远程服务器
	ftp> quit        //关闭连接

**shell**  

```

#!/bin/bash

#shell参数传递
echo "第一个参数为：$1 "
echo "参数个数为：$# "
echo "传递的参数作为一个字符串显示：$* "

#shell基本运算
#expr 是表达式计算工具,使用它完成表达式求值操作
#变量名和等号之间不能有空格
val=`expr 3 + 3`  
val2=`expr $1 + $2`
echo "$val2  $val"

if[$val == $val2]
then
    echo "val等于val2";
else
    echo "val不等于val2"
fi
#文件测试判断运算
file="./test.sh"
if [ -r $file ]
then
   echo "文件可读"
else
   echo "文件不可读"
fi

#echo后面""可以显示转义字符,''原样输出
echo "\"hello tom\""  
echo "将内容定向到文件中" > filename 
echo `date` #显示命令结果

for loop in 1 2 3 4 5 6
do 
    echo "the number is : $loop " 
done

a=8;
while(($a<=10))
do
    echo $a 
	let "a++"   #let命令用于执行一个或多个表达式
done

command > file  #覆盖file文件原本的内容
command >> file #文件末尾累加输入,不会覆盖

source file.sh  #文件包含其他file文件内容

脚本执行：
$ chmod +x test.sh
$ ./test.sh 1 2 3

```

<br>
<br>
<br>

*转载请注明* [from tomsun28](http://usthe.com)