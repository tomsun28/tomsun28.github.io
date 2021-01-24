---
layout: post
title: restful api 权限设计 - 快速搭建权限项目
date: 2020-12-12
tag: restful api auth
---

## restful api 权限设计 - 快速搭建权限项目   

  在现在主流的前后端分离的系统中，或者是专注于提供api服务的系统，如何保护好所提供的后端restful api，使得非常重要。保护api的方面很多，限流，防刷，校验可以说都是保护，今天来谈谈很重要的api的认证鉴权保护，大概的思路。需求两点:首先认证 -- 我们可以配置哪些api需要用户认证通过才能访问哪些可以直接访问，还有就是鉴权 -- 我们可以配置管理哪些api对于某个用户能调用哪些不能调用。  
  在上篇[restful api权限设计 - 初探](https://usthe.com/2020/11/restful-api-auth/)我们大致说到了要保护我们restful api的认证鉴权所需的方向点。多说无益，现在就一步一步来实战下基于springboot sureness来快速搭建一个完整功能的权限认证项目。  

<br>  

这里为了照顾到刚入门的同学，图文展示了每一步操作。有基础可直接略过。  

### 初始化一个springboot web工程  

在IDEA如下操作:  

![image1](/images/posts/bootstrap/springboot1.PNG)  

![image2](/images/posts/bootstrap/springboot2.PNG)  

![image3](/images/posts/bootstrap/springboot3.PNG)  

![image4](/images/posts/bootstrap/springboot4.PNG)  

![image5](/images/posts/bootstrap/springboot5.PNG)  

### 提供一些模拟的restful api  

新建一个controller, 在里面实现一些简单的restful api供外部测试调用  

````
/**
 * simulate api controller, for testing
 * @author tomsun28
 * @date 17:35 2019-05-12
 */
@RestController
public class SimulateController {

    /** access success message **/
    public static final String SUCCESS_ACCESS_RESOURCE = "access this resource success";

    @GetMapping("/api/v1/source1")
    public ResponseEntity<String> api1Mock1() {
        return ResponseEntity.ok(SUCCESS_ACCESS_RESOURCE);
    }

    @PutMapping("/api/v1/source1")
    public ResponseEntity<String> api1Mock3() {
        return ResponseEntity.ok(SUCCESS_ACCESS_RESOURCE);
    }

    @DeleteMapping("/api/v1/source1")
    public ResponseEntity<String> api1Mock4() {
        return ResponseEntity.ok(SUCCESS_ACCESS_RESOURCE);
    }

    @GetMapping("/api/v1/source2")
    public ResponseEntity<String> api1Mock5() {
        return ResponseEntity.ok(SUCCESS_ACCESS_RESOURCE);
    }

    @GetMapping("/api/v1/source2/{var1}/{var2}")
    public ResponseEntity<String> api1Mock6(@PathVariable String var1, @PathVariable Integer var2 ) {
        return ResponseEntity.ok(SUCCESS_ACCESS_RESOURCE);
    }

    @PostMapping("/api/v2/source3/{var1}")
    public ResponseEntity<String> api1Mock7(@PathVariable String var1) {
        return ResponseEntity.ok(SUCCESS_ACCESS_RESOURCE);
    }

    @GetMapping("/api/v1/source3")
    public ResponseEntity<String> api1Mock11(HttpServletRequest request) {
        return ResponseEntity.ok(SUCCESS_ACCESS_RESOURCE);
    }

}
````

### 项目中加入sureness依赖  

在项目的pom.xml加入sureness的maven依赖坐标    
```
<dependency>
    <groupId>com.usthe.sureness</groupId>
    <artifactId>sureness-core</artifactId>
    <version>0.4.3</version>
</dependency>
```
如下：  

![image6](/images/posts/bootstrap/sureness1.PNG)  

### 使用默认配置来配置sureness    

新建一个配置类，创建对应的sureness默认配置bean  
sureness默认配置使用了文件数据源`sureness.yml`作为账户权限数据源  
默认配置支持了`jwt, basic auth, digest auth`认证  
```
@Configuration
public class SurenessConfiguration {

    /**
     * sureness default config bean
     * @return default config bean
     */
    @Bean
    public DefaultSurenessConfig surenessConfig() {
        return new DefaultSurenessConfig();
    }

}
```

### 配置默认文本配置数据源   

认证鉴权当然也需要我们自己的配置数据:账户数据，角色权限数据等  
这些配置数据可能来自文本，关系数据库，非关系数据库  
我们这里使用默认的文本形式配置 - sureness.yml, 在resource资源目录下创建sureness.yml文件  
在sureness.yml文件里配置我们的角色权限数据和账户数据，如下：  

````
## -- sureness.yml文本数据源 -- ##

# 加载到匹配字典的资源,也就是需要被保护的,设置了所支持角色访问的资源
# 没有配置的资源也默认被认证保护,但不鉴权
# eg: /api/v1/source1===get===[role2] 表示 /api/v2/host===post 这条资源支持 role2这一种角色访问
# eg: /api/v1/source2===get===[] 表示 /api/v1/source2===get 这条资源支持所有角色或无角色访问 前提是认证成功
resourceRole:
  - /api/v1/source1===get===[role2]
  - /api/v1/source1===delete===[role3]
  - /api/v1/source1===put===[role1,role2]
  - /api/v1/source2===get===[]
  - /api/v1/source2/*/*===get===[role2]
  - /api/v2/source3/*===get===[role2]

# 需要被过滤保护的资源,不认证鉴权直接访问
# /api/v1/source3===get 表示 /api/v1/source3===get 可以被任何人访问 无需登录认证鉴权
excludedResource:
  - /api/v1/account/auth===post
  - /api/v1/source3===get
  - /**/*.html===get
  - /**/*.js===get
  - /**/*.css===get
  - /**/*.ico===get

# 用户账户信息
# 下面有 admin root tom三个账户
# eg: admin 拥有[role1,role2]角色,明文密码为admin,加盐密码为0192023A7BBD73250516F069DF18B500
# eg: root 拥有[role1],密码为明文23456
# eg: tom 拥有[role3],密码为明文32113
account:
  - appId: admin
    # 如果填写了加密盐--salt,则credential为MD5(password+salt)的32位结果
    # 没有盐认为不加密,credential为明文
    # 若密码加盐 则digest认证不支持  
    credential: 0192023A7BBD73250516F069DF18B500
    salt: 123
    role: [role1,role2]
  - appId: root
    credential: 23456
    role: [role1]
  - appId: tom
    credential: 32113
    role: [role3]

````

### 添加过滤器拦截所有请求,对所有请求进行认证鉴权      

新建一个filter, 拦截所有请求，用sureness对所有请求进行认证鉴权。认证鉴权失败的请求sureness会抛出对应的异常，我们捕获响应的异常进行处理返回response即可。  

````
@Order(1)
@WebFilter(filterName = "SurenessFilterExample", urlPatterns = "/*", asyncSupported = true)
public class SurenessFilterExample implements Filter {

    @Override
    public void init(FilterConfig filterConfig) {}

    @Override
    public void destroy() {}

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain)
            throws IOException, ServletException {

        try {
            SubjectSum subject = SurenessSecurityManager.getInstance().checkIn(servletRequest);
            // 认证鉴权成功则会返回带用户信息的subject 可以将subject信息绑定到当前线程上下文holder供后面使用
            if (subject != null) {
                SurenessContextHolder.bindSubject(subject);
            }
        } catch (ProcessorNotFoundException | UnknownAccountException | UnsupportedSubjectException e4) {
            // 账户创建相关异常
            responseWrite(ResponseEntity
                    .status(HttpStatus.BAD_REQUEST).body(e4.getMessage()), servletResponse);
            return;
        } catch (DisabledAccountException | ExcessiveAttemptsException e2 ) {
            // 账户禁用相关异常
            responseWrite(ResponseEntity
                    .status(HttpStatus.UNAUTHORIZED).body(e2.getMessage()), servletResponse);
            return;
        } catch (IncorrectCredentialsException | ExpiredCredentialsException e3) {
            // 认证失败相关异常
            responseWrite(ResponseEntity
                    .status(HttpStatus.UNAUTHORIZED).body(e3.getMessage()), servletResponse);
            return;
        } catch (NeedDigestInfoException e5) {
            // digest认证需要重试异常
            responseWrite(ResponseEntity
                    .status(HttpStatus.UNAUTHORIZED)
                    .header("WWW-Authenticate", e5.getAuthenticate()).build(), servletResponse);
            return;
        } catch (UnauthorizedException e6) {
            // 鉴权失败相关异常，即无权访问此api
            responseWrite(ResponseEntity
                    .status(HttpStatus.FORBIDDEN).body(e6.getMessage()), servletResponse);
            return;
        } catch (RuntimeException e) {
            // 其他异常
            responseWrite(ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).build(),
                    servletResponse);
            return;
        }
        try {
            // 若未抛出异常 则认证鉴权成功 继续下面请求流程
            filterChain.doFilter(servletRequest, servletResponse);
        } finally {
            SurenessContextHolder.clear();
        }
    }

    /**
     * write response json data
     * @param content content
     * @param response response
     */
    private static void responseWrite(ResponseEntity content, ServletResponse response) {
        response.setCharacterEncoding("UTF-8");
        response.setContentType("application/json;charset=utf-8");
        ((HttpServletResponse)response).setStatus(content.getStatusCodeValue());
        content.getHeaders().forEach((key, value) ->
                ((HttpServletResponse) response).addHeader(key, value.get(0)));
        try (PrintWriter printWriter = response.getWriter()) {
            if (content.getBody() != null) {
                if (content.getBody() instanceof String) {
                    printWriter.write(content.getBody().toString());
                } else {
                    ObjectMapper objectMapper = new ObjectMapper();
                    printWriter.write(objectMapper.writeValueAsString(content.getBody()));
                }
            } else {
                printWriter.flush();
            }
        } catch (IOException e) {}
    }
}

