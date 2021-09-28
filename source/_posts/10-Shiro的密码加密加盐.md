---
title: 10-Shiro的密码加密加盐
date: 2021-09-11 10:46:42
tags: shiro
categories: shiro权限框架
cover: https://s3.bmp.ovh/imgs/2021/09/4ffd9cf6a603d931.jpg
---

#### 密码加密

​	一般加密的方式有2种，一种是base64加密是可以双向破解的。另一种的md5码加密只能由明文转成密文，不能有密文转成明文。这种单向转换更加安全，即使黑客进入数据库了无法根据密文密码登录。特别注意在注册时密码加密规则是什么就在shiro认证是告诉MyRealm密码匹配规则。myRealm最终匹配的是密文（先把input的密码转换成密文再和数据库的密文比对）代码如下

```java
 在ShiroConfig类中
	@Bean
    public HashedCredentialsMatcher getHashedCredentialsMatcher (){
     	// 设置密码匹配规则
        HashedCredentialsMatcher matcher = new HashedCredentialsMatcher();
        matcher.setHashAlgorithmName("md5");
        matcher.setHashIterations(1);
        return matcher;
    }
    @Bean
    public MyRealm getMyRealm(HashedCredentialsMatcher matcher) {
        MyRealm myRealm = new MyRealm();
        myRealm.setCredentialsMatcher(matcher);
        return myRealm;
    }
```

#### 加密测试

​	现在数据库数据里username=zhoudu存的是123456的32位md5码，现在登录访问localhost：8080/输入username=zhoudu,password=123456。流程如下图。

| ![avatar](https://z3.ax1x.com/2021/09/11/hx0mu9.png) |
| ---------------------------------------------------- |
| ![avatar](https://z3.ax1x.com/2021/09/11/hxBnxg.png) |
|                                                      |
|                                                      |

#### 密码加盐

##### 注册时加密加盐规则

​	加盐是让密码更安全的手段，可以生成随机数加密码前面后面等等。关键是在注册时规定密码加密规则和加盐规则。加盐加密后密码需要存入数据库。

```java
@Controller
@RequestMapping("user")
public class UserController {
	@RequestMapping("/register")
    public String register(String username, String pwd) {
        Md5Hash md5 = new Md5Hash(pwd);
        System.out.println("单纯加密后 md5---》"+md5);
        int num = new Random().nextInt(100)+100;
        String salt = num+"";
        Md5Hash md5_salt = new Md5Hash(pwd,salt);
        System.out.println("md5_salt----->"+md5_salt); 
        // 不能把单纯加密后 md5存入数据库，
        // 因为在MyRealm中的认证里写入加盐参数
  		// AuthenticationInfo info = new SimpleAuthenticationInfo(username,users.getPwd(),ByteSource.Util.bytes(users.getPwdSalt(),getName());
        // 所以必须把密码加盐加密的md5码存入数据库，
        userService.insertOneUser(username, md5_salt.toString(), salt);
        return "index";
    }
}
```

还需要增加insert方法在service，dao，mapper里。并添加register.html

##### 注册页面register.html

```java
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <form action="/user/register" method="post">
      <p>账号<input type="text" name="username" /></p>
      <p>密码<input type="text" name="pwd"/></p>
      <input type="submit" value="提交"/>
    </form>
</body>
</html>
```

##### UserService，UserDao，UserMapper.xml一套

```java
UserServiceImpl
	// 注册
    public void insertOneUser(String username, String pwd, String salt) {
        userDao.insertOneUser(username,pwd,salt);
    }
    ------------------------------------
 UserDao
 	public int insertOneUser(String username, String password, String salt) ;
 	------------------------------------
 UserMapper.xml	
 <insert id="insertOneUser" >
        insert into users (username,password,password_salt) values(#{username}, #{password}, #{salt});
    </insert>
```

#### MyRealm认证规则

​	因为注册时对密码加密加盐看，所以MyRealm认证时要查询数据库的password_salt，并存入AuthenticationInfo对象里。

```java

public class MyRealm extends AuthorizingRealm {
    @Resource
    private UserDao userDao;
protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        UsernamePasswordToken token = (UsernamePasswordToken) authenticationToken;
        String username = token.getUsername();
        Users users = userDao.queryUserByUsername(username);
        // 用户名，密码，salt,realm.
        AuthenticationInfo info = new SimpleAuthenticationInfo(
                username,users.getPwd(),
                ByteSource.Util.bytes(users.getPwdSalt()),
                getName());
        return info;
    }
 }
```

#### 加盐测试

​	先注册新账号、密码。laowang13 ，112233。注册成功后登陆

| ![avatar](https://z3.ax1x.com/2021/09/11/hx5F5F.png) |
| ---------------------------------------------------- |
| ![avatar](https://z3.ax1x.com/2021/09/11/hx5V29.png) |
| ![avatar](https://z3.ax1x.com/2021/09/11/hx5tKI.png) |

