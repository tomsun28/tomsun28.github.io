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

<br>
<br>
<br>

*转载请注明* [from tomsun28](http://usthe.com)