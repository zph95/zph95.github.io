---

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
个人并不喜欢滥用复杂的缓存方式，因为这样很容易导致数据一致性，缓存击穿，缓存穿透，缓存雪崩等一系列问题，而且还不便于查错，只用当出现性能问题时，才考虑能否用缓存解决，并且缓存的数据尽量简单，常用，一定要设置过期时间，因为大量的低使用率数据积累在缓存中，必然导致缓存太大影响程序的运行。



## 整合EhCache

Ehcache 是一个纯Java开源缓存框架，配置简单、结构清晰、功能强大，非常轻量级的缓存实现。

官网地址[Ehcache](https://www.ehcache.org/)

## 所需依赖

在pom.xml引入maven依赖

```xml
<dependency>
            <groupId>net.sf.ehcache</groupId>
            <artifactId>ehcache</artifactId>
            <version>2.10.9.2</version>
        </dependency>
```

## 配置文件

在resource目录下添加ehcache.xml。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="http://ehcache.org/ehcache.xsd">
    <!--
           diskStore:为缓存路径，ehcache分为内存和磁盘 2级，此属性定义磁盘的缓存位置
           user.home - 用户主目录
           user.dir - 用户当前工作目录
           java.io.tmpdir - 默认临时文件路径
       -->
    <!-- 磁盘缓存位置 -->
    <diskStore path="java.io.tmpdir/ehcache"/>

    <!-- 默认缓存 -->
    <defaultCache
            maxEntriesLocalHeap="10000"
            eternal="false"
            timeToIdleSeconds="120"
            timeToLiveSeconds="120"
            maxEntriesLocalDisk="10000000"
            diskExpiryThreadIntervalSeconds="120"
            memoryStoreEvictionPolicy="LRU">
        <persistence strategy="localTempSwap"/>
    </defaultCache>

    <cache name="ehcache" maxEntriesLocalHeap="10000"
           eternal="false"
           timeToIdleSeconds="120"
           timeToLiveSeconds="120"
           maxEntriesLocalDisk="10000000"
           diskExpiryThreadIntervalSeconds="120"
           memoryStoreEvictionPolicy="LRU">
        <persistence strategy="localTempSwap"/>
    </cache>
    <!--
            name:缓存名称。
            maxElementsInMemory:缓存最大数目
            maxElementsOnDisk：硬盘最大缓存个数。
            eternal:对象是否永久有效，一但设置了，timeout将不起作用。
            overflowToDisk:是否保存到磁盘，当系统宕机时
            timeToIdleSeconds:设置对象在失效前的允许闲置时间（单位：秒）。仅当eternal=false对象不是永久有效时使用，可选属性，默认值是0，也就是可闲置时间无穷大。
            timeToLiveSeconds:设置对象在失效前允许存活时间（单位：秒）。最大时间介于创建时间和失效时间之间。仅当eternal=false对象不是永久有效时使用，默认是0.，也就是对象存活时间无穷大。
            diskPersistent：是否缓存虚拟机重启期数据 Whether the disk store persists between restarts of the Virtual Machine. The default value is false.
            diskSpoolBufferSizeMB：这个参数设置DiskStore（磁盘缓存）的缓存区大小。默认是30MB。每个Cache都应该有自己的一个缓冲区。
            diskExpiryThreadIntervalSeconds：磁盘失效线程运行时间间隔，默认是120秒。
            memoryStoreEvictionPolicy：当达到maxElementsInMemory限制时，Ehcache将会根据指定的策略去清理内存。默认策略是LRU（最近最少使用）。你可以设置为FIFO（先进先出）或是LFU（较少使用）。
            clearOnFlush：内存数量最大时是否清除。
            memoryStoreEvictionPolicy:可选策略有：LRU（最近最少使用，默认策略）、FIFO（先进先出）、LFU（最少访问次数）。
                FIFO，first in first out，这个是大家最熟的，先进先出。
                LFU， Less Frequently Used，就是上面例子中使用的策略，直白一点就是讲一直以来最少被使用的。如上面所讲，缓存的元素有一个hit属性，hit值最小的将会被清出缓存。
                LRU，Least Recently Used，最近最少使用的，缓存的元素有一个时间戳，当缓存容量满了，而又需要腾出地方来缓存新的元素的时候，那么现有缓存元素中时间戳离当前时间最远的元素将被清出缓存。
    -->

</ehcache>
```

## 整合springboot-cache

```java
@Configuration
public class EhCacheConf {
    @Bean
    public EhCacheManagerFactoryBean ehCacheManagerFactoryBean() {
        EhCacheManagerFactoryBean cacheManagerFactoryBean = new EhCacheManagerFactoryBean();
        cacheManagerFactoryBean.setConfigLocation(new ClassPathResource("ehcache.xml"));
        cacheManagerFactoryBean.setShared(true);
        return cacheManagerFactoryBean;
    }

    @Bean
    public EhCacheCacheManager eCacheCacheManager(EhCacheManagerFactoryBean bean) {
        return new EhCacheCacheManager(bean.getObject());
    }
}
```



## 简单使用

```java
@Service("ehcache")
public class EhcacheServiceImpl implements CacheService {
    private final static String ehcacheName = "ehcache";
    @Autowired
    private EhCacheCacheManager ehCacheCacheManager;

    @Override
    public String get(String key) throws CacheException {
        Cache cache = ehCacheCacheManager.getCacheManager().getCache(ehcacheName);
        if (cache != null) {
            Element element = cache.get(key);
            return element == null ? null : String.valueOf(element.getObjectValue());
        } else {
            throw new CacheException("缓存" + ehcacheName + "异常！");

        }
    }

    @Override
    public void set(String key, String value, int second) throws CacheException {
        Cache cache = ehCacheCacheManager.getCacheManager().getCache(ehcacheName);
        if (cache != null) {

            Element element = new Element(key, value, second, second);

            /**
             * element
             * timeToLiveSeconds=x：缓存自创建日期起至失效时的间隔时间x；
             *
             * timeToIdleSeconds=y：缓存创建以后，最后一次访问缓存的日期至失效之时的时间间隔y；
             * 若自创建缓存后一直都没有访问缓存，那么间隔x后失效，若自创建缓存后有N次访问缓存，那么计算（最后一次访问缓存时间+y ） 即：按照timeToIdleSeconds计算，但总存活时间不超过 y;举个例子：
             *
             * timeToIdleSeconds=120；
             *
             * timeToLiveSeconds=180；
             *
             * 上面的表示此缓存最多可以存活3分钟，如果期间超过2分钟未访问 那么此缓存失效！
             */
            cache.put(element);
        } else {
            throw new CacheException("缓存" + ehcacheName + "异常！");
        }
    }

    @Override
    public void del(String key) throws CacheException {
        Cache cache = ehCacheCacheManager.getCacheManager().getCache(ehcacheName);
        if (cache != null) {
            cache.remove(key);
        } else {
            throw new CacheException("缓存" + ehcacheName + "异常！");

        }
    }
}

```



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
