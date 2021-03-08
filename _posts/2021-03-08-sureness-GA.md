大家周一好，经过22个版本，2年多的迭代，很高兴激动宣布面向rest api的安全框架-sureness，正式GA啦。

## 📫 背景         

在主流的前后端分离架构中，如何通过有效快速的认证鉴权来保护后端提供的`restful api`变得尤为重要。对现存框架，不原生支持`rest`的`apache shiro`，
还是深度绑定`spring`，较慢性能，学习曲线陡峭的`spring security`，或多或少都不是我们的理想型。   
于是乎`sureness`诞生了，我们希望能解决这些，提供一个面向**restful api**，**无框架依赖**，可以**动态修改权限**，**多认证策略**，**更快速度**，**易用易扩展**的认证鉴权框架。      

## 🎡 介绍

> `sureness` 是我们在深度使用权限框架 `apache shiro` 之后,吸取其一些优点全新设计开发的一个认证鉴权框架  
> 面向 `restful api` 的认证鉴权,基于 `rbac` (用户-角色-资源)主要关注于对 `restful api` 的安全保护  
> 无特定框架依赖(本质就是过滤器处拦截判断,已有`springboot,quarkus,javalin,ktor`等集成样例)  
> 支持动态修改权限配置(动态修改配置每个`rest api`谁有权访问)    
> 支持 `websocket` ,主流`http`容器  `servlet` 和 `jax-rs`  
> 支持多种认证策略, `jwt, basic auth, digest auth` ... 可扩展自定义支持的认证方式   
> 基于改进的字典匹配树拥有的高性能    
> 良好的扩展接口, 样例和文档  

>`sureness`的低配置，易扩展，不耦合其他框架，希望能帮助开发者对自己的项目多场景快速安全的进行保护   

##### 🔍 框架对比     

| ~                | sureness         | shiro      | spring security |
| ---------------- | ---------------- | ---------- | --------------- |
| **多框架支持**   | 支持             | 需改动支持 | 不支持          |
| **restful api**  | 支持             | 需改动支持 | 支持            |
| **websocket**    | 支持             | 不支持     | 不支持          |
| **过滤链匹配**   | 优化的字典匹配树 | ant匹配    | ant匹配         |
| **注解支持**     | 支持             | 支持       | 支持            |
| **servlet**      | 支持             | 支持       | 支持            |
| **jax-rs**       | 支持             | 不支持     | 不支持          |
| **权限动态修改** | 支持             | 需改动支持 | 需改动支持      |
| **性能速度**     | 较快             | 较慢       | 较慢            |
| **学习曲线**     | 简单             | 简单       | 陡峭            |

##### 📈 基准性能测试  

![benchmark](https://gitee.com/tomsun28/sureness/raw/master/docs/_images/benchmark_cn.png)    

**基准测试显示sureness对比无权限框架应用损耗0.026ms性能，shiro损耗0.088ms,spring security损耗0.116ms，
相比之下sureness基本不消耗性能，且性能(参考TPS损耗)是shiro的3倍，spring security的4倍**     
**性能差距会随着api匹配链的增加而进一步拉大**     
详见[基准测试](https://github.com/tomsun28/sureness-shiro-spring-security-benchmark)    

##### ✌ 框架支持样例    

- sureness集成springboot样例(配置文件方案) [sample-bootstrap](https://github.com/tomsun28/sureness/tree/master/sample-bootstrap)   
- sureness集成springboot样例(数据库方案) [sample-tom](https://github.com/tomsun28/sureness/tree/master/sample-tom)  
- sureness集成quarkus样例 [sample-quarkus](https://github.com/tomsun28/sureness/tree/master/samples/quarkus-sureness)  
- sureness集成javalin样例 [sample-javalin](https://github.com/tomsun28/sureness/tree/master/samples/javalin-sureness)    
- sureness集成ktor样例 [sample-ktor](https://github.com/tomsun28/sureness/tree/master/samples/ktor-sureness)   
- sureness集成spring webflux样例 [sample-spring-webflux](https://github.com/tomsun28/sureness/tree/master/samples/spring-webflux-sureness)   
- more samples todo   

##### 项目仓库地址，欢迎使用，开源不易，觉得不错请大佬们star下给予鼓励，弯腰感谢。  

[GITHUB仓库地址](https://github.com/tomsun28/sureness)    
[GITEE仓库地址](https://gitee.com/tomsun28/sureness)            