---
title: 13-Shiro的rememberMe
date: 2021-09-14 11:55:51
tags: shiro
categories: shiro权限框架
cover: https://s3.bmp.ovh/imgs/2021/09/4ffd9cf6a603d931.jpg
---

#### 权限分类

| 用户访问页面分为3中级别        |                    |
| ------------------------------ | ------------------ |
| 未认证可以访问的页面           | 比如登录、注册页面 |
| “记住我”（曾登录过）访问的页面 | index.html         |
| 以认证可访问的页面             | 转账页面等         |

##### 记住密码的流程

​	第一次登录login.html页面时，点击“登录”会把用户的username和password存入cookie中，第二次登录login.html页面时输入username后，会调用js方法获取cookie里的密码并自动把密码填写到input标签里。

##### shiro的“记住我”流程

​	现在上述功能交给shiro的SecurityManager实现，只需要告诉shiro把UsernamePasswordToekn类型token传入SecurityManager中，就会自动检测。

##### 流程示意图

​	![avatar](https://z3.ax1x.com/2021/09/14/4FzFV1.png)

#### rememberMe实现

​	在ShiroConfig中设置过滤器的权限，让访问index.html页面是以登录或曾登录用户的才能访问。设置过滤filterMap规则。

```java
filterMap.put("/index.html","user");
```

​	并且设置cookie管理器，设置cookie的名字和存在时间。并且把rememberMe管理器传送给shiro的SecurityManager

```java
	@Bean
    public CookieRememberMeManager getCookieRememberMeManager() {
        CookieRememberMeManager rememberMeManager = new CookieRememberMeManager();
        SimpleCookie simpleCookie = new SimpleCookie("rememberMe");
        simpleCookie.setMaxAge(5*60);
        rememberMeManager.setCookie(simpleCookie);
        return rememberMeManager;
    }
    @Bean
    public DefaultWebSecurityManager getDefaultWebSecurityManager(MyRealm myRealm, EhCacheManager ehCacheManager) {
        DefaultWebSecurityManager defaultWebSecurityManager = new DefaultWebSecurityManager();
        defaultWebSecurityManager.setRealm(myRealm);
        defaultWebSecurityManager.setCacheManager(ehCacheManager);
        defaultWebSecurityManager.setRememberMeManager(getCookieRememberMeManager());
        return defaultWebSecurityManager;
    }
```

​	并设置当subject.login(token)时，不仅要传username和password还要穿rememberMe

```java
@Service
public class UserServiceImpl {
    @Resource
    private UserDao userDao;
    public void checkLogin(String username, String pwd, boolean rememberMe) throws Exception {
        UsernamePasswordToken token = new UsernamePasswordToken(username, pwd);
        Subject subject = SecurityUtils.getSubject();
        token.setRememberMe(rememberMe);
        subject.login(token);
    }
}
```

检测

​	login.html页面，填写好了username和password后点击“记住我”，进入index.html后，关闭所有页面，重新打开浏览器登录localhost:8080/index.html。

![avatar](https://z3.ax1x.com/2021/09/14/4EA0oD.png)

![avatar](https://z3.ax1x.com/2021/09/14/4EAoWj.png)