````

像上面一样，
1. 若认证鉴权成功,`checkIn`会返回包含用户信息的`SubjectSum`对象  
2. 若中间认证鉴权失败，`checkIn`会抛出不同类型的认证鉴权异常,用户需根据这些异常来继续后面的流程(返回相应的请求响应)

为了使filter在springboot生效 需要在boot启动类加注解 @ServletComponentScan  

````
@SpringBootApplication
@ServletComponentScan
public class BootstrapApplication {

    public static void main(String[] args) {
        SpringApplication.run(BootstrapApplication.class, args);
    }
}
````

### 验证测试  

通过上面的步骤 我们的一个完整功能认证鉴权项目就搭建完成了，有些同学想 就这几步骤 它的完整功能体现在哪里啊 能支持啥。  

这个搭好的认证鉴权项目基于rbac权限模型，支持 baisc 认证，digest认证, jwt认证。能细粒度的控制用户对后台提供的restful api的访问权限，即哪些用户能访问哪些api。 我们这里来测试一下。  
IDEA上启动工程项目。  

1. basic认证测试  
认证成功：  

![image6](/images/posts/bootstrap/sureness2.PNG)  

密码错误：  

![image6](/images/posts/bootstrap/sureness3.PNG)  

账户不存在：  

![image6](/images/posts/bootstrap/sureness4.PNG)  

