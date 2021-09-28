---
title: 4-Shiro的JdbcRealm介绍和表结构
date: 2021-09-07 14:56:56
tags: shiro
categories: shiro权限框架
cover: https://s3.bmp.ovh/imgs/2021/09/4ffd9cf6a603d931.jpg
---

#### JdbcRealm简介

​	JdbcRealm是shiro框架内置的安全数据源，需要连接数据库才能使用。连接的数据库的表结构必须遵循某种规范。

#### jdbcRealm表结构规范

​	至少包含三张表，并且表名和字段名都是固定的。如下

```sql
create table users (
  id bigint auto_increment,
  username varchar(100),
  password varchar(100),
  password_salt varchar(100),
  constraint pk_users primary key(id)
) charset=utf8 ENGINE=InnoDB;
create unique index idx_users_username on users(username);

create table user_roles(
  id bigint auto_increment,
  username varchar(100),
  role_name varchar(100),
  constraint pk_user_roles primary key(id)
) charset=utf8 ENGINE=InnoDB;
create unique index idx_user_roles on user_roles(username, role_name);

create table roles_permissions(
  id bigint auto_increment,
  role_name varchar(100),
  permission varchar(100),
  constraint pk_roles_permissions primary key(id)
) charset=utf8 ENGINE=InnoDB;
create unique index idx_roles_permissions on roles_permissions(role_name, permission);
```

#### 初始化表数据

##### 	用户信息表users

```sql
INSERT into users(username,password) values('zhangsan','123456');
INSERT into users(username,password) values('lisi','123456');
INSERT into users(username,password) values('wangwu','55555');
INSERT into users(username,password) values('zhaoliu','666666');
INSERT into users(username,password) values('chengqi','77777');
```

##### 用户角色表user_roles

```sql
-- admin 超级管理员；seller销售员；repoManager仓库管理员 ；customerstaff 客服；repairman 维修工
INSERT into user_roles(username,role_name) values('zhangsan','admin');
INSERT into user_roles(username,role_name) values('lisi','seller');
INSERT into user_roles(username,role_name) values('wangwu','repoManager');
INSERT into user_roles(username,role_name) values('zhaoliu','customerstaff');
INSERT into user_roles(username,role_name) values('chengqi','repairman');
```

##### 角色权限表roles_permissions

```sql
-- 管理员有所有权限
insert into roles_permissions(role_name,permission) values('admin','*');

-- 库管员有仓库的增删改查
insert into roles_permissions(role_name,permission) values('repoManager','sys:repo:save');
insert into roles_permissions(role_name,permission) values('repoManager','sys:repo:delete');
insert into roles_permissions(role_name,permission) values('repoManager','sys:repo:update');
insert into roles_permissions(role_name,permission) values('repoManager','sys:repo:find');

-- 销售员有销售订单的增删改查和客服人员的增删改查和 仓库的查询
insert into roles_permissions(role_name,permission) values('seller','sys:seller:save');
insert into roles_permissions(role_name,permission) values('seller','sys:seller:delete');
insert into roles_permissions(role_name,permission) values('seller','sys:seller:update');
insert into roles_permissions(role_name,permission) values('seller','sys:seller:find');
insert into roles_permissions(role_name,permission) values('seller','sys:repo:find');
insert into roles_permissions(role_name,permission) values('seller','sys:customerstaff:save');
insert into roles_permissions(role_name,permission) values('seller','sys:customerstaff:delete');
insert into roles_permissions(role_name,permission) values('seller','sys:customerstaff:update');
insert into roles_permissions(role_name,permission) values('seller','sys:customerstaff:find');

-- 客服有客户的查询和修改
insert into roles_permissions(role_name,permission) values('customerstaff','sys:customerstaff:find');
insert into roles_permissions(role_name,permission) values('customerstaff','sys:customerstaff:update');
-- 修理工有所有的查询权限
insert into roles_permissions(role_name,permission) values('repairman','sys:*:find');
```

