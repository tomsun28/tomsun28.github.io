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

    adduser username   ubuntu下新增一个用户  
    passwd  username   给username修改密码  
    
    groupadd -g 502 groupname    添加一个用户组，组id为502
    useradd  -u 502 -g groupname username    新增一个用户username，将其添加到用户组groupname
    usermod -a -G groupname username 将一个已有用户添加到一个已有用户组，使得该用户组成为该用户的附加组 -a 代表append
        eg: usermod -a -G root username  将username添加到root组，使其有root权限 
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

**主机相关操作**

	reboot,shutdown -r now 立刻重启  
	shutdown -h now 立刻关机  
	shutdown -h 20:25 20:25关机  
	/etc/sysconfig/network 配置主机名hostname(redhat使用过)  


<br>

**网络IP等配置**  

---

```
/etc/sysconfig/network-scripts/ifcfg-eth*       (redhat,centos系列)对其修改配置ip  
一个网卡对应一个ifcfg-eth*文件  
```

```
DEVICE=eth0                         #网卡名      
HWADDR=00:0C:29:99:ED:C1            #mac地址  
TYPE=Ethernet                       #网卡类型  
UUID=ecb1517d-d450-4409-8289-1deed3224b96       #uuid  
ONBOOT=yes                          #开机上电自启动网络连接(yes/no)  
NM_CONTROLLED=yes  
BOOTPROTO=static                    #启用静态IP地址  若是dhcp则是动态获取ip后面就不需要填  
IPADDR=192.167.2.146                #ip地址  
NETMASK=255.255.255.0               #子网掩码  
GATEWAY=192.167.2.1                 #网关  
```
```service network restart  #重启网卡```  

---

```
/etc/network/interfaces                         (ubuntu)对其修改配置ip  
所有网卡对应一个interfaces文件  
```

```
auto ens192                       #要设置的网卡  
iface ens192 inet static          #设置静态IP；如果是使用自动IP用dhcp，后面的不用设置，一般少用  
address 192.167.2.144             #IP地址  
gateway 192.167.2.1               #子网掩码  
netmask 255.255.255.0             #网关  
dns-nameservers 192.162.76.9 27.26.90.3  #dns
```
```/etc/init.d/networking restart   #重启网卡```  

以上如果ifconfig还是找不到配置的网卡，重启虚拟机  

---

**配置各主机之间ssh节点互信(ssh直接连接不输入密码)**  

提前各节点添加对应hosts  

各个节点执行:  

```
mkdir ~/.ssh
chmod 755 .ssh 
/usr/bin/ssh-keygen -t rsa
/usr/bin/ssh-keygen -t dsa
```
将所有的key文件汇总到一个总的认证文件中，在节点1上执行:  
```
ssh node1 cat ~/.ssh/id_rsa.pub >> authorized_keys
ssh node2 cat ~/.ssh/id_rsa.pub >> authorized_keys
ssh node1 cat ~/.ssh/id_dsa.pub >> authorized_keys
ssh node2 cat ~/.ssh/id_dsa.pub >> authorized_keys
```
节点1上存在一份完整的key，拷贝到节点2:  
```
[root@node1 ~] cd ~/.ssh/
[root@node1 .ssh] scp authorized_keys node2:~/.ssh/
[root@node2 .ssh] chmod 600 authorized_keys
```
各个节点执行:  
```
ssh node1 date
ssh node2 date
```
检验是否配置成功，在节点1上不用输入密码就可以通过ssh连接节点2，说明配置成功  


**查看端口占用情况**

	netstat -apn
	netstat -apn|grep portnums

<br>

**查找**

	find / -name test*

<br>

**vim**  
gg 移动到文本头部   
GG 移动到文本尾部  
/ 查找字符串  
n 移动到查找的下一个  
N 移动到查找的上一个  

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

````

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

if [ $val == $val2 ]
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

````

**cat**  

````
#cat命令输出显示整个文件
cat file.log #显示打印整个file.log
````

**grep**  

````
#grep命令使用正则表达式搜索匹配文本,把匹配的行打印
cat file.log | grep '出现错误'    #grep将file.log中的有"出现错误"字段的行打印
grep "SQLSTATE"  *.log          #查找所有后缀名为.log的日志文件并且打印出含有字符串SQLSTATE 的行

````

**wc**  

````
#wc命令为统计指定文件的字节数,字数,行数。
wc -l #统计行数
wc -c #统计字节数
wc -w #统计字数
num=`cat file.log | grep '出现错误' | wc -l` #统计file.log中有'出现错误'字段的行数

````

**tr**

````
#tr命令对来自标准输入的字符进行替换
echo "HELLO TOM" | tr 'A-Z' 'a-z' #将输入字符由大写转换为小写

````

**linux遍历**  

````
filelist=`ls /home/temp*.sh`  #获取home目录下所有temp开头的.sh文件
for file in $filelist         #遍历执行
do
    sh $file 
done
````

**awk**

````
#awk强大的文本分析工具,把文件逐行读入,以给定的分隔符(默认为空格)将每行切片,切片的部分再进行各种分析处理
#awk适合对文本列操作
awk [-F filed-separator] 'commands' input-files
awk -F ":" 'print $2' temp.log           #显示输出用":"分隔符将temp.log分割后的第二列所有内容
cat /etc/passwd | awk -F ":" 'print $1'  #显示输出用":"分隔符将passwd分割后的第一列所有内容

````

**xargs**  
````
xargs能够捕获一个命令的输出，然后传递给另外一个命令
kubectl get po | grep tom-server | awk '{print $1}' | xargs kubectl delete po   # 批量删除tom-server*名称的pod  
````


**sed**  

````
#sed命令用来自动编辑文件,简化对文件的反复操作,对文件内容的编辑,替换,读取
#sed适合对文本行操作
sed [options] 'command' files
sed -n '1p' temp.log                          #打印temp.log的第一行
awk -F ":" 'print $2' temp.log | sed -n '1p'  #打印输出temp.log的第二列第一行的内容
sed -i "s/dddd/mmmm/g" /home/test.txt  #将test.txt中所有dddd替换成mmmm
sed -i "s/dddd/mmmm/" /hone/test.txt   #将test.txt的第一个dddd替换成mmmm


````

**set**  

````
#set :Linux set命令用于设置shell,能设置所用shell的执行方式依照不同的需求进行设置。
项目实例中主要是在创建控制文件前对shell的一些设置。
````

**spool**  

````
#spool是Oracle数据库交互工具SQLPULS的文件操作命令,其可以将屏幕显示及查询的结果输入到指定的文本文件
在项目实例中使用spool操作生成对应的控制文件
````

**tee**  

````
#tee :这个命令用于读取标准输入的数据,并将其内容输出到指定的文件中
tee –a file  #就是将获得的内容附加到文件file的后面
````

**ssh**  

````
#连接到远程主机
ssh userName@remoteServerIp
#连接到远程主机指定的端口22
ssh userName@remoteServerIp -p 22

````

<br>
<br>
<br>

*转载请注明* [from tomsun28](http://usthe.com)
