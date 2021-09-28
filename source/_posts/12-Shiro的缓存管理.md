---
title: 12-Shiro的缓存管理
date: 2021-09-12 16:21:45
tags: shiro
categories: shiro权限框架
cover: https://s3.bmp.ovh/imgs/2021/09/4ffd9cf6a603d931.jpg
---

#### 场景描述

​	现在再添加量Shiro标签后，以HTML形式进行权限管理。是否每一次点击页面的某个菜单，都要经过MyRealm的授权方法，从数据库里查数据。在MyRealm的doGetAuthorizationInfo打印一行sout（“----------查询用户权限----”）；看看登录成功后，引入index.html这句话打印了几次。现在用户zhoudu登录。

##### 检测结果

![avatar](https://z3.ax1x.com/2021/09/12/4phdbt.png)

![avatar](https://z3.ax1x.com/2021/09/12/4p4CxH.png)

![avatar](https://z3.ax1x.com/2021/09/12/4pIFEt.png)

![avatar](https://z3.ax1x.com/2021/09/12/4pI35V.png)

​	由上图所示，在初始index.html时一共16个shiro标签，所以执行了16次授权方法。并且每次点击一个菜单又会调用授权方法。如点击“客户查询”，虽然没有用shiro标签修饰但是加上了权限判断的注解，也会打印"----------查询用户权限----===")。如下图。

![avatar](https://z3.ax1x.com/2021/09/12/4pTemj.png)

​	说明无论是初始化，还是点击某个菜单都会查询数据库的数据，而且这些查询是同样的重复的操作，当用户很多时用户点击量很大时会频繁的访问数据库，非常消耗数据库的性能，所以在初始化界面时把数据从数据库里读出来存入缓存中，下次点击某个菜单直接从缓存里寻找，这样性能提高很多。

#### 使用缓存

##### 引入依赖

​	首页springboot要支持缓存，在引入第三方缓存，然后shiro支持缓存

```xml
<dependency>
            <groupId>org.apache.shiro</groupId>
            <artifactId>shiro-ehcache</artifactId>
            <version>1.4.0</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-cache</artifactId>
        </dependency>
        <dependency>
            <groupId>net.sf.ehcache</groupId>
            <artifactId>ehcache</artifactId>
        </dependency>
```

##### 设置缓存策略

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ehcache updateCheck="false" dynamicConfig="false">
    <!--持久化磁盘路径-->
    <diskStore path="java.io.tmpdir"/>
    <cache name="users" timeToLiveSeconds="300" maxEntriesLocalHeap="1000"/>
    <defaultCache    maxElementsInMemory="1000"
                     eternal="false"
                     timeToIdleSeconds="3600"
                     timeToLiveSeconds="0"
                     overflowToDisk="true"
                     maxElementsOnDisk="10000"
                     diskPersistent="false"
                    diskExpiryThreadIntervalSeconds="120"
                     memoryStoreEvictionPolicy="LRU"
    />
</ehcache>
```

##### 配置ShiroConfig

```java
@Bean
    public EhCacheManager getEhCacheManager() {
        EhCacheManager ehCacheManager = new EhCacheManager();
        ehCacheManager.setCacheManagerConfigFile("classpath:ehcache.xml");
        return ehCacheManager;
    }
    @Bean
    public DefaultWebSecurityManager getDefaultWebSecurityManager(MyRealm myRealm, EhCacheManager ehCacheManager) {
        DefaultWebSecurityManager defaultWebSecurityManager = new DefaultWebSecurityManager();
        defaultWebSecurityManager.setRealm(myRealm);
        defaultWebSecurityManager.setCacheManager(ehCacheManager);
        return defaultWebSecurityManager;
    }
```

