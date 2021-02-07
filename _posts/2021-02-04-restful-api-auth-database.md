---
layout: post
title: restful api 权限设计 - sureness集成springboot样例-数据库方案
date: 2021-02-04
tag: restful api auth
---

## restful api 权限设计 -  sureness集成springboot样例-数据库方案   

  在之前的文章[restful api 权限设计 - 快速搭建权限项目](https://usthe.com/2020/12/restful-api-auth-bootstrap/)  中，我们详细一步一步形式的描述了从零开始使用sureness集成springboot搭建一个拥有完整认证鉴权功能的权限项目后台，这个快速搭建的后台是基于配置文件的，也就是说并没有走数据库，用户信息和资源信息和对应的权限信息都写在了配置文件中。然而在常规的企业应用后台中，这些数据信息都是存在数据库中，今天我们就来详细描述下使用数据库作为数据源来集成sureness，springboot搭建一个企业级的完整功能的认证鉴权后台项目。   

<br>  
  对surenss来说，无论时配置文件方案还是数据库方案，其都是替换了不同的数据源而已，我们完全可以在之前搭建好的基于配置文件的项目进行修改替换，替换成数据库作为数据源。  

### 了解sureness提供的数据加载接口  

  首先我们先来认识下sureness提供的两个用户信息和资源权限信息的接口，用户可以实现这些接口自定义从不同的数据源给sureness提供数据。当我们把项目从配置文件模式切换成数据库模式时，也只是简单的替换了这些接口的实现类而已。  

一. `PathTreeProvider` 资源权限配置信息的数据源接口,我们可以实现从数据库,文本等加载接口想要的资源权限配置数据  

````
public interface PathTreeProvider {

    Set<String> providePathData();

    Set<String> provideExcludedResource();
}

````

此接口主要是需要实现上面这两个方法，providePathData是加载资源权限配置信息，也就是我们配置文件模式下sureness.yml的resourceRole信息列，provideExcludedResource是加载哪些资源可以被过滤不认证鉴权，也就是sureness.yml下的excludedResource信息列，如下。  

````
resourceRole:
  - /api/v2/host===post===[role2,role3,role4]
  - /api/v2/host===get===[role2,role3,role4]
  - /api/v2/host===delete===[role2,role3,role4]
  - /api/v2/host===put===[role2,role3,role4]
  - /api/mi/**===put===[role2,role3,role4]
  - /api/v1/getSource1===get===[role1,role2]
  - /api/v2/getSource2/*/*===get===[role2]

excludedResource:
  - /api/v1/source3===get
  - /api/v3/host===get
  - /**/*.css===get
  - /**/*.ico===get
  - /**/*.png===get
````

而当我们使用数据库模式时，实现这些信息从数据库关联读取就ok了，规范返回 eg: /api/v2/host===post===[role2,role3,role4] 格式的数据列，具体的数据库实现类参考类   - [DatabasePathTreeProvider](https://github.com/tomsun28/sureness/blob/master/sample-tom/src/main/java/com/usthe/sureness/sample/tom/sureness/provider/DatabasePathTreeProvider.java)   

二. `SurenessAccountProvider`这第二个相关的接口就是用户的账户密钥信息提供接口,我们需要实现从数据库或者文本等其他数据源那里去加载我们想要的用户的账户信息数据，这些数据提供给sureness让他进行用户的认证。  

````
public interface SurenessAccountProvider {
    SurenessAccount loadAccount(String appId);
}
````

此接口主要需要实现上面这个loadAccount方法，通过用户的唯一标识appid来从数据库或者redis缓存中查找到用户的账户信息返回即可。用户账户信息类SurenessAccount如下：  

````
public class DefaultAccount implements SurenessAccount {

    private String appId;
    private String password;
    private String salt;
    private List<String> ownRoles;
    private boolean disabledAccount;
    private boolean excessiveAttempts;
}
````

比较简单，主要是需要提供用户的密码相关信息即可，供sureness认证时密钥判断正确与否。  
这个具体的数据库接口实现可参考类 - [DatabaseAccountProvider](https://github.com/tomsun28/sureness/blob/master/sample-tom/src/main/java/com/usthe/sureness/sample/tom/sureness/provider/DatabaseAccountProvider.java)  

### 使用自定义配置来配置sureness  

在之前的简单样例中，我们使用的是sureness提供的默认配置，我们新建了一个配置类，创建对应的sureness默认配置bean  
sureness默认配置使用了文件数据源`sureness.yml`作为提供账户权限数据  
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

我们再来了解下sureness的大致流程，这样让我们更容易理解之后的自定义sureness配置。流程如下：  

![flow](/images/posts/sureness/flow-cn.png)   
如上面的流程所讲，Subject被SubjectCreate根据请求体所创造，不同的认证鉴权处理器Processor来出来所支持的Subject  
然后这里我们就来不使用默认配置，使用自定义配置来配置sureness的特性。  
这个自定义配置我们将原本的默认文件数据源替换为了数据库数据源，配置如下：  

````
@Configuration
public class SurenessConfiguration {

    @Bean
    ProcessorManager processorManager(SurenessAccountProvider accountProvider) {
        // 处理器Processor初始化
        List<Processor> processorList = new LinkedList<>();
        // 使用了默认的支持NoneSubject的处理器NoneProcessor 
        NoneProcessor noneProcessor = new NoneProcessor();
        processorList.add(noneProcessor);
        // 使用了默认的支持JwtSubject的处理器JwtProcessor  
        JwtProcessor jwtProcessor = new JwtProcessor();
        processorList.add(jwtProcessor);
        // 使用了默认的支持PasswordSubject的处理器PasswordProcessor  
        PasswordProcessor passwordProcessor = new PasswordProcessor();
        // 这里注意，PasswordProcessor需要对用户账户密码验证，所以其需要账户信息提供者来给他提供想要的账户数据，
        // 这里的 SurenessAccountProvider accountProvider bean就是这个账户数据提供源，
        // 其实现bean是上面讲到的 DatabaseAccountProvider bean,即数据库实现的账户数据提供者。 
        passwordProcessor.setAccountProvider(accountProvider);
        processorList.add(passwordProcessor);
        return new DefaultProcessorManager(processorList);
    }

    @Bean
    TreePathRoleMatcher pathRoleMatcher(PathTreeProvider databasePathTreeProvider) {
        // 这里的PathTreeProvider databasePathTreeProvider 就是通过数据库来提供资源权限配置信息bean实例
        // 下面我们再实例化一个通过文件sureness.yml提供资源权限配置信息的提供者
        PathTreeProvider documentPathTreeProvider = new DocumentPathTreeProvider();
        // 下面我们再实例化一个通过注解形式@RequiresRoles @WithoutAuth提供资源权限配置信息的提供者
        AnnotationPathTreeProvider annotationPathTreeProvider = new AnnotationPathTreeProvider();
        // 设置注解扫描包路径，也就是你提供api的controller路径 annotationPathTreeProvider.setScanPackages(Collections.singletonList("com.usthe.sureness.sample.tom.controller"));
        // 开始初始化资源权限匹配器，我们可以把上面三种提供者都加入到匹配器中为其提供资源权限数据，匹配器中的数据就是这三个提供者提供的数据集合。t
        DefaultPathRoleMatcher pathRoleMatcher = new DefaultPathRoleMatcher();
        pathRoleMatcher.setPathTreeProviderList(Arrays.asList(
                documentPathTreeProvider,
                annotationPathTreeProvider,
                databasePathTreeProvider));
        // 使用资源权限配置数据来建立对应的匹配树
        pathRoleMatcher.buildTree();
        return pathRoleMatcher;
    }

    @Bean
    SubjectFactory subjectFactory() {
        // 我们之前知道了可以有各种Processor来处理对应的Jwt，那这Subject怎么得到呢，就需要不同的SubjectCreator来根据请求信息创建对应的Subject对象供之后的流程使用
        SubjectFactory subjectFactory = new SurenessSubjectFactory();
        // 这里我们注册我们需要的SubjectCreator
        subjectFactory.registerSubjectCreator(Arrays.asList(
                // 注意! 强制必须首先添加一个 noSubjectCreator
                new NoneSubjectServletCreator(),
                // 注册用来创建PasswordSubject的creator
                new BasicSubjectServletCreator(),
                // 注册用来创建JwtSubject的creator
                new JwtSubjectServletCreator(),
                // 当然你可以自己实现一个自定义的creator，实现SubjectCreate接口即可
                new CustomPasswdSubjectCreator()));
        return subjectFactory;
    }

    @Bean
    SurenessSecurityManager securityManager(ProcessorManager processorManager,
                                            TreePathRoleMatcher pathRoleMatcher, SubjectFactory subjectFactory) {
        // 我们把上面初始化好的配置bean整合到一起初始化surenessSecurityManager 
        SurenessSecurityManager securityManager = SurenessSecurityManager.getInstance();
        // 设置资源权限匹配者
        securityManager.setPathRoleMatcher(pathRoleMatcher);
        // 设置subject创建工厂
        securityManager.setSubjectFactory(subjectFactory);
        // 设置处理器processor管理者
        securityManager.setProcessorManager(processorManager);
        return securityManager;
    }

}

````
这上面的SurenessConfiguration配置详细的说明了每个配置的用途，我们这里再来总结一下。  
SubjectCreator创建Subject，Processor来处理对应的Subject。  

processorManager方法就是提供我们配置支持的processor，每个processor需要的依赖不一样，比如PasswordProcessor需要对用户的账户密码信息做对比认证，所以其需要注入账户信息提供者来给他提供想要的账户数据，上面我们就注入了数据库方式账户信息提供者。  

pathRoleMatcher方法就是提供资源路径和对应所支持角色的匹配器，此匹配器当然也需要对应的资源权限数据来支撑，所以上面我们提供了三种资源权限数据提供者(数据库，文件，注解形式)，将其提供的配置数据都塞入到匹配器中使用。  

subjectFactory方法就是subject工厂，我们需要注册不同类型的subject创建方式-即SubjectCreator到工厂中，供不同的请求创建对应的Subject来使用  

securityManager就是将上面的所有配置整合到一个管理器中。  

这个具体的配置案例可以参考类 [SurenessConfiguration](https://github.com/tomsun28/sureness/tree/master/sample-tom/src/main/java/com/usthe/sureness/sample/tom/sureness/config)   

### 其他   

上面介绍了通过替换文件数据源为数据库数据源，来实现认证鉴权项目。然而数据库的表结构设计，怎么通过设计好的表数据关联提供出sureness想要的数据提供接口格式。这个可以参考 [sureness集成springboot样例(数据库方案)](https://github.com/tomsun28/sureness/tree/master/sample-tom) 里面提供了一个完整的数据库样例实现和对应的表设计和数据。  

更多自定义请访问sureness官方网站文档： https://su.usthe.com/  







