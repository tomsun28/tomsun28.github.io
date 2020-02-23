---
layout: post
title:  http https 
date: 2020-02-23
tag: network
---
<br>

## 应用层 http https学习  

将之前在幕布整理的http https知识点再学习加深下   

### http版本  

* 1.0 - 每个请求都会触发TCP连接的建立和断开  
* 1.1 - 多个请求串行复用一个TCP连接  

### 请求方式  
* get - 获取内容  
* head - 仅获取报文头 
* post - 提交新增内容  
* put - 提交更新内容  
* delete - 删除内容  
* option - 查看服务器性能，设置选项  
* tarce - 回显服务器收到的请求，用于诊断测试   

### SSL_TLS  
* 对称加密  
> RSA (私钥解密-公钥加密)
> 非对称在TLS里面主要是身份校验，对称密钥的协商  

* 非对称加密  
> AES-CBC,DES
> 对数据传输内容的加密  

* 散列算法  
> MD5,SHA1(散列算法非加密算法)
> 数据完整性校验  

### https流程  

1. 客户端发起https请求  
2. 服务端返回CA证书(证书含公钥，公司信息，域名，有效期，证书颁发机构)  
3. 客户端解析证书，校验其合法性  
4. 合法性通过后，客户端取出证书里的公钥，生成随机码KEY，用公钥加密KEY发送给服务端  
5. 服务端使用私钥解密密码KEY，使用密码KEY对称加密传输数据  
6. 客户端使用密码KEY对称解密收到的加密数据内容  

### http状态码  

* 1XX - 通知  
  1. 101 - continue - 表示目前为止所有的内容正常  
  2. 102 - switching propotols 服务器正在根据发送的upgrade请求头切换到指定协议(http升级websockets)  

* 2XX - 成功  
  1. 200 - ok 成功处理请求  
  2. 201 - created 成功创建请求的资源  
  3. 202 - accepted 成功接受请求，请求之后会异步处理  
  4. 203 - non-authoriative information 请求处理成功，返回的内容由变换代理修改  
  5. 204 - no content 请求已成功，但客户端无需离开当前页面  

* 3XX - 重定向  
  1. 301 - multiple choices 指示所请求的资源已明确移动到location标题给定的url  
  2. 303 - see other 返回一个响应文档url，其可能是静态资源也可能是其他资源  

* 4XX - 客户端错误  
  1. 400 - bad request 来自客户端错误的请求，服务端不识别  
  2. 401 - unauthorized 没有认证信息或错误的认证信息  
  3. 403 - forbidden 服务器理解请求但拒绝授权，即没有权限  
  4. 404 - not found 服务器找不到请求的资源  
  5. 405 - method not allowed 服务器已经请求方法，但该已被禁用且无法使用  
  6. 406 - confict 请求与服务器当前状态冲突  

* 5XX -  服务端错误  
  1.  500 - internal server error 服务器执行请求异常  
  2.  501 - not implmented 请求方法不受服务器支持且无法处理  
  3.  502 - bad gateway 网关服务器收到来自上游服务器的无效响应(nginx)
  4.  503 - service unavailable 服务器尚未准备好处理请求  
  5.  504 - gateway timeout 网关服务器无法及时得到响应  
  6.  505 - http version not support 服务器不支持请求使用的http版本  

<br>
<br>

*转载请注明* [from tomsun28](http://usthe.com)
