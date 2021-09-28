---
title: 11-Shiro的授权管理
date: 2021-09-11 18:20:12
tags: shiro
categories: shiro权限框架
cover: https://s3.bmp.ovh/imgs/2021/09/4ffd9cf6a603d931.jpg
---

#### 权限管理的方式

##### HTML页面限制

​	方式有两种，第一种是基于HTML页面展示的形式。在登录时查询到用户权限结合shiro标签库展示，展示出该用户所有的权限菜单。此方法有缺点，比如wangtaiwen用户是HR职务只有修改、查询客户这两个权限，在登录页面只能看到这两个菜单，但是ta无意间知道了“入库”权限的url，只要wangtaiwen是登录状态可以不用点击菜单直接输入url--->"localhost:8080/repoAdd.html"，就能越级访问。所以要在ShiroConfig中加入限制条件。只有改用户有sys:repo:save时才能访问“入库”

```java
filterMap.put("/repoAdd.html","perms[sys:repo:save]");
```

##### 注解限制

​	此方式不依赖shiro标签，在进入index.html页面后不管用户身份是什么，展示全部的菜单。但是用户只能点击ta有权限的那些菜单，其它菜单看得见但是点不进去。这需要在每个菜单的a标签的href=url对应的controller方法上加上注解，说明此方式是具有某个权限才能访问。比如”查询客户“菜单没有用户都可以看到，不用加《shiro:hasPermission name="sys:customer:find"》标签。

```java
<dd><a href="customer/customerfind" target="myFrame"> 查询客户</a></dd>
```

在CustomerController中加上注解

```java
@Controller
@RequestMapping("customer")
public class CustomerController {
    @RequestMapping("customerfind")
    @RequiresPermissions("sys:customer:find") // 当此用户有sys:customer:find是才能有客户查询权限
    public String cFind() {
        return "customerFind";
    }
}
```

#### 未授权处理

​	使用注解方式授权时，用户可以看到某个菜单，单点击就报错443。是因为缺少某个权限。比如用户yangtian是库管，ta有库存的增删改查权限。没有“查询客户”权限，此时点击就报错，就可以弹出页面提示用户，缺少权限lackpermission.html弹出。需要在ShiroConfig中设置未授权url链接。

```java
filter.setUnauthorizedUrl("/lackpermission.html");
```

​	还在PageController中处理次请求，跳转到lackpermision.html页面

```java
  @RequestMapping("/lackpermission.html")
    public String lackpermission() {
        return "lackpermission";
    }
```

​	弹出页面lackpermission.html

```html
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    缺少权限
</body>
</html>
```

