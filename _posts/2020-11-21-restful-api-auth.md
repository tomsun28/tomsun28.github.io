---
layout: post
title: jvm restful api auth
date: 2020-11-21
tag: restful api auth
---

## restful api 权限设计 - 初探一   

  在现在主流的前后端分离的系统中，或者是专注于提供api服务的系统，如何保护好所提供的后端restful api，使得非常重要。保护api的方面很多，限流，防刷，校验可以说都是保护，今天来谈谈很重要的api的认证鉴权保护，大概的思路。需求两点:首先认证 -- 我们可以配置哪些api需要用户认证通过才能访问哪些可以直接访问，还有就是鉴权 -- 我们可以配置管理哪些api对于某个用户能调用哪些不能调用。  
  题外-有些会说页面按钮或者页面显示的权限问题，个人认为按钮，页面这类的显示不显示，是前端的权限管理责任划分，所以这次并不会去讨论它。  


### 认证  

  主流的认证方式有 token - jwt, basic auth, digest auth等，用户携带认证信息访问restful api，后端会有个前置过滤拦截器(filter,interceptor,aop都可以实现)来拦截请求，获取请求信息里的认证信息，通过算法校验或者和系统内用户数据源(可以来自数据库，文本等)对比判断，校验通过才会让用户请求流程继续下去。  
  这是个大概的认证流程，还有很多细节需要考虑到，比如有些api不需要认证也要被访问，数据源的设计是怎么样的，用户携带多种认证信息怎么判断等。

### 鉴权  

  用户认证流程通过后，就需要对认证后的用户访问此api进行鉴权，判断此用户是否能访问此api。主流易用的权限模型是RBAC - 基于角色的权限模型。我们这里对restful api来说就是， api赋权给角色，用户拥有此角色，用户就能访问此restful api。即 用户-角色-资源。      
  我们将restful api请求视作一个资源,资源格式为: `requestUri===httpMethod`    即请求的路径加上其请求方式(`post,get,put,delete,patch`)作为一个整体被视作一个资源    `eg: /api/v2/book===get` `get`方式请求`/api/v2/book`接口数据。角色资源映射: 用户所属角色--角色拥有资源--用户拥有资源(用户就能访问此api)   
  上面这个流程大概可以抽象出这样的配置数据：
  对restful api来说，就是 `/api/v2/host===post===[role2,role3,role4]` 即对`/api/v2/host`进行post请求的这个rest api，其赋权给角色role2,role3,role4。
  对用户来说，就是 ` - appId: root, credential: 23456, role: [role1,role2]` 即用户root密码凭据是23456，拥有角色role1,role2。  
  当用户admin认证通过后， post方式请求接口/api/v2/host时，由于其拥有角色role2，而role2被赋权了`/api/v2/host===post`此资源，所以用户可以访问此api。  
  抽象数据ok，这些数据从哪里来也是问题。我们可以把它抽象为数据源，一般的数据源我们可以是数据库-5表结构即可，用户表-角色表-资源表和两个关系关联表。如果没有数据库的系统，可以是配置文本或者代码注解形式等。  

### 动态修改  

  对于权限配置数据，我们当然希望是可以动态修改而不是写死到代码里面。如果要做到动态修改，那使用spring aop类似的切面来认证鉴权就不合适了。数据源应该需要来自数据库或者配置文本，当数据源通过接口改动后，权限配置信息在内存中的值也需要同步修改。

### url路径匹配  

  认证鉴权之前还有个重要的一点就是路径匹配，即我们需要知道用户访问的restful api和我们从数据源获取的配置数据里面的api或者规则进行匹配，匹配成功的我们才知道这个api被赋权给了谁，这个api是否免登录等。当然你也可以使用类似spring aop一样的切面在方法调用前认证鉴权，这样就免去了路径匹配，但是这样的话就需要我们写死代码，做不到动态修改权限信息。api url路径匹配主流的就是ant匹配，看spring security 或者apache shiro都是用ant匹配规则，用请求的url和配置的链一个一个`ant`匹配。这其中是否还有更高效的方式呢。

<br>  
----   

上面就是对restful api 权限设计的粗略讨论，之后的文章会讨论下具体的实现细节，配置数据源为数据库时的表设计，jwt等各个认证方式实现，token刷新，路径过滤链匹配更优化解等。  

<br>    

#### 额外    
介绍一款简单高效面向restful api的jvm认证鉴权框架，shiro,spring security之外的第3个选择。  

**sureness**  [主页](https://su.usthe.com/) - https://su.usthe.com/   [github](https://github.com/tomsun28/sureness/) - https://github.com/tomsun28/sureness/  
>  基于 rbac 主要关注于对 restful api 的安全保护  
>  无特定框架依赖(本质就是过滤器处拦截判断,已有springboot,quarkus,javalin,ktor等demo)  
>  支持动态修改权限配置(动态修改哪些api需要被认证，可以被谁访问)    
>  支持主流http容器  servlet 和 jax-rs  
>  支持多种认证策略, `jwt, basic auth, digest auth` ... 可扩展自定义支持的认证方式 
>  基于改进的字典匹配树拥有的高性能  
>  良好的扩展接口, demo和文档  







