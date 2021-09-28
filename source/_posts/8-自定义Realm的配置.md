---
title: 8-自定义Realm的配置
date: 2021-09-09 16:41:31
tags: shiro
categories: shiro权限框架
cover: https://s3.bmp.ovh/imgs/2021/09/4ffd9cf6a603d931.jpg
---

#### 改变Realm来源

​	在ShiroConfig类中之前是连接JdbcRealm，JdbcRealm自动封装了认证和授权的SQL，而自定义realm到自己写，并且安全数据源要换成myRealm。

```java
    @Bean
    public MyRealm getMyRealm(){
        MyRealm myRealm = new MyRealm();
        return myRealm;
    }
     @Bean
    public DefaultWebSecurityManager getDefaultWebSecurityManager(MyRealm myRealm) {
        DefaultWebSecurityManager defaultWebSecurityManager = new DefaultWebSecurityManager();
        defaultWebSecurityManager.setRealm(myRealm);
        return defaultWebSecurityManager;
    }
```

#### 配置MyRealm

​	myRealm主要负责登录和授权，并把查询结果返回给SecurityManager安全管理器。并表示MyRealm类的真实名字。MyRealm类继承AuthorizingRealm类shiro从才认识自定义的realm。并把认证和授权的信息封装成Realm规定的对象

```java
public class MyRealm extends AuthorizingRealm {
    @Resource
    private UserDao userDao;
    @Resource
    private RoleDao roleDao;
    @Resource
    private PermissionDao permissionDao;
    public String getName(){
        return "myRealm";
    }
    /**
    *  授权功能：获取用户的角色和权限信息
    */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        String username = (String) principalCollection.iterator().next();
        Set<String> roleNames = roleDao.queryRolenameByUsername(username);
        Set<String> permissionCode = permissionDao.queryPermissionCodeByUsername(username);
        SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
        info.setRoles(roleNames);
        info.setStringPermissions(permissionCode);
        return info;
    }
    /**
    * 认证方法从自定义realm中获取安全数据
     * 因为jdbcRealm会自动认证，只需要subject.login(token)
     * 而自定义realm要查安全数据
     * 并把结果包装成AuthenticationInfo对象
    */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        UsernamePasswordToken token = (UsernamePasswordToken) authenticationToken;
        String username = token.getUsername();
        Users users = userDao.queryUserByUsername(username);
        // 用户名，密码，realm名
        AuthenticationInfo info = new SimpleAuthenticationInfo(username,users.getPwd(),getName());
        return info;
    }
}
```

