---
layout: post
title: ubantu_learn
date: 2017-09-28
tag: linux
---

## about ubantu

**snap包管理器: **  

snap是一种全新的软件包管理方式，它类似一个容器拥有一个应用程序所有的文件和库，各个应用程序之间完全独立。所以使用snap包的好处就是它解决了应用程序之间的依赖问题，使应用程序之间更容易管理。但是由此带来的问题就是它占用更多的磁盘空间  

- sudo snap list 列出已经安装的snap包  
- sudo snap find testName 搜索要安装的snap包  
- sudo snap install testName 安装一个snap包  
- sudo snap refresh testName 更新一个snap包，如果你后面不加包的名字的话那就是更新所有的snap包  
- sudo snap revert testName 把一个包还原到以前安装的版本  
- sudo snap remove testName 删除一个snap包  
  ​          
  ​          

**dpkg命令用法：**

 - sudo dpkg -i test.deb  安装test软件包，一般是手动下载的Debian文件   
 - sudo dpkg -r test           卸载一个已安装的test软件包
 - sudo dpkg -l   列出已安装的软件信息

**apt-get命令用法** 
用于自动从互联网软件仓库中搜索，安装，升级，卸载软件或操作系统

 - sudo apt-get update  更新软件包列表，更新源 
 - sudo apt-get upgrade 更新已经安装的包
 - sudo apt-cache search package 搜索软件包
 - sudo apt-cache show package   获取对应的软件包相关信息
 - sudo apt-get install package    安装包列表中存在的对应的包
 - sudo apt-get install package --reinstall  重新安装
 - sudo apt-get remove package 删除包，有残余
 - sudo apt-get autoremove package 卸载一个已安装的软件包,删除包及其依赖 
 - sudo apt-get remove package --purge 卸载一个已安装的软件包及其配置文件等
 - sudo apt-get dist-upgrade 升级系统
 - sudo apt-cache depend package 了解该包依赖哪些包
 - sudo apt-cache rdepend package 查看该包被哪些包依赖
 - sudo apt-get build-dep package 安装包相关的编译环境
 - sudo apt-get autoclean 清理无用包
 - sudo add-apt-respository ppa:user/ppa-name 添加PPA源
 - sudo apt-get update 添加源之后更新
 - sudo add-apt-repository -r ppa:user/ppa-name 删除源,之后进人/etc/apt/sources.list.d目录将PPA源对应文件删除  

**ubuntu源**  
1. cd /ect/apt 查看是否有sources.list文件
2. sudo cp sources.list sources.list.bak 备份
3. sudo vi sources.list 编辑内容为其他ubuntu的源 eg: "deb http://archive.ubuntu.com/ubuntu precise main universe"  









<br>
<br>
<br>

*转载请注明* [from tomsun28](http://usthe.com)