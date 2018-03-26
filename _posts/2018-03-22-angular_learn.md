---
layout: post
title: angular_learn
date: 2018-03-22
tag: 笔记
---

# angular学习笔记  


## angular cli  

````
angular cli 是便于angular开发的命令行接口,用于实现自动化开发工作。

# angular cli安装
npm install -g typescript #安装最新版typescript
npm install -g @angular/cli  #安装angular cli
ng version

# 用cli创建angualr应用
ng new my-app --option #my-app为应用程序名
--option可选项:
--skip-install:boolean #默认false,表示跳过 npm install
--skip-git:boolean     #默认false,是否初始化git仓库
--directory:string     #设置创建的目录名,默认为应用程序名
--prefix:string        #设置创建新组件时,组件选择器使用的前缀
--routing:booean       #默认false,是否新增带有路由信息模块并添加到根模块

#运行应用程序
cd my-app
ng serve

#创建新功能(组件,服务)到应用程序
ng generate class my-class
ng generate component my-component
ng generate directive my-directive
ng generate enum my-enum
ng generate module my-module
ng generate pip my-pip
ng generate service my-service

````

