---
title: 9-引入layui实现Shiro后台管理页面
date: 2021-09-10 11:08:37
tags: shiro
categories: shiro权限框架
cover: https://s3.bmp.ovh/imgs/2021/09/4ffd9cf6a603d931.jpg
---

#### 引入layui

​	因为自己写的index.html页面简单难看，就要引入现成的框架Layui，可以去官网下载也可以引入SDN。把下载好的layui文件放入static中。

| 引入方式                                                     |
| ------------------------------------------------------------ |
| 本地引入                                                     |
| 在head标签里 添加《link rel="stylesheet" href="layui/css/layui.css》 |
| 在底部 body的上方添加《script src="layui/layui.js"></script》 |
| SDN引入                                                      |
| 在head标签里 添加《link rel="stylesheet" href="//unpkg.com/layui@2.6.8/dist/css/layui.css"》 |
| 在底部 body的上方添加《script src="//unpkg.com/layui@2.6.8/dist/layui.js"》 |

#### 页面布局

​	直接在官网上找到管理布局页面复制粘贴，原始模板页面如下

| 原始页面                                                     |
| ------------------------------------------------------------ |
| ![avatar](https://z3.ax1x.com/2021/09/10/hXdd1A.png)         |
| 需要修改左侧导航栏为权限列表                                 |
| ![avatar](https://z3.ax1x.com/2021/09/10/hXBPld.png)         |
| 改变右上角的用户信息,只用用户登录了，才会显示用户名，若是游客状态就不显示用户名 |
| ![avatar](https://z3.ax1x.com/2021/09/10/hXrNQJ.png)         |

#### index.html代码

```html
<!DOCTYPE html>
<html xmlns:shiro="http://pollix.at/themeleaf/shiro">
<head>
    <base href="/">
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1">
    <title>shiro后台 - Layui</title>
    <link rel="stylesheet" href="layui/css/layui.css">
    <!-- 引入 layui.css -->
    <!--<link rel="stylesheet" href="//unpkg.com/layui@2.6.8/dist/css/layui.css">-->
</head>
<body>
<div class="layui-layout layui-layout-admin">
    <div class="layui-header">
        <div class="layui-logo layui-hide-xs layui-bg-black">权限菜单</div>
        <!-- 头部区域（可配合layui 已有的水平导航） -->
        <ul class="layui-nav layui-layout-left">
            <li class="layui-nav-item"><a href="">控制台</a></li>
            <li class="layui-nav-item"><a href="">商品管理</a></li>
            <li class="layui-nav-item"><a href="">用户</a></li>
            <li class="layui-nav-item">
                <a href="javascript:;">其它系统</a>
                <dl class="layui-nav-child">
                    <dd><a href="">邮件管理</a></dd>
                    <dd><a href="">消息管理</a></dd>
                    <dd><a href="">授权管理</a></dd>
                </dl>
            </li>
        </ul>
        <ul class="layui-nav layui-layout-right">
            <!--登录状态-->
            <shiro:user>
                <li class="layui-nav-item layui-hide layui-show-md-inline-block">
                    <a href="javascript:;">
                        <img src="https://z3.ax1x.com/2021/08/08/fQDPht.png" class="layui-nav-img">
                        用户<shiro:principal/>欢迎你,当前用户职位是
                        <shiro:hasRole name="admin">超级管理员</shiro:hasRole>
                        <shiro:hasRole name="HR">人事</shiro:hasRole>
                        <shiro:hasRole name="seller">销售</shiro:hasRole>
                        <shiro:hasRole name="repoManager">库管</shiro:hasRole>
                        <shiro:hasRole name="planner">策划</shiro:hasRole>
                    </a>
                    <dl class="layui-nav-child">
                        <dd><a href="">基本资料</a></dd>
                        <dd><a href="">设置</a></dd>
                        <dd><a href="">退出</a></dd>
                    </dl>
                </li>
            </shiro:user>
            <!--游客-->
            <shiro:guest>
                请<label style="color: white;text-decoration: underline" onclick="javascript:location.href='login.html'">登录</label>
            </shiro:guest>
        </ul>
    </div>

    <div class="layui-side layui-bg-black">
        <div class="layui-side-scroll">
            <!-- 左侧导航区域（可配合layui已有的垂直导航） -->
            <ul class="layui-nav layui-nav-tree" lay-filter="test">
                <li class="layui-nav-item layui-nav-itemed">
                    <a class="" href="javascript:;">仓库管理</a>
                    <dl class="layui-nav-child">
                        <shiro:hasPermission name="sys:repo:save"><dd><a href="javascript:;">入库</a></dd></shiro:hasPermission>
                        <shiro:hasPermission name="sys:repo:delete"><dd><a href="javascript:;">出库</a></dd></shiro:hasPermission>
                        <shiro:hasPermission name="sys:repo:update"><dd><a href="javascript:;">修改库存</a></dd></shiro:hasPermission>
                        <shiro:hasPermission name="sys:repo:find"><dd><a href="">查询库存</a></dd></shiro:hasPermission>
                    </dl>
                </li>
                <li class="layui-nav-item">
                    <a href="javascript:;">客服管理</a>
                    <dl class="layui-nav-child">
                        <shiro:hasPermission name="sys:customer:save"><dd><a href="javascript:;">新增客户</a></dd></shiro:hasPermission>
                        <shiro:hasPermission name="sys:customer:delete"><dd><a href="javascript:;">删除客户</a></dd></shiro:hasPermission>
                        <shiro:hasPermission name="sys:customer:update"><dd><a href="javascript:;">修改客户</a></dd></shiro:hasPermission>
                        <shiro:hasPermission name="sys:customer:find"><dd><a href="javascript:;">查询客户</a></dd></shiro:hasPermission>
                    </dl>
                </li>
                <li class="layui-nav-item">
                    <a href="javascript:;">订单管理</a>
                    <dl class="layui-nav-child">
                        <shiro:hasPermission name="sys:order:save"><dd><a href="javascript:;">新增订单</a></dd></shiro:hasPermission>
                        <shiro:hasPermission name="sys:order:delete"><dd><a href="javascript:;">删除订单</a></dd></shiro:hasPermission>
                        <shiro:hasPermission name="sys:order:update"><dd><a href="javascript:;">修改订单</a></dd></shiro:hasPermission>
                        <shiro:hasPermission name="sys:order:find"><dd><a href="javascript:;">查询订单</a></dd></shiro:hasPermission>
                    </dl>
                </li>
            </ul>
        </div>
    </div>
    <div class="layui-body">
        <!-- 内容主体区域 -->
        <div style="padding: 15px;">内容主体区域。记得修改 layui.css 和 js 的路径</div>
    </div>

    <div class="layui-footer">
        <!-- 底部固定区域 -->
        底部固定区域
    </div>
</div>
<script src="layui/layui.js"></script>
<!-- 引入 layui.js -->
<!--
<script src="//unpkg.com/layui@2.6.8/dist/layui.js">
-->
<script>
    //JavaScript代码区域
    layui.use('element', function(){
        var element = layui.element;
    });
</script>
</body>
</html>
```

