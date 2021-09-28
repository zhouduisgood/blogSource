---
title: 7-自定义Realm的DAO操作
date: 2021-09-08 18:49:13
tags: shiro
categories: shiro权限框架
cover: https://s3.bmp.ovh/imgs/2021/09/4ffd9cf6a603d931.jpg
---

#### dao流程的分析

​	由于这次自定义myRealm,而shiro自带JdbcRealm帮我们封装好了，负责认证和授权过程sql查询。所以自定义myRealm也要写这些SQL语句。如下

| myRealm也要完成认证和授权的SQL                       |
| ---------------------------------------------------- |
| 1。 认证（登录时）验证查询                           |
| 根据username查询用户信息。简单的单表查询             |
| 2。授权查询                                          |
| 根据用户名查询该用户的角色信息                       |
| 根据用户名查询该用户的权限信息                       |
| ![avatar](https://z3.ax1x.com/2021/09/08/hbm66x.png) |

#### 我的SQL

##### 认证SQL

​	单表查询根据username查询用户信息

```sql
select u.id from users u where u.username = 'zhoudu';
```

##### 授权SQL

​	  根据用户名查询该用户的角色信息，①先查用户角色表中该用户的角色编号。②再根据role_id联合查询用户表、角色表，并且2个表的角色id一样。

```sql
第一步 
SELECT
	ur.rid FROM users_roles ur 
WHERE
	ur.uid = (	SELECT	u.id FROM users u WHEREu.username = 'zhoudu')

第二步  
SELECT
	u.username,r.role_name FROM roles r, users u 
WHERE
	r.role_id IN (
	SELECT ur.rid FROM	users_roles ur 
        WHERE
	ur.uid = ( SELECT u.id FROM users u WHERE u.username = 'zhoudu' )) AND u.username = 'zhoudu';
```

​	根据用户名查询权限信息。①先查用户角色表中该用户的角色编号，②再根据rid关联角色权限表、权限表、用户表，并且角色权限表中的rid和①的rid一样，角色权限表中的pid和权限表permission_id一样

```sql
第一步 
SELECT
	ur.rid FROM users_roles ur 
WHERE
	ur.uid = (	SELECT	u.id FROM users u WHEREu.username = 'zhoudu')
第二步
SELECT DISTINCT
	rp.pid, p.permission_code, p.permission_name, u.username 
FROM
	roles_permissions rp, permissions p, users u 
WHERE
	rp.rid IN (
	SELECT ur.rid FROM users_roles ur WHERE
	ur.uid = ( SELECT u.id FROM users u WHERE u.username = 'zhoudu' )) AND p.permission_id = rp.pid AND u.username = 'zhoudu';
```

#### 改进的SQL

​	通过username查询到角色名称。使用连接查询的方法(涉及3张表，users、users_roles、roles)。SQL如下

```sql
SELECT
	u.username, r.role_name 
FROM
	users u
	INNER JOIN users_roles ur ON u.id = ur.uid
	INNER JOIN roles r ON r.role_id = ur.rid 
WHERE u.username = 'zhoudu';
```

​	通过username查询权限信息，关联5张表

```sql
select u.username, p.permission_code 
from 
users u inner join users_roles ur on u.id=ur.uid 
				inner join roles r on r.role_id=ur.rid
				inner join roles_permissions rp on rp.rid=r.role_id
				inner join permissions p on p.permission_id=rp.pid
where u.username='zhoudu';
```

#### Dao流程

​	以查询权限为例，先创建PermissionDao和对应的Mapper.xml。因为application.yml中设置mappers和生成beans的路径。所以PermissionMapper.xml必须在resource的mappers包下，而实体类必须在beans包下

![avatar](https://z3.ax1x.com/2021/09/09/hq7Ndg.png)

​	并在启动配置类中SpringbootAndShiroApplication类中配置mapper扫描注解。指定dao的SQL语句，而不用在每个dao接口上写@Mapper，所以的SpringbootAndShiroApplication类的上方加上@MapperScan(basePackages = "com.example.springbootandshiro.dao")。有我们只需要权限代码不需要权限的其它信息，所以PermissionMapper.xml的返回值是Set<String>类型。string--->”sys:order:add“等等

##### PermissionDao

```java
public interface PermissionDao {
    public Set<String> queryPermissionCodeByUsername(String username);
}
```

##### PermissionMapper

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.springbootandshiro.dao.PermissionDao">
    <select id="queryPermissionCodeByUsername" resultSets="java.util.Set" resultType="string">
        select p.permission_code
        from
            users u inner join users_roles ur on u.id=ur.uid
                    inner join roles r on r.role_id=ur.rid
                    inner join roles_permissions rp on rp.rid=r.role_id
                    inner join permissions p on p.permission_id=rp.pid
        where u.username=#{username};
    </select>
</mapper>
```

##### PermissionDaoTest

```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = SpringBootAndShiroApplication.class)
public class PermissionDaoTest {
    @Resource
    private PermissionDao permissionDao;
    @Test
    public void queryRolenameByUsername() {
        Set<String> permissionCodes = permissionDao.queryPermissionCodeByUsername("zhoudu");
        Iterator<String> it = permissionCodes.iterator();
        while(it.hasNext()) {
            System.out.println(it.next());
        }
    }
}
```

##### 项目结构与测试结果

| ![avatar](https://z3.ax1x.com/2021/09/09/hqLfHK.png) |
| ---------------------------------------------------- |
| ![avatar](https://z3.ax1x.com/2021/09/09/hqOnC4.png) |