2. digest认证测试  

注意如果密码配置了加密盐，则无法使用digest 认证  

![image6](/images/posts/bootstrap/sureness5.PNG)  

![image6](/images/posts/bootstrap/sureness6.PNG)  

3. jwt认证测试  

jwt认证首先你得拥有一个签发的jwt，创建如下api接口提供jwt签发- `/api/v1/account/auth`  
````
@RestController()
public class AccountController {

    private static final String APP_ID = "appId";
    /**
     * account data provider
     */
    private SurenessAccountProvider accountProvider = new DocumentAccountProvider();

    /**
     * login, this provider a get jwt api, convenient to test other api with jwt
     * @param requestBody request
     * @return response
     *
     */
    @PostMapping("/api/v1/account/auth")
    public ResponseEntity<Object> login(@RequestBody Map<String,String> requestBody) {
        if (requestBody == null || !requestBody.containsKey(APP_ID)
                || !requestBody.containsKey("password")) {
            return ResponseEntity.badRequest().build();
        }
        String appId = requestBody.get("appId");
        String password = requestBody.get("password");
        SurenessAccount account = accountProvider.loadAccount(appId);
        if (account == null || account.isDisabledAccount() || account.isExcessiveAttempts()) {
            return ResponseEntity.status(HttpStatus.FORBIDDEN).build();
        }
        if (account.getPassword() != null) {
            if (account.getSalt() != null) {
                password = Md5Util.md5(password + account.getSalt());
            }
            if (!account.getPassword().equals(password)) {
                return ResponseEntity.status(HttpStatus.FORBIDDEN).build();
            }
        }
        // Get the roles the user has - rbac
        List<String> roles = account.getOwnRoles();
        long refreshPeriodTime = 36000L;
        // issue jwt
        String jwt = JsonWebTokenUtil.issueJwt(UUID.randomUUID().toString(), appId,
                "token-server", refreshPeriodTime >> 1, roles,
                null, Boolean.FALSE);
        Map<String, String> body = Collections.singletonMap("token", jwt);
        return ResponseEntity.ok().body(body);
    }


}
````

