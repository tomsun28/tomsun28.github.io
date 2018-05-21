---
layout: post
title:  基于shiro的改造集成真正支持restful请求
date: 2018-04-22
tag: bootshiro+usthe项目
---

<br>
<br>

## 基于shiro的改造集成真正支持restful请求          

这个模块分离至上上上一篇[api权限管理系统与前后端分离实践](http://usthe.com/2018/04/api权限管理小系统与前后端分离实践/)，感觉那样太长了找不到重点，分离出来要好点。  

- - - -


首先说明设计的这个安全体系是是RBAC(基于角色的权限访问控制)授权模型，即```用户--角色--资源```，用户不直接和权限打交道，角色拥有资源，用户拥有这个角色就有权使用角色所用户的资源。所有这里没有权限一说，签发jwt里面也就只有用户所拥有的角色而没有权限。  

为啥说是真正的restful风格集成，虽说shiro对rest不友好但他本身是有支持rest集成的filter--```HttpMethodPermissionFilter```，这个shiro rest的 风格拦截器，会自动根据请求方法构建权限字符串```( GET=read,POST=create,PUT=update,DELETE=delete)```构建权限字符串;eg: ``` /users=rest[user]``` , 会 自动拼接出```user:read,user:create,user:update,user:delete```”权限字符串进行权限匹配(所有都得匹配，isPermittedAll)。  

但是这样感觉不利于基于jwt的角色的权限控制，在细粒度上验权url(即支持get,post,delete鉴别)就更没法了(个人见解)。打个比方：我们对一个用户签发的jwt写入角色列(role_admin,role_customer)。对不同request请求：```url="api/resource/",httpMethod="GET"```，```url="api/resource",httpMethod="POST"```,在基于角色-资源的授权模型中，这两个url相同的请求对HttpMethodPermissionFilter是一种请求，用户对应的角色拥有的资源url="api/resource"，只要请求的url是"api/resource"，不论它的请求方式是什么，都会判定通过这个请求，这在restful风格的api中肯定是不可取的，对同一资源有些角色可能只要查询的权限而没有修改增加的权限。    

可能会说在jwt中再增加权限列就好了嘛，但是在基于用户-资源的授权模型中，虽然能判别是不同的请求，但是太麻烦了，对每个资源我们都要设计对应的权限列然后再塞入到jwt中，对每个用户都要单独授权资源这也是不可取的。  

对shiro的改造这里自定义了一些规则:  
shiro过滤器链的url=```url+"=="+httpMethod```  
eg:对于```url="api/resource/",httpMethod="GET"```的资源,其拼接出来的过滤器链匹配url=```api/resource==GET```  
这样对相同的url而不同的访问方式，会判定为不同的资源，即资源不再简单是url，而是url和httpMethod的组合。基于角色的授权模型中，角色所拥有的资源形式为```url+"=="+httpMethod```。  
这里改变了过滤器的过滤匹配url规则，重写PathMatchingFilterChainResolver的getChain方法，增加对上述规则的url的支持。  

````
/* *
 * @Author tomsun28
 * @Description 
 * @Date 21:12 2018/4/20
 */
public class RestPathMatchingFilterChainResolver extends PathMatchingFilterChainResolver {

    private static final Logger LOGGER = LoggerFactory.getLogger(RestPathMatchingFilterChainResolver.class);

    public RestPathMatchingFilterChainResolver() {
        super();
    }

    public RestPathMatchingFilterChainResolver(FilterConfig filterConfig) {
        super(filterConfig);
    }

    /* *
     * @Description 重写filterChain匹配
     * @Param [request, response, originalChain]
     * @Return javax.servlet.FilterChain
     */
    @Override
    public FilterChain getChain(ServletRequest request, ServletResponse response, FilterChain originalChain) {
        FilterChainManager filterChainManager = this.getFilterChainManager();
        if (!filterChainManager.hasChains()) {
            return null;
        } else {
            String requestURI = this.getPathWithinApplication(request);
            Iterator var6 = filterChainManager.getChainNames().iterator();

            String pathPattern;
            boolean flag = true;
            String[] strings = null;
            do {
                if (!var6.hasNext()) {
                    return null;
                }

                pathPattern = (String)var6.next();

                strings = pathPattern.split("==");
                if (strings.length == 2) {
                    // 分割出url+httpMethod,判断httpMethod和request请求的method是否一致,不一致直接false
                    if (WebUtils.toHttp(request).getMethod().toUpperCase().equals(strings[1].toUpperCase())) {
                        flag = false;
                    } else {
                        flag = true;
                    }
                } else {
                    flag = false;
                }
                pathPattern = strings[0];
            } while(!this.pathMatches(pathPattern, requestURI) || flag);

            if (LOGGER.isTraceEnabled()) {
                LOGGER.trace("Matched path pattern [" + pathPattern + "] for requestURI [" + requestURI + "].  Utilizing corresponding filter chain...");
            }
            if (strings.length == 2) {
                pathPattern = pathPattern.concat("==").concat(WebUtils.toHttp(request).getMethod().toUpperCase());
            }

            return filterChainManager.proxy(originalChain, pathPattern);
        }
    }

}

````
重写PathMatchingFilter的路径匹配方法pathsMatch()，加入httpMethod支持。  

````
/* *
 * @Author tomsun28
 * @Description 重写过滤链路径匹配规则，增加REST风格post,get.delete,put..支持
 * @Date 23:37 2018/4/19
 */
public abstract class BPathMatchingFilter extends PathMatchingFilter {

    public BPathMatchingFilter() {

    }
    /* *
     * @Description 重写URL匹配  加入httpMethod支持
     * @Param [path, request]
     * @Return boolean
     */
    @Override
    protected boolean pathsMatch(String path, ServletRequest request) {
        String requestURI = this.getPathWithinApplication(request);
        // path: url==method eg: http://api/menu==GET   需要解析出path中的url和httpMethod
        String[] strings = path.split("==");
        if (strings.length <= 1) {
            // 分割出来只有URL
            return this.pathsMatch(strings[0], requestURI);
        } else {
            // 分割出url+httpMethod,判断httpMethod和request请求的method是否一致,不一致直接false
            String httpMethod = WebUtils.toHttp(request).getMethod().toUpperCase();
            return httpMethod.equals(strings[1].toUpperCase()) && this.pathsMatch(strings[0], requestURI);
        }
    }
}
````
这样增加httpMethod的改造就完成了，重写ShiroFilterFactoryBean使其使用改造后的chainResolver：RestPathMatchingFilterChainResolver  

````
/* *
 * @Author tomsun28
 * @Description rest支持的shiroFilterFactoryBean
 * @Date 21:35 2018/4/20
 */
public class RestShiroFilterFactoryBean extends ShiroFilterFactoryBean {

    private static final Logger LOGGER = LoggerFactory.getLogger(RestShiroFilterFactoryBean.class);

    public RestShiroFilterFactoryBean() {
        super();
    }

    @Override
    protected AbstractShiroFilter createInstance() throws Exception {
        LOGGER.debug("Creating Shiro Filter instance.");
        SecurityManager securityManager = this.getSecurityManager();
        String msg;
        if (securityManager == null) {
            msg = "SecurityManager property must be set.";
            throw new BeanInitializationException(msg);
        } else if (!(securityManager instanceof WebSecurityManager)) {
            msg = "The security manager does not implement the WebSecurityManager interface.";
            throw new BeanInitializationException(msg);
        } else {
            FilterChainManager manager = this.createFilterChainManager();
            RestPathMatchingFilterChainResolver chainResolver = new RestPathMatchingFilterChainResolver();
            chainResolver.setFilterChainManager(manager);
            return new RestShiroFilterFactoryBean.SpringShiroFilter((WebSecurityManager)securityManager, chainResolver);
        }
    }

    private static final class SpringShiroFilter extends AbstractShiroFilter {
        protected SpringShiroFilter(WebSecurityManager webSecurityManager, FilterChainResolver resolver) {
            if (webSecurityManager == null) {
                throw new IllegalArgumentException("WebSecurityManager property cannot be null.");
            } else {
                this.setSecurityManager(webSecurityManager);
                if (resolver != null) {
                    this.setFilterChainResolver(resolver);
                }

            }
        }
    }
}
````
上面是一些核心的代码片段，更多请看项目代码。  

对用户账户登录注册的过滤filter：PasswordFilter  
````
/* *
 * @Author tomsun28
 * @Description 基于 用户名密码 的认证过滤器
 * @Date 20:18 2018/2/10
 */
public class PasswordFilter extends AccessControlFilter {

    private static final Logger LOGGER = LoggerFactory.getLogger(PasswordFilter.class);

    private StringRedisTemplate redisTemplate;

    @Override
    protected boolean isAccessAllowed(ServletRequest request, ServletResponse response, Object mappedValue) {

        Subject subject = getSubject(request,response);
        // 如果其已经登录，再此发送登录请求
        if(null != subject && subject.isAuthenticated()){
            return true;
        }
        //  拒绝，统一交给 onAccessDenied 处理
        return false;
    }

    @Override
    protected boolean onAccessDenied(ServletRequest request, ServletResponse response) throws Exception {

        // 判断若为获取登录注册加密动态秘钥请求
        if (isPasswordTokenGet(request)) {
            //动态生成秘钥，redis存储秘钥供之后秘钥验证使用，设置有效期5秒用完即丢弃
            String tokenKey = CommonUtil.getRandomString(16);
            try {
                redisTemplate.opsForValue().set("PASSWORD_TOKEN_KEY_"+request.getRemoteAddr().toUpperCase(),tokenKey,5, TimeUnit.SECONDS);
                // 动态秘钥response返回给前端
                Message message = new Message();
                message.ok(1000,"issued tokenKey success")
                        .addData("tokenKey",tokenKey);
                RequestResponseUtil.responseWrite(JSON.toJSONString(message),response);

            }catch (Exception e) {
                LOGGER.warn(e.getMessage(),e);
                // 动态秘钥response返回给前端
                Message message = new Message();
                message.ok(1000,"issued tokenKey fail");
                RequestResponseUtil.responseWrite(JSON.toJSONString(message),response);
            }
            return false;
        }

        // 判断是否是登录请求
        if(isPasswordLoginPost(request)){
            AuthenticationToken authenticationToken = createPasswordToken(request);
            Subject subject = getSubject(request,response);
            try {
                subject.login(authenticationToken);
                //登录认证成功,进入请求派发json web token url资源内
                return true;
            }catch (AuthenticationException e) {
                LOGGER.warn(authenticationToken.getPrincipal()+"::"+e.getMessage(),e);
                // 返回response告诉客户端认证失败
                Message message = new Message().error(1002,"login fail");
                RequestResponseUtil.responseWrite(JSON.toJSONString(message),response);
                return false;
            }catch (Exception e) {
                LOGGER.error(e.getMessage(),e);
                // 返回response告诉客户端认证失败
                Message message = new Message().error(1002,"login fail");
                RequestResponseUtil.responseWrite(JSON.toJSONString(message),response);
                return false;
            }
        }
        // 判断是否为注册请求,若是通过过滤链进入controller注册
        if (isAccountRegisterPost(request)) {
            return true;
        }
        // 之后添加对账户的找回等

        // response 告知无效请求
        Message message = new Message().error(1111,"error request");
        RequestResponseUtil.responseWrite(JSON.toJSONString(message),response);
        return false;
    }

    private boolean isPasswordTokenGet(ServletRequest request) {
//        String tokenKey = request.getParameter("tokenKey");
        String tokenKey = RequestResponseUtil.getParameter(request,"tokenKey");

        return (request instanceof HttpServletRequest)
                && ((HttpServletRequest) request).getMethod().toUpperCase().equals("GET")
                && null != tokenKey && "get".equals(tokenKey);
    }

    private boolean isPasswordLoginPost(ServletRequest request) {
//        String password = request.getParameter("password");
//        String timestamp = request.getParameter("timestamp");
//        String methodName = request.getParameter("methodName");
//        String appId = request.getParameter("appId");
        Map<String ,String> map = RequestResponseUtil.getRequestParameters(request);
        String password = map.get("password");
        String timestamp = map.get("timestamp");
        String methodName = map.get("methodName");
        String appId = map.get("appId");
        return (request instanceof HttpServletRequest)
                && ((HttpServletRequest) request).getMethod().toUpperCase().equals("POST")
                && null != password
                && null != timestamp
                && null != methodName
                && null != appId
                && methodName.equals("login");
    }

    private boolean isAccountRegisterPost(ServletRequest request) {
//        String uid = request.getParameter("uid");
//        String methodName = request.getParameter("methodName");
//        String username = request.getParameter("username");
//        String password = request.getParameter("password");
        Map<String ,String> map = RequestResponseUtil.getRequestParameters(request);
        String uid = map.get("uid");
        String username = map.get("username");
        String methodName = map.get("methodName");
        String password = map.get("password");
        return (request instanceof HttpServletRequest)
                && ((HttpServletRequest) request).getMethod().toUpperCase().equals("POST")
                && null != username
                && null != password
                && null != methodName
                && null != uid
                && methodName.equals("register");
    }

    private AuthenticationToken createPasswordToken(ServletRequest request) {

//        String appId = request.getParameter("appId");
//        String password = request.getParameter("password");
//        String timestamp = request.getParameter("timestamp");
        Map<String ,String> map = RequestResponseUtil.getRequestParameters(request);
        String appId = map.get("appId");
        String timestamp = map.get("timestamp");
        String password = map.get("password");
        String host = request.getRemoteAddr();
        String tokenKey = redisTemplate.opsForValue().get("PASSWORD_TOKEN_KEY_"+host.toUpperCase());
        return new PasswordToken(appId,password,timestamp,host,tokenKey);
    }

    public void setRedisTemplate(StringRedisTemplate redisTemplate) {
        this.redisTemplate = redisTemplate;
    }

}

````
支持restful风格的jwt鉴权filter：BJwtFilter  

````
/* *
 * @Author tomsun28
 * @Description 支持restful url 的过滤链  JWT json web token 过滤器，无状态验证
 * @Date 0:04 2018/4/20
 */
public class BJwtFilter extends BPathMatchingFilter {

    private static final Logger LOGGER = LoggerFactory.getLogger(BJwtFilter.class);


    private StringRedisTemplate redisTemplate;
    private AccountService accountService;

    protected boolean isAccessAllowed(ServletRequest servletRequest, ServletResponse servletResponse, Object mappedValue) throws Exception {
        Subject subject = getSubject(servletRequest,servletResponse);
        // 判断是否为JWT认证请求
        if ((null == subject || !subject.isAuthenticated()) && isJwtSubmission(servletRequest)) {
            AuthenticationToken token = createJwtToken(servletRequest);
            try {
                subject.login(token);
//                return this.checkRoles(subject,mappedValue) && this.checkPerms(subject,mappedValue);
                return this.checkRoles(subject,mappedValue);
            }catch (AuthenticationException e) {
                LOGGER.info(e.getMessage(),e);
                // 如果是JWT过期
                if (e.getMessage().equals("expiredJwt")) {
                    // 这里初始方案先抛出令牌过期，之后设计为在Redis中查询当前appId对应令牌，其设置的过期时间是JWT的两倍，此作为JWT的refresh时间
                    // 当JWT的有效时间过期后，查询其refresh时间，refresh时间有效即重新派发新的JWT给客户端，
                    // refresh也过期则告知客户端JWT时间过期重新认证

                    // 当存储在redis的JWT没有过期，即refresh time 没有过期
                    String appId = WebUtils.toHttp(servletRequest).getHeader("appId");
                    String jwt = WebUtils.toHttp(servletRequest).getHeader("authorization");
                    String refreshJwt = redisTemplate.opsForValue().get("JWT-SESSION-"+appId);
                    if (null != refreshJwt && refreshJwt.equals(jwt)) {
                        // 重新申请新的JWT
                        // 根据appId获取其对应所拥有的角色(这里设计为角色对应资源，没有权限对应资源)
                        String roles = accountService.loadAccountRole(appId);
                        long refreshPeriodTime = 36000L;  //seconds为单位,10 hours
                        String newJwt = JsonWebTokenUtil.issueJWT(UUID.randomUUID().toString(),appId,
                                "token-server",refreshPeriodTime >> 2,roles,null, SignatureAlgorithm.HS512);
                        // 将签发的JWT存储到Redis： {JWT-SESSION-{appID} , jwt}
                        redisTemplate.opsForValue().set("JWT-SESSION-"+appId,newJwt,refreshPeriodTime, TimeUnit.SECONDS);
                        Message message = new Message().ok(1005,"new jwt").addData("jwt",newJwt);
                        RequestResponseUtil.responseWrite(JSON.toJSONString(message),servletResponse);
                        return false;
                    }else {
                        // jwt时间失效过期,jwt refresh time失效 返回jwt过期客户端重新登录
                        Message message = new Message().error(1006,"expired jwt");
                        RequestResponseUtil.responseWrite(JSON.toJSONString(message),servletResponse);
                        return false;
                    }

                }
                // 其他的判断为JWT错误无效
                Message message = new Message().error(1007,"error Jwt");
                RequestResponseUtil.responseWrite(JSON.toJSONString(message),servletResponse);
                return false;

            }catch (Exception e) {
                // 其他错误
                LOGGER.warn(servletRequest.getRemoteAddr()+"JWT认证"+e.getMessage(),e);
                // 告知客户端JWT错误1005,需重新登录申请jwt
                Message message = new Message().error(1007,"error jwt");
                RequestResponseUtil.responseWrite(JSON.toJSONString(message),servletResponse);
                return false;
            }
        }else {
            // 请求未携带jwt 判断为无效请求
            Message message = new Message().error(1111,"error request");
            RequestResponseUtil.responseWrite(JSON.toJSONString(message),servletResponse);
            return false;
        }
    }

    protected boolean onAccessDenied(ServletRequest servletRequest, ServletResponse servletResponse) throws Exception {
        Subject subject = getSubject(servletRequest,servletResponse);

        // 未认证的情况
        if (null == subject || !subject.isAuthenticated()) {
            // 告知客户端JWT认证失败需跳转到登录页面
            Message message = new Message().error(1006,"error jwt");
            RequestResponseUtil.responseWrite(JSON.toJSONString(message),servletResponse);
        }else {
            //  已经认证但未授权的情况
            // 告知客户端JWT没有权限访问此资源
            Message message = new Message().error(1008,"no permission");
            RequestResponseUtil.responseWrite(JSON.toJSONString(message),servletResponse);
        }
        // 过滤链终止
        return false;
    }

    private boolean isJwtSubmission(ServletRequest request) {

        String jwt = RequestResponseUtil.getHeader(request,"authorization");
        String appId = RequestResponseUtil.getHeader(request,"appId");
        return (request instanceof HttpServletRequest)
                && !StringUtils.isEmpty(jwt)
                && !StringUtils.isEmpty(appId);
    }

    private AuthenticationToken createJwtToken(ServletRequest request) {

        Map<String,String> maps = RequestResponseUtil.getRequestHeaders(request);
        String appId = maps.get("appId");
        String ipHost = request.getRemoteAddr();
        String jwt = maps.get("authorization");
        String deviceInfo = maps.get("deviceInfo");

        return new JwtToken(ipHost,deviceInfo,jwt,appId);
    }

    // 验证当前用户是否属于mappedValue任意一个角色
    private boolean checkRoles(Subject subject, Object mappedValue){
        String[] rolesArray = (String[]) mappedValue;
        return rolesArray == null || rolesArray.length == 0 || Stream.of(rolesArray).anyMatch(role -> subject.hasRole(role.trim()));
    }

    // 验证当前用户是否拥有mappedValue任意一个权限
    private boolean checkPerms(Subject subject, Object mappedValue){
        String[] perms = (String[]) mappedValue;
        boolean isPermitted = true;
        if (perms != null && perms.length > 0) {
            if (perms.length == 1) {
                if (!subject.isPermitted(perms[0])) {
                    isPermitted = false;
                }
            } else {
                if (!subject.isPermittedAll(perms)) {
                    isPermitted = false;
                }
            }
        }
        return isPermitted;
    }

    public void setRedisTemplate(StringRedisTemplate redisTemplate) {
        this.redisTemplate = redisTemplate;
    }


    public void setAccountService(AccountService accountService) {
        this.accountService = accountService;
    }
}

````
realm数据源，数据提供service，匹配matchs，自定义token，spring集成shiro配置等其他详见项目代码。  
最后项目实现了基于jwt的动态restful api权限认证。  



<br>


### 效果展示  

![image4](/images/posts/api/image4.PNG)   

![image5](/images/posts/api/image5.PNG)   

![image6](/images/posts/api/image6.PNG)   

![image7](/images/posts/api/image7.PNG)   



github:  
[bootshiro](https://github.com/tomsun28/bootshiro)  
[usthe](https://github.com/tomsun28/usthe)  

码云:  
[bootshiro](https://gitee.com/tomsun28/bootshiro)  
[usthe](https://gitee.com/tomsun28/usthe)  


<br>
持续更新。。。。。。
<br>

**分享一波阿里云代金券[快速上云](https://promotion.aliyun.com/ntms/act/ambassador/sharetouser.html?userCode=rjlzz3uf&utm_source=rjlzz3uf)**  

*转载请注明* [from tomsun28](http://usthe.com)
