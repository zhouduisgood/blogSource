---
title: 2-Shiro安全框架简介
date: 2021-09-04 12:00:47
tags: shiro
categories: shiro权限框架
cover: https://s3.bmp.ovh/imgs/2021/09/4ffd9cf6a603d931.jpg
---

#### 认证授权流程

​	认证：对用户信息身份检查（登录认证）

​	授权：对用户权限检查（是否有对应的操作权限）

​	流程图如下：

![avatar](https://z3.ax1x.com/2021/09/04/hgm81I.png)

#### 安全框架

​	安全框架相当于保安，对应请求完成认证（有没资格登录）和授权（有没有资格获取对应的权限）的工作。

##### 常用的安全框架有

| shiro：        | Apache组织的一种开源java安全框架（小而简单）                 |
| -------------- | ------------------------------------------------------------ |
| SpringSecurity | 对spring有依赖性                                             |
| OAuth2：       | 第三方授权登录框架（比如王者荣耀可以用微信、QQ账号、微博等等账号登录） |

#### Shrio简介

​	可以完成用户认证、授权、密码和会话管理。可以引用任何应用系统（主要针对单体项目的权限管理）。

#### Shiro工作原理

​	Shiro框架主要有四大功能。

![avatar](https://z3.ax1x.com/2021/09/04/hguYTS.png)

​	Authentication:身份认证 / 登录，验证用户是不是拥有相应的身份；

​	Authorization：授权，即权限验证，验证某个已认证的用户是否拥有某个权限或某个角色；

​	Session Management：会话管理，即用户登录后就是一次会话，在没有退出之前，它的所有信息都在会话中；会话可以是普通 JavaSE 环境的，也可以是如 Web 环境的；

​	Cryptography：对敏感信息加密，保护数据的安全性，如密码加密存储到数据库，而不是明文存储；

#### Shiro的核心组件

![avatar](https://z3.ax1x.com/2021/09/04/hgws2t.png)

- **Subject**：主体，可以看到主体可以是任何可以与应用交互的 “用户”；subject中包含用户的账号和密码，角色权限，是否登录状态等等一切信息，shiro只接收subject。web页面from表单传递username和pwd---->controller----->service---->shiro；
- **SecurityManager**：相当于 SpringMVC 中的 DispatcherServlet 或者 Struts2 中的 FilterDispatcher；是 Shiro 的心脏；所有具体的交互都通过 SecurityManager 进行控制；它管理着所有 Subject、且负责进行认证和授权、及会话、缓存的管理。
- **Realm**：可以有 1 个或多个 Realm（dataSource），可以认为是shiro安全实体数据源，即用于获取安全实体的；是shiro和数据之间桥梁。
- Shrio的工作流程

![avatar](https://z3.ax1x.com/2021/09/04/h2SwjJ.png)

![avatar](https://z3.ax1x.com/2021/09/04/h2svl9.png)