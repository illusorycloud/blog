---
title: "Shiro安全框架(三)---Shiro组件详解"
description: "Shiro组件介绍和源码简单分析"
date: 2019-03-28 22:00:00
draft: false
tags: ["Shiro"]
categories: ["Shiro"]
---

本文主要讲述了Shiro安全框架的各大组件及其作用，同时使用实例代码做出简单演示。

<!--more-->

> **[Shiro安全框架系列文章目录](https://www.lixueduan.com/categories/Shiro/)**
>
> [Shiro安全框架(一)---什么是Shiro](https://www.lixueduan.com/posts/7dd8d38.html)
>
> [Shiro安全框架(二)---SpringBoot整合Shiro](https://www.lixueduan.com/posts/fd462737.html)
>
>  [Shiro安全框架(三)---Shiro组件详解](https://www.lixueduan.com/posts/f8844037.html)
>
> 源码下载：[GItHub](https://github.com/illusorycloud/springboot-learning)
>
> 更多文章欢迎访问我的个人博客-->[幻境云图](https://www.lixueduan.com/)

## 1. Authentication 用户认证

### 1.1 身份和凭

![](https://github.com/illusorycloud/illusorycloud.github.io/raw/hexo/myImages/shiro/ShiroFeatures_Authentication.png)

需要提供身份和凭证给 Shiro。

`Princirpals` ：用户身份信息，是 Subject 标识信息，能够标识唯一 subject 。如电话、邮箱、身份证号码等。

`Credentials` : 凭证，就是密码，是只被subject知道的秘密值，可以是密码也可以是数字证书等。

Princirpals/Credentials的常见组合：账号+密码。在 Shiro 中使用`UsernamePasswordToken`来指定身份信息和凭证。

### 1.2 认证流程

![](https://github.com/illusorycloud/illusorycloud.github.io/raw/hexo/myImages/shiro/ShiroAuthenticationSequence.png)



- 1.把用户输入的账号密码封装成 Token 给 Subject
- 2.Subject 把 Token 给 SecurityManager
- 3.SecurityManager 调用 Authenticator 认证器
- 4.Authenticator 根据配置的策略去调用 对应Realms 获取相对应的数据
- 5.最后返回认证结果

### 1.3 实例代码
#### 1. Controller

获取用户输入的账号密码 然后交给Subject去登录

```java
    @RequestMapping(value = "/login")
    public String login(HttpServletRequest request,User inuser,String uname,String upwd) {
        System.out.println("用户名和密码是" + uname + upwd + " User-->" + inuser.toString());
        UsernamePasswordToken usernamePasswordToken = new UsernamePasswordToken(uname,upwd);
        Subject subject = SecurityUtils.getSubject();
        try {
            //登录
            subject.login(usernamePasswordToken);
            User user = (User) subject.getPrincipal();
            request.getSession().setAttribute("user", user);
            return "index";
        } catch (AuthenticationException e) {
            return "login";
        }
    }
```

根据前面的步骤，Subject 获取到 Token 后会交给SecurityManager，最后 Authenticator 去 Realms 中获取数据并进行登录认证。

#### 2. Realm

```java
public class AuthRealmTest extends AuthorizingRealm {
    @Autowired
    private UserService userService;

    /**
     * 授权
     *
     * @param principalCollection
     * @return
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        //1.获取session中的用户
        User user = (User) principalCollection.fromRealm(this.getClass().getName()).iterator().next();

        //2.去数据库查询当前user的权限
        List<String> strings = userService.selectPermissionByUserId(user.getUid());

        //3.将权限放入shiro中.
        SimpleAuthorizationInfo simpleAuthorizationInfo = new SimpleAuthorizationInfo();
        simpleAuthorizationInfo.addStringPermissions(strings);
        //4.返回授权信息AuthorizationInfo
        return simpleAuthorizationInfo;
    }

    /**
     * 登录认证
     *
     * @param authenticationToken
     * @return
     * @throws AuthenticationException ex
     *                                 密码校验在{@link CredentialsMatcherTest#doCredentialsMatch(AuthenticationToken, AuthenticationInfo)}
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        //1.将用户输入的token 就是authenticationToken强转为UsernamePasswordToken
        UsernamePasswordToken usernamePasswordToken = (UsernamePasswordToken) authenticationToken;
        //2.获取用户名
        String username = usernamePasswordToken.getUsername();
        //3.数据库中查询出user对象
        User user = userService.findUserByName(username);
        //4.查询出这个user的权限
        Set<Role> roles = user.getRoles();
        for (Role r : roles) {
            Set<Permission> permissions = r.getPermissions();
            for (Permission p : permissions) {
                String permission = p.getPermission();
                System.out.println("权限--》" + permission);
            }
        }
        //5.返回认证信息AuthenticationInfo 这里是没进行密码校验的 密码校验在CredentialsMatcherTest类中
        return new SimpleAuthenticationInfo(user, user.getUpwd(), this.getClass().getName());
    }
}

```

其中 `doGetAuthenticationInfo`方法中的Token就是用户前面输入的账号密码，我们还需要些一个类用来校验密码：

```java
/**
 * 密码校验类
 */
public class CredentialsMatcherTest extends SimpleCredentialsMatcher {
    /**
     * 校验密码
     *
     * @param token
     * @param info
     * @return 密码校验结果
     * {@link AuthRealmTest#doGetAuthenticationInfo(AuthenticationToken)}
     */
    @Override
    public boolean doCredentialsMatch(AuthenticationToken token, AuthenticationInfo info) {
        //1.强转
        UsernamePasswordToken usernamePasswordToken = (UsernamePasswordToken) token;
        //2.获取用户输入的密码
        char[] password = usernamePasswordToken.getPassword();
        String pwd = new String(password);
        //3.获取数据库中的真实密码
        //这个info就是前面AuthRealmTest类中的doGetAuthenticationInfo返回的info
        String relPwd = (String) info.getCredentials();
        //4.返回校验结果
        return this.equals(pwd, relPwd);

    }
}
```

到这里就算是认证成功了，但是还没有授权。

### 1.4 小结

认证流程：

1.用户输入账号密码，`Controller`中调`subject.login`去认证

2.认证方法就是自定义`realm`中的`doGetAuthenticationInfo`

3.认证过程中需要校验密码，就是自定义的`CredentialsMatcherTest`

## 2. Realm

Realm 是一个接口，在接口中定义了 Token 获得认证信息的方法，Shiro 内实现了一系列的 Realm，这些不同的Realm 提供了不同的功能，`AuthenticatingRealm`实现了获取身份信息的功能，`AuthorizingRealm`实现了获取权限信息的功能且继承了`AuthenticatingRealm`,自定义realm时要继承`AuthorizingRealm`,这样既可以提供身份认证的自定义方法，也可以实现授权的自定义方法。
**shiro只实现了功能，并不维护数据**，所以自定义realm中也只是从数据库中查询数据然后和用户输入进行对比，其中密码校验是单独的

### 2.1 实例代码

自定义Realm，继承`AuthorizingRealm`并实现授权方法`doGetAuthorizationInfo`和身份认证方法`doGetAuthenticationInfo`。

```java
public class AuthRealm extends AuthorizingRealm {
    @Autowired
    private UserServiceImpl userService;

    /**
     * 授权
     *
     * @param principalCollection
     * @return
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        //获取session中的用户
        User user = (User) principalCollection.fromRealm(this.getClass().getName()).iterator().next();
        //查询权限
        List<String> strings = userService.selectPermissionByUserId(user.getUid());
        SimpleAuthorizationInfo simpleAuthorizationInfo = new SimpleAuthorizationInfo();
        //将权限放入shiro中.
        simpleAuthorizationInfo.addStringPermissions(strings);
//        System.out.println("添加时的权限" + permission.toString());
        System.out.println("-------------授权-------------");
        return simpleAuthorizationInfo;
    }

    /**
     * 完成身份认证并返回认证信息
     * 认证失败则返回空
     *
     * @param authenticationToken
     * @return
     * @throws AuthenticationException
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        //用户输入的token
        UsernamePasswordToken usernamePasswordToken = (UsernamePasswordToken) authenticationToken;
        String username = usernamePasswordToken.getUsername();
        User user = userService.findUserByName(username);
        //放入shiro.调用CredentialsMatcher检验密码
        System.out.println("获取到的密码" + user.getUpwd());
//        ByteSource salt = ByteSource.Util.bytes(user.getSalt());
//        System.out.println(salt);
        return new SimpleAuthenticationInfo(user, user.getUpwd(),this.getClass().getName());
    }
}
```

## 3. Authentication Strategy 认证策略

在 Shiro 中有三种认证策略：

- **AtLeatOneSuccessfulStrategy(默认策略)** `:只要有一个Realm验证成功即可`，和`FirstSuccessfulStrategy`不同，将`返回所有`Realm身份校验成功的认证信息。

- **FirstSuccessfulStrategy** :`只要有一个Realm验证成功即可`，`只返回第一个`Realm身份验证成功的认证
  信息，其他的忽略。

- **AllSuccessfulStrategy** :所有Realm验证成功才算成功，且返回所有Realm身份认证成功的认证信息，如

  果`有一个失败就失败`了。

具体配置

```java
    /**
     * 认证策略配置
     *
     * @return modularRealmAuthenticator
     */
    @Bean
    public ModularRealmAuthenticator modularRealmAuthenticator() {
        ModularRealmAuthenticator modularRealmAuthenticator = new ModularRealmAuthenticator();
        AuthenticationStrategy atLeastOneSuccessfulStrategy = new AtLeastOneSuccessfulStrategy();
        modularRealmAuthenticator.setAuthenticationStrategy(atLeastOneSuccessfulStrategy);
        return modularRealmAuthenticator;
    }
```

## 4. CredentialsMatcher 凭证匹配器

### 1. CredentialsMatcherHash

自定义凭证匹配器,用来校验密码，其中数据库保存的密码是Hash后的，匹配是也需要Hash后进行对比。

```java
public class CredentialsMatcher extends SimpleCredentialsMatcher {
    @Override
    public boolean doCredentialsMatch(AuthenticationToken token, AuthenticationInfo info) {
        //强转 获取token
        UsernamePasswordToken usernamePasswordToken = (UsernamePasswordToken) token;
        //获取用户输入的密码
        char[] password = usernamePasswordToken.getPassword();
        String inputPassword = new String(password);
        Md5Hash md5Hash = new Md5Hash(inputPassword);
        //获取数据库中的密码
        String realPassword = (String) info.getCredentials();
        System.out.println("输入的密码"+md5Hash);
        System.out.println("数据库中的密码"+realPassword);
        //对比
        return this.equals(md5Hash, realPassword);
    }
}
```

### 2. CredentialsMatcherLimit

同时可以在凭证匹配器中设定登录次数，多次登录失败后限制一段时间内不让登录。

```java
public class CredentialsMatcherLimit extends SimpleCredentialsMatcher {
    /**
     * 当前登录次数 放在缓存中10分钟后清空 即连续登录失败后要等一段时间
     */
    private AtomicInteger tryTime;
    /**
     * 短时间内最大登录次数
     */
    private static final int MAX_TIMES = 5;

    @Override
    public boolean doCredentialsMatch(AuthenticationToken token, AuthenticationInfo info) {
        if (tryTime.get() < MAX_TIMES) {
            int currentTime = tryTime.getAndIncrement();
            System.out.println("登录次数：" + currentTime);
            //强转 获取token
            UsernamePasswordToken usernamePasswordToken = (UsernamePasswordToken) token;
            //获取用户输入的密码
            char[] password = usernamePasswordToken.getPassword();
            String inputPassword = new String(password);
             Md5Hash md5Hash = new Md5Hash(inputPassword);
            //获取数据库中的密码
            String realPassword = (String) info.getCredentials();
            System.out.println("输入的密码" + md5Hash);
            System.out.println("数据库中的密码" + realPassword);
            //对比
            return this.equals(md5Hash, realPassword);
        } else {
            System.out.println("登录次数过多，请稍后重试");
            return false;
        }
    }
}
```

## 5. Authorization 授权

![](https://github.com/illusorycloud/illusorycloud.github.io/raw/hexo/myImages/shiro/ShiroFeatures_Authorization.png)

### 5.1 简介

* `授权` ：给身份认证通过的人，授予某些权限。
* `权限粒度` ：分为粗粒度和细粒度，
  * 粗粒度 ：对某张表的操作，如对user表的crud。
  * 细粒度 ：对表中某条记录的操作，如：只能对user表中ID为1的记录进行curd，shiro一般管理的是粗粒度的权限，比如：菜单、URL，细粒度的权限控制通过业务来实现。
* `角色` ：权限的集合
* `权限表现规则`：格式:  `资源:操作:实例 `(可以用通配符表示)
  * user:add   对user有add权限
  * user:*        对user有所有操作
  * user:add:1   对ID为1的user有add操作

### 5.2 流程



![](https://github.com/illusorycloud/illusorycloud.github.io/raw/hexo/myImages/shiro/ShiroAuthorizationSequence.png)



Shiro中权限检测方式有三种：

- 1.编程式 业务代码前手动检测 
  - `subject.checkPermission("delete");/subject.hasRole("admin");`
- 2.注解式 方法上添加注解
  -  `@RequiresPermissions(value = "add")/@RequiresRoles("admin")`
- 3.标签式 写在html中
  - ` <p shiro:hasPermission="add">添加用户</p>` 需要在html页面中引入 `xmlns:shiro="http://www.pollix.at/thymeleaf/shiro"`

#### 具体流程

- 1.获取 Subject 主体
- 2.判断 Subject 主体是否通过认证
- 2.Subject 调用 isPermitted()/hasRole() 方法开始授权
- 3.SecurityManager执行授权，通过 ModularRealmAuthorizer 执行授权 
- 4.调用自定义 Realm 的授权方法：doGetAuthorizationInfo
- 5.返回授权结果

#### 实例代码

```java
    /**
     * 授权
     *
     * @param principalCollection
     * @return
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        //1.获取principalCollection中的用户
        User user = (User) principalCollection.fromRealm(this.getClass().getName()).iterator().next();
        //2.通过数据库查询当前userde权限
        List<String> permissions = userService.selectPermissionByUserId(user.getUid());
        SimpleAuthorizationInfo simpleAuthorizationInfo = new SimpleAuthorizationInfo();
        //3.将权限放入shiro中.
        simpleAuthorizationInfo.addStringPermissions(permissions);
//        System.out.println("添加时的权限" + permission.toString());
        System.out.println("-------------授权-------------");
        //4.返回
        return simpleAuthorizationInfo;
    }
```

## 6. 散列算法(加密)

### 6.1 简介

为了提高应用系统的安全性，在身份认证过程中往往会涉及加密，这里主要关注shiro提供的密码服务模块；通过shiro进行散列算法操作，常见的有两个MD5，SHA-1等。

如`1111`的MD5为`b59c67bf196a4758191e42f76670ceba`,但是这个`b59c67bf196a4758191e42f76670ceba`很容易就会被破解，轻松就能获取到加密前的数据。

### 6.2 加盐

但是`1111+userName`进行加密，这样就不容易被破解了，破解难度增加。

例如:

`qwer`的MD5为`962012d09b8170d912f0669f6d7d9d07`
`qwer`加盐`illusory`后的MD5为`6aee9c0e35ad7a12e59ff67b663a32ca`

用户在注册的时候就把加密后的密码和盐值存到数据库，用户登录时就先根据用户名查询盐值，然后把用户输入的密码加密后在和数据库中的密码做对比。

代码如下：
自定义密码校验,shiro也提供了一下内置的加密密码校验器

> 1.根据name查询user 然后获取到盐值
> 2.然后把输入的密码加密
> 3.最后在于数据库中的密码对比

```java
public class CredentialsMatcherHash extends SimpleCredentialsMatcher {
    @Autowired
    UserService service;

    @Override
    public boolean doCredentialsMatch(AuthenticationToken token, AuthenticationInfo info) {
        //强转 获取token
        UsernamePasswordToken usernamePasswordToken = (UsernamePasswordToken) token;
        //获取用户输入的密码
        char[] password = usernamePasswordToken.getPassword();
        String inputPassword = new String(password);
        String username = usernamePasswordToken.getUsername();
        User userByName = service.findUserJustByName(username);
        String salt = userByName.getSalt();
        //这个盐值是从数据库查出来的
        Md5Hash md5Hash = new Md5Hash(inputPassword, salt);
        String inputMD5Hash = new String(String.valueOf(md5Hash));
        //获取数据库中的密码
        String realPassword = (String) info.getCredentials();
        System.out.println("输入的密码" + inputPassword);
        System.out.println("输入的密码加密" + md5Hash);
        System.out.println("数据库中的密码" + realPassword);
        //对比
        return this.equals(inputMD5Hash, realPassword);
    }
}
```

## 7. 缓存

每次检查都会去数据库中获取权限，这样效率很低，可以通过设置缓存来解决问题。如` Ehcache` 或者 `Redis`。

这里使用`Redis`。

### 7.1 引入依赖

```xml
        <!--redis-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
```

### 7.2 重写方法

使用 Redis 作为缓存需要 Shiro重写 Cache、cacheManager

#### CacheManager

```java
/**
 * Cachemanager
 *
 * @author illusoryCloud
 */
public class ShiroRedisCacheManager extends AbstractCacheManager {
    private RedisTemplate<byte[], byte[]> redisTemplate;

    public ShiroRedisCacheManager(RedisTemplate redisTemplate) {
        this.redisTemplate = redisTemplate;
    }

    //为了个性化配置redis存储时的key，我们选择了加前缀的方式，所以写了一个带名字及redis操作的构造函数的Cache类
    @Override
    protected Cache createCache(String name) throws CacheException {
        return new ShiroRedisCache(redisTemplate, name);
    }
}
```

#### RedisCache

```java
/**
 * Shiro缓存
 *
 * @author illusoryCloud
 */
public class ShiroRedisCache<K, V> implements Cache<K, V> {
    /**
     * redis操作对象
     */
    private RedisTemplate redisTemplate;
    /**
     * key 前缀
     */
    private String prefix = "shiro_redis";

    public String getPrefix() {
        return prefix + ":";
    }

    public void setPrefix(String prefix) {
        this.prefix = prefix;
    }

    public ShiroRedisCache(RedisTemplate redisTemplate) {
        this.redisTemplate = redisTemplate;
    }

    public ShiroRedisCache(RedisTemplate redisTemplate, String prefix) {
        this(redisTemplate);
        this.prefix = prefix;
    }

    /**
     * get方法
     * @param k redis中的key
     * @return redis中的value
     * @throws CacheException
     */
    @Override
    public V get(K k) throws CacheException {
        if (k == null) {
            return null;
        }
        byte[] bytes = getBytesKey(k);
        return (V) redisTemplate.opsForValue().get(bytes);

    }

    /**
     * put方法
     * @param k key
     * @param v value
     * @return
     * @throws CacheException
     */
    @Override
    public V put(K k, V v) throws CacheException {
        if (k == null || v == null) {
            return null;
        }

        byte[] bytes = getBytesKey(k);
        redisTemplate.opsForValue().set(bytes, v);
        return v;
    }

    /**
     * delete方法
     * @param k
     * @return
     * @throws CacheException
     */
    @Override
    public V remove(K k) throws CacheException {
        if (k == null) {
            return null;
        }
        byte[] bytes = getBytesKey(k);
        V v = (V) redisTemplate.opsForValue().get(bytes);
        redisTemplate.delete(bytes);
        return v;
    }

    /**
     * 清除数据库
     * @throws CacheException
     */
    @Override
    public void clear() throws CacheException {
        redisTemplate.getConnectionFactory().getConnection().flushDb();

    }

    @Override
    public int size() {
        return redisTemplate.getConnectionFactory().getConnection().dbSize().intValue();
    }

    /**
     * 查询所有的key
     * key
     * @return
     */
    @Override
    public Set<K> keys() {
        byte[] bytes = (getPrefix() + "*").getBytes();
        Set<byte[]> keys = redisTemplate.keys(bytes);
        Set<K> sets = new HashSet<>();
        for (byte[] key : keys) {
            sets.add((K) key);
        }
        return sets;
    }

    /**
     * 查询所有的value
     * @return
     */
    @Override
    public Collection<V> values() {
        Set<K> keys = keys();
        List<V> values = new ArrayList<>(keys.size());
        for (K k : keys) {
            values.add(get(k));
        }
        return values;
    }

    private byte[] getBytesKey(K key) {
        String prekey = this.getPrefix() + key;
        return prekey.getBytes();
    }
```

### 7.3 Shiro配置缓存管理器

在 ShiroConfiguration 中配置 ShiroRedisCacheManager。

```java
    @Bean
    public ShiroRedisCacheManager cacheManager(RedisTemplate redisTemplate) {
        return new ShiroRedisCacheManager(redisTemplate);
    }

    //配置核心安全事务管理器
    @Bean(name = "securityManager")
    public SecurityManager securityManager(@Qualifier("authRealm") AuthRealm authRealm,
                                           @Qualifier("authRealm2") AuthRealm authRealm2,
                                           @Qualifier("authRealm3") AuthRealm authRealm3
            , RedisTemplate<Object, Object> template) {
        System.err.println("----------------------------shiro已经加载---------------------------");
        DefaultWebSecurityManager manager = new DefaultWebSecurityManager();
        //配置缓存 必须放在realm前面
        manager.setCacheManager(cacheManager(template));
        //配置两个测试一下认证策略AllSuccessfulStrategy
//        manager.setRealm(authRealm);
//        manager.setRealm(authRealm2);

        //测试一下密码加密
        manager.setRealm(authRealm3);

        manager.setSessionManager(sessionManager());
//        manager.setCacheManager(ehCacheManager);
        return manager;
    }
```

到这里就ok了

Shiro 在认证时会首先去 Redis 缓存中查询，缓存中没有才会去去查询数据库。

## 8. Session

### Shiro 中的 Session 特性

* 基于POJO/J2SE：shiro中session相关的类都是基于接口实现的简单的java对象（POJO），兼容所有java对象的配置方式，扩展也更方便，完全可以定制自己的会话管理功能 。
* 简单灵活的会话存储/持久化：因为shiro中的session对象是基于简单的java对象的，所以你可以将session存储在任何地方，例如，文件，各种数据库，内存中等。
* 容器无关的集群功能：shiro中的session可以很容易的集成第三方的缓存产品完成集群的功能。例如，Ehcache + Terracotta, Coherence, GigaSpaces等。你可以很容易的实现会话集群而无需关注底层的容器实现。
* 异构客户端的访问：可以实现web中的session和非web项目中的session共享。
* 会话事件监听：提供对对session整个生命周期的监听。
* 保存主机地址：在会话开始session会存用户的ip地址和主机名，以此可以判断用户的位置。
* 会话失效/过期的支持：用户长时间处于不活跃状态可以使会话过期，调用touch()方法，可以主动更新最后访问时间，让会话处于活跃状态。
* 透明的Web支持：shiro全面支持Servlet 2.5中的session规范。这意味着你可以将你现有的web程序改为shiro会话，而无需修改代码。

单点登录的支持：shiro session基于普通java对象，使得它更容易存储和共享，可以实现跨应用程序共享。可以根据共享的会话，来保证认证状态到另一个程序。从而实现单点登录。

### 8.2 SessionListener

```java
/**
 * 监听session变化
 *
 * @author illusoryCloud
 */
public class ShiroSessionListener implements SessionListener {
    /**
     * 统计在线人数
     * juc包下线程安全自增
     */
    private final AtomicInteger sessionCount = new AtomicInteger(0);

    /**
     * 会话创建时触发
     *
     * @param session
     */
    @Override
    public void onStart(Session session) {
        //会话创建，在线人数加一
        sessionCount.incrementAndGet();
    }

    /**
     * 退出会话时触发
     *
     * @param session
     */
    @Override
    public void onStop(Session session) {
        //会话退出,在线人数减一
        sessionCount.decrementAndGet();
    }

    /**
     * 会话过期时触发
     *
     * @param session
     */
    @Override
    public void onExpiration(Session session) {
        //会话过期,在线人数减一
        sessionCount.decrementAndGet();
    }

    /**
     * 获取在线人数使用
     *
     * @return
     */
    public AtomicInteger getSessionCount() {
        return sessionCount;
    }
}
```

### 8.3 ShiroConfiguration 配置

```java
 /**
     * 配置session监听
     *
     * @return
     */
    @Bean("sessionListener")
    public ShiroSessionListener sessionListener() {
        ShiroSessionListener sessionListener = new ShiroSessionListener();
        return sessionListener;
    }

    /**
     * 配置会话ID生成器
     *
     * @return
     */
    @Bean
    public SessionIdGenerator sessionIdGenerator() {
        return new JavaUuidSessionIdGenerator();
    }

    /**
     * SessionDAO的作用是为Session提供CRUD并进行持久化的一个shiro组件
     * MemorySessionDAO 直接在内存中进行会话维护
     * EnterpriseCacheSessionDAO  提供了缓存功能的会话维护，默认情况下使用MapCache实现，内部使用ConcurrentHashMap保存缓存的会话。
     *
     * @return
     */
    @Bean
    public SessionDAO sessionDAO() {
        EnterpriseCacheSessionDAO enterpriseCacheSessionDAO = new EnterpriseCacheSessionDAO();
        //使用ehCacheManager
        enterpriseCacheSessionDAO.setCacheManager(cacheManager(new RedisTemplate()));
        //设置session缓存的名字 默认为 shiro-activeSessionCache
        enterpriseCacheSessionDAO.setActiveSessionsCacheName("shiro-activeSessionCache");
        //sessionId生成器
        enterpriseCacheSessionDAO.setSessionIdGenerator(sessionIdGenerator());
        return enterpriseCacheSessionDAO;
    }

    @Bean("sessionManager")
    public SessionManager sessionManager() {
        DefaultWebSessionManager sessionManager = new DefaultWebSessionManager();
        Collection<SessionListener> listeners = new ArrayList<SessionListener>();
        //配置监听
        listeners.add(sessionListener());
        sessionManager.setSessionListeners(listeners);
        sessionManager.setSessionIdCookie(sessionIdCookie());
        sessionManager.setSessionDAO(sessionDAO());
        //全局会话超时时间（单位毫秒），默认30分钟  暂时设置为10秒钟 用来测试
        sessionManager.setGlobalSessionTimeout(1800000);
        //是否开启删除无效的session对象  默认为true
        sessionManager.setDeleteInvalidSessions(true);
        //是否开启定时调度器进行检测过期session 默认为true
        sessionManager.setSessionValidationSchedulerEnabled(true);
        //设置session失效的扫描时间, 清理用户直接关闭浏览器造成的孤立会话 默认为 1个小时
        //设置该属性 就不需要设置 ExecutorServiceSessionValidationScheduler 底层也是默认自动调用ExecutorServiceSessionValidationScheduler
        //暂时设置为 5秒 用来测试
        sessionManager.setSessionValidationInterval(3600000);
        return sessionManager;
    }

    /**
     * 配置保存sessionId的cookie
     * 注意：这里的cookie 不是上面的记住我 cookie 记住我需要一个cookie session管理 也需要自己的cookie
     *
     * @return
     */
    @Bean("sessionIdCookie")
    public SimpleCookie sessionIdCookie() {
        //这个参数是cookie的名称
        SimpleCookie simpleCookie = new SimpleCookie("sid");
        //setcookie的httponly属性如果设为true的话，会增加对xss防护的安全系数。它有以下特点：

        //setcookie()的第七个参数
        //设为true后，只能通过http访问，javascript无法访问
        //防止xss读取cookie
        simpleCookie.setHttpOnly(true);
        simpleCookie.setPath("/");
        //maxAge=-1表示浏览器关闭时失效此Cookie
        simpleCookie.setMaxAge(-1);
        return simpleCookie;
    }

```

## 9. RememberMe

### ShiroConfiguration 配置

```java
    @Bean
    public RememberMeManager rememberMeManager() {
        CookieRememberMeManager cookieRememberMeManager = new CookieRememberMeManager();
        cookieRememberMeManager.setCookie(rememberMeCookie());
        //rememberMe cookie加密的密钥 建议每个项目都不一样 默认AES算法 密钥长度(128 256 512 位)
        cookieRememberMeManager.setCipherKey(Base64.decode("2AvVhdsgUs0FSA3SDFAdag=="));
        return cookieRememberMeManager;
    }

    @Bean
    public SimpleCookie rememberMeCookie() {
        SimpleCookie simpleCookie = new SimpleCookie("rememberMe");
        simpleCookie.setMaxAge(259200);
        return simpleCookie;
    }
```

### 2. Controller

前端页面传过来一个Boolean变量，然后存放在`UsernamePasswordToken`中就可以了,不过User对象因为要序列化所以要实现`Serializable`接口，同样的还有User对象引用的permission和role对象都要实现这个。

```java
    @RequestMapping(value = "/login")
    public String login(HttpServletRequest request, User inuser, String uname, String upwd,Boolean rememberMe) {
        System.out.println("用户名和密码是" + uname + upwd + " User-->" + inuser.toString());
        UsernamePasswordToken usernamePasswordToken = new UsernamePasswordToken(uname, upwd, rememberMe);
        Subject subject = SecurityUtils.getSubject();
        try {
            //登录
            subject.login(usernamePasswordToken);
            User user = (User) subject.getPrincipal();
            return "index";
        } catch (AuthenticationException e) {
            usernamePasswordToken.clear();
            return "login";
        }
    }
```

## 10. 参考

`https://blog.csdn.net/yangwenxue_admin/article/details/73936803`

`官方文档：http://shiro.apache.org/documentation.html`