请求api接口登录认证获取jwt  

![image7](/images/posts/bootstrap/sureness7.PNG)  

携带使用获取的jwt值请求api接口  

![image8](/images/posts/bootstrap/sureness8.PNG)  

![image9](/images/posts/bootstrap/sureness9.PNG)  


4. 鉴权测试  

通过上面的sureness.yml文件配置的用户-角色-资源，我们可以关联下面几个典型测试点  
1. `/api/v1/source3===get`资源可以被任何直接访问，不需要认证鉴权  
2. `api/v1/source2===get`资源持所有角色或无角色访问 前提是认证成功  
3. 用户admin能访问`/api/v1/source1===get`资源,而用户root,tom无权限  
4. 用户tom能访`/api/v1/source1===delete`资源，而用户admin.root无权限  
测试如下：  

![image15](/images/posts/bootstrap/sureness10.PNG)  

![image10](/images/posts/bootstrap/sureness2.PNG)  

![image11](/images/posts/bootstrap/sureness11.PNG)  

![image12](/images/posts/bootstrap/sureness12.PNG)  

![image13](/images/posts/bootstrap/sureness13.PNG)  

![image14](/images/posts/bootstrap/sureness14.PNG)  


### 其他  

这次图文一步一步的详细描述了构建一个简单但完整的认证鉴权项目的流程，当然里面的授权账户等信息是写在配置文件里面的，实际的项目是会把这些数据写在数据库中。万变不离其宗，无论是写配置文件还是数据库，它只是作为数据源提供数据，基于sureness我们也能轻松快速构建基于数据库的认证鉴权项目，支持动态刷新等各种功能，这个就下次再写咯。  

若等不及下次文章，可以直接去看基于数据库的认证鉴权DEMO 仓库地址： https://github.com/tomsun28/sureness/tree/master/sample-tom  


<br>    

#### 源代码仓库    

这篇文章的完整DEMO代码仓库地址：https://github.com/tomsun28/sureness/tree/master/sample-bootstrap    








