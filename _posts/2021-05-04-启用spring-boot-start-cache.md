---
layout: post
title:  "启用spring-boot-start-cache"
date:   2021-05-04 22:29:36 +0800
categories: springboot
typora-root-url: ..
---

# 启用spring-boot-start-cache
以下所有内容均可在[springboot-demo](https://github.com/zph-programmer/springboot)中找到。  

Spring Cache是Spring针对Spring应用，给出的一整套应用缓存解决方案。  
Spring Cache本身并不提供缓存实现，而是通过统一的接口和代码规范，配置、注解等使你可以在Spring应用中使用各种Cache，而不用太关心Cache的细节。通过Spring Cache，你可以方便的使用各种缓存实现，包括ConcurrentMap,Ehcache,JCache,Redis等。
## 所需依赖
在pom.xml引入maven依赖，注意是否存在依赖冲突。
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
```

## Spring中Cache的定义
Spring 中关于缓存的定义，包括在接口 org.springframework.cache.Cache 中，它主要提供了如下方法

```java
// 根据指定key获取值
<T> T get(Object key, Class<T> type)
// 将指定的值，根据相应的key，保存到缓存中
void put(Object key, Object value);
// 根据键，回收指定的值
void evict(Object key)
从定义中不难看着,Cache事实上就是一个key-value的结构，我们通过个指定的key来操作相应的value。
```
## Cache Manager
Cache是key-value的集合，但我们在项目中，可能会存在各种业务主题的不同的Cache，比如用户的cache,部门的Cache等，这些cache在逻辑上是分开的。为了区分这些Cache，提供了org.springframework.cache.CacheManager用来管理各种Cache.该接口只包含两个方法
```java
// 根据名字获取对应主题的缓存
Cache getCache(String name);
// 获取所有主题的缓存
Collection<String> getCacheNames();
```
在该接口中，不允许对Cache进行增加、删除操作，这些操作应该在各种CacheManager实现的内部完成，而不应该公开出来。
## 简单使用
定义一个缓存的操作接口
```java
public interface CacheService {

    class keyValue{
        public String key;
        public String value;
    }

    String get(String key)throws Exception;

    void set(String key,String value)throws Exception;

    void del(String key)throws Exception;

}
```
使用一个默认的缓存主题，实现get,set,del方法。
```java
import com.zph.programmer.springboot.cache.CacheService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cache.Cache;
import org.springframework.cache.CacheManager;
import org.springframework.stereotype.Service;

@Service
public class CacheServiceImpl implements CacheService {

    private final static String cacheName="spring-boot-demo-cache";
    @Autowired
    private CacheManager cacheManager;

    @Override
    public String get(String key)throws Exception {
        Cache cache = cacheManager.getCache(cacheName);
        if(cache!=null) {
            return cache.get(key, String.class);
        }
        else{
            throw new Exception("缓存"+cacheName+"异常！");
        }
    }

    @Override
    public void set(String key, String value)throws Exception {
        Cache cache = cacheManager.getCache(cacheName);
        if(cache!=null) {
            cache.put(key, value);
        }
        else{
            throw new Exception("缓存"+cacheName+"异常！");
        }
    }

    @Override
    public void del(String key) throws Exception{
        Cache cache = cacheManager.getCache(cacheName);
        if(cache!=null) {
            cache.evict(key);
        }
        else{
            throw new Exception("缓存"+cacheName+"异常！");
        }
    }
}
```
最后在启动类或者配置类里添加注解@EnableCaching即可。

## 其他
spring-boot默认的缓存是存储在ConcurrentHashMap里的，对于分布式应用应该引入redis等缓存来实现CacheManager。此外，spring-boot的缓存抽象还支持基于注解的使用方式，可参考
[官方文档——Cache Abstraction](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#cache)。  
个人并不喜欢滥用复杂的缓存方式，因为这样很容易导致数据一致性，缓存击穿，缓存穿透，缓存雪崩等一系列问题，而且还不便于查错，只用当出现性能问题时，才考虑能否用缓存解决，并且缓存的数据尽量简单，常用，一定要设置过期时间，因为大量的低使用率数据积累在缓存中，必然导致缓存太大影响程序的运行。（上面的cache暂时没有设置时间）

## 参考

一、缓存处理流程

    前台请求，后台先从缓存中取数据，取到直接返回结果，取不到时从数据库中取，数据库取到更新缓存，并返回结果，数据库也没取到，那直接返回空结果。


二、缓存穿透

    描述：

    缓存穿透是指缓存和数据库中都没有的数据，而用户不断发起请求，如发起为id为“-1”的数据或id为特别大不存在的数据。这时的用户很可能是攻击者，攻击会导致数据库压力过大。

    解决方案：

    接口层增加校验，如用户鉴权校验，id做基础校验，id<=0的直接拦截；
    从缓存取不到的数据，在数据库中也没有取到，这时也可以将key-value对写为key-null，缓存有效时间可以设置短点，如30秒（设置太长会导致正常情况也没法使用）。这样可以防止攻击用户反复用同一个id暴力攻击。
 

三、缓存击穿

    描述：

    缓存击穿是指缓存中没有但数据库中有的数据（一般是缓存时间到期），这时由于并发用户特别多，同时读缓存没读到数据，又同时去数据库去取数据，引起数据库压力瞬间增大，造成过大压力

    解决方案：

    设置热点数据长时间有效。
    从数据库获取数据时加互斥锁。

四、缓存雪崩

    描述：

    缓存雪崩是指缓存中数据大批量到过期时间，而查询数据量巨大，引起数据库压力过大甚至down机。和缓存击穿不同的是，缓存击穿指并发查同一条数据，缓存雪崩是不同数据都过期了，很多数据都查不到从而查数据库。

    解决方案：

    缓存数据的过期时间设置随机，防止同一时间大量数据过期现象发生。
    如果缓存数据库是分布式部署，将热点数据均匀分布在不同搞得缓存数据库中。
    设置热点数据长时间有效。
