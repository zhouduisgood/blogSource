---
title: 6-自定义Realm的数据表设计
date: 2021-09-08 11:10:56
tags: shiro
categories: shiro权限框架
cover: https://s3.bmp.ovh/imgs/2021/09/4ffd9cf6a603d931.jpg
---

#### 创建表

​	根据RBAC权限设计，至少需要5张表。

| 解释       | 表名              | 描述                                 |
| ---------- | ----------------- | ------------------------------------ |
| 用户表     | users             | 自增id,账号，密码，加盐密码          |
| 角色表     | roles             | 自增role_id,角色名称                 |
| 权限表     | permissions       | 自增permission_id,权限代码，权限名称 |
| 用户角色表 | users_roles       | 用户id，角色id，                     |
| 角色权限表 | roles_permissions | 角色id，权限id                       |

##### 用户表

```sql
create table users(
	id int PRIMARY key auto_increment,
	username varchar(60) not null unique,
	password varchar(20) not null,
	password_salt varchar(60)
)charset=utf8 ENGINE=InnoDB;
insert into users(username,password) values('zhoudu','123456');
insert into users(username,password) values('wangtaiwen','321654');
insert into users(username,password) values('tangyifu','741852');
insert into users(username,password) values('yangtian','963852');
insert into users(username,password) values('caiyonghu','789456');
```

##### 角色表

```sql
create table roles(
	role_id int primary key auto_increment,
	role_name varchar(60) not null
)charset=utf8 ENGINE=InnoDB;
insert into roles(role_name) values('admin');  -- 超级管理员
insert into roles(role_name) values('HR');     -- 人事 
insert into roles(role_name) values('seller'); -- 销售
insert into roles(role_name) values('repoManager'); -- 库管
insert into roles(role_name) values('planner'); -- 策划
```

##### 权限表

```sql
create table permissions(
	permission_id int primary key auto_increment, -- 1
	permission_code varchar(60) not null,         -- sys:repo:find
	permission_name varchar(60) 				-- 库存查询
)charset=utf8 ENGINE=InnoDB;

insert into permissions (permission_code,permission_name) values('sys:repo:save','添加库存');
insert into permissions (permission_code,permission_name) values('sys:repo:delete','减少库存');
insert into permissions (permission_code,permission_name) values('sys:repo:update','修改库存');
insert into permissions (permission_code,permission_name) values('sys:repo:find','查询库存');

insert into permissions (permission_code,permission_name) values('sys:order:save','添加订单');
insert into permissions (permission_code,permission_name) values('sys:order:delete','减少订单');
insert into permissions (permission_code,permission_name) values('sys:order:update','修改订单');
insert into permissions (permission_code,permission_name) values('sys:order:find','查询订单');

insert into permissions (permission_code,permission_name) values('sys:customer:save','添加客户');
insert into permissions (permission_code,permission_name) values('sys:customer:delete','减少客户');
insert into permissions (permission_code,permission_name) values('sys:customer:update','修改客户');
insert into permissions (permission_code,permission_name) values('sys:customer:find','查询客户');
```

##### 用户角色表

​	为某个用户分配某个角色，一个用户可以对用多个角色，一个角色也可由对应多个用户。拥有某个角色就等于拥有这个角色的所有信息，比如id=1，username=zhoudu是超级管理员，就给zhoudu分配所有角色，有HR、seller、repoManager、planner。除了id=1，其它id要和角色表role_id对应。比如id=2的wangtaiwen对应人事HR，id=3的tangyifu对应销售。等等

```sql
create table users_roles(
	uid int not null,
	rid int not null
	-- primary key(uid,rid),
	-- constraint FK_user foreign key(uid) references users(id);
	-- constraint FK_roles foreign key(rid) references roles(role_id);
)charset=utf8 ENGINE=InnoDB;
insert into users_roles(uid,rid) values(1,1);
insert into users_roles(uid,rid) values(1,2);
insert into users_roles(uid,rid) values(1,3);
insert into users_roles(uid,rid) values(1,4);
insert into users_roles(uid,rid) values(1,5);
insert into users_roles(uid,rid) values(2,2);
insert into users_roles(uid,rid) values(3,3);
insert into users_roles(uid,rid) values(4,4);
insert into users_roles(uid,rid) values(5,5);
```

##### 角色权限表

​	为某个角色分配一些权限，一些功能。某个角色可以有多个权限也可以有一个权限。先规定库管有库存的增删改查；销售人员有订单的增删改查和客户的增删改查和添加库存功能。人事HR有修改客户和查询客户。为从策划planner分配存库、订单、客户的查询。

```sql
create table roles_permissions(
	rid int not null,
	pid int not null
)charset=utf8 ENGINE=InnoDB;
truncate table roles_permissions;
-- 为库管rid=4 设置权限
insert into roles_permissions(rid,pid) values(4,1);
insert into roles_permissions(rid,pid) values(4,2);
insert into roles_permissions(rid,pid) values(4,3);
insert into roles_permissions(rid,pid) values(4,4);
-- 为销售rid=3 设置权限
insert into roles_permissions(rid,pid) values(3,5);
insert into roles_permissions(rid,pid) values(3,6);
insert into roles_permissions(rid,pid) values(3,7);
insert into roles_permissions(rid,pid) values(3,8);
insert into roles_permissions(rid,pid) values(3,9);
insert into roles_permissions(rid,pid) values(3,10);
insert into roles_permissions(rid,pid) values(3,11);
insert into roles_permissions(rid,pid) values(3,12);
insert into roles_permissions(rid,pid) values(3,4);
-- 为HR rid=2 设置权限
insert into roles_permissions(rid,pid) values(2,11);
insert into roles_permissions(rid,pid) values(2,12);
-- 为策划 rid=5 设置权限
insert into roles_permissions(rid,pid) values(5,4);
insert into roles_permissions(rid,pid) values(5,8);
insert into roles_permissions(rid,pid) values(5,12);
```

