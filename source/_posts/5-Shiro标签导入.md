---
title: 5-Shiro标签导入
date: 2021-09-07 17:50:24
tags: shiro
categories: shiro权限框架
cover: https://s3.bmp.ovh/imgs/2021/09/4ffd9cf6a603d931.jpg
---

#### 导入方式

​	在认证成功进入index页面后，需要看到用户信息，和相关的权限信息。Shiro就提供一套类似jstl的标签使用。shiro标签可以在jsp中也可以在ThemeLeaf中使用。导入后就能使用shiro的标签库。

| 引入方式                                                     |
| ------------------------------------------------------------ |
| 1。 jsp中===>   <%@ taglib prefix="shiro"  uri="hhtp://shiro.apache.org/tags" %> |
| 2。 ThemeLeaf中===>  <br /><html xmlns:th="http://www.thymeleaf.org"     xmlns:shiro="http://pollix.at/themeleaf/shiro"> |
|                                                              |

#### 引入依赖

​	要引入shiro对ThemeLeaf模板标签的依赖

```xml
<dependency>
    <groupId>com.github.theborakompanioni</groupId>
    <artifactId>thymeleaf-extras-shiro</artifactId>
    <version>2.0.0</version>
</dependency>
```

设置ShiroConfig类中shiro自定义tags

```java
// 注册这个bean目的就是为了在thymeleaf中使用shiro的自定义tag。
@Bean
public ShiroDialect shiroDialect() {
    return new ShiroDialect();
}
```

#### shiro的常用标签

|                   |                              |
| ----------------- | ---------------------------- |
| <shiro:guest>     | 游客身份访问页面展示的内容   |
| <shiro:user>      | 登录后用户在页面上看到的内容 |
| <shiro:principal> | 显示用户名                   |
| <shiro:hasRole>   | 判断用户是否有某个角色       |

#### 登录成功后index页面

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org"
      xmlns:shiro="http://pollix.at/themeleaf/shiro">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    index登录成功后/////////
    <hr>
    <shiro:guest>
        游客访问状态，<a href="/login.html">登录</a>
    </shiro:guest>
    <hr>
    <shiro:user>
        用户<shiro:principal/>欢迎你
        <br>
        当前用户职位是
        <shiro:hasRole name="admin">超级管理员</shiro:hasRole>
        <shiro:hasRole name="repairman">维修工</shiro:hasRole>
        <shiro:hasRole name="seller">销售</shiro:hasRole>
        <shiro:hasRole name="repoManager">库管</shiro:hasRole>
        <shiro:hasRole name="customerstaff">客服</shiro:hasRole>
        <br>
        你有一下权限<br>
        仓库管理权限
        <ul>
            <shiro:hasPermission name="sys:repo:save"><li><a href="#">入库</a></li></shiro:hasPermission>
            <shiro:hasPermission name="sys:repo:delete"><li><a href="#">出库</a></li></shiro:hasPermission>
            <shiro:hasPermission name="sys:repo:update"><li><a href="#">修改库存</a></li></shiro:hasPermission>
            <shiro:hasPermission name="sys:repo:find"><li><a href="#">查询库存</a></li></shiro:hasPermission>
        </ul>
        订单管理权限
        <ul>
            <shiro:hasPermission name="sys:seller:save"><li><a href="#">增加订单</a></li></shiro:hasPermission>
            <shiro:hasPermission name="sys:seller:delete"><li><a href="#">删除订单</a></li></shiro:hasPermission>
            <shiro:hasPermission name="sys:seller:update"><li><a href="#">修改订单</a></li></shiro:hasPermission>
            <shiro:hasPermission name="sys:seller:find"><li><a href="#">查找订单</a></li></shiro:hasPermission>
        </ul>
        <br>
        客服管理权限
        <ul>
            <shiro:hasPermission name="sys:customerstaff:save"><li><a href="#">增加客户</a></li></shiro:hasPermission>
            <shiro:hasPermission name="sys:customerstaff:delete"><li><a href="#">删除客户</a></li></shiro:hasPermission>
            <shiro:hasPermission name="sys:customerstaff:update"><li><a href="#">修改客户</a></li></shiro:hasPermission>
            <shiro:hasPermission name="sys:customerstaff:find"><li><a href="#">查找客户</a></li></shiro:hasPermission>
        </ul>
    </shiro:user>
</body>
</html>
```

