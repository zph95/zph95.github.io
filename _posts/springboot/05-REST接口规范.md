---
layout: post
title:  "REST接口规范"
date:   2021-05-28 14:29:36 +0800
categories: springboot
---

# REST接口规范

# 使用spring-boot-start-web
以下所有内容均可在[springboot-demo](https://github.com/zph-programmer/springboot)中找到。  

REST接口规范只是个约定，并没有强制要求遵守。因为设计的再好，再严谨的规范也满足不了产品的快速变更，所以一千个项目就有一千种规范。
但有一个规范总比没有规范要好，哪怕项目经理就是要所有请求都走POST请求,所有的动作都在url中体现，所有的信息都用json存放在body种，
只要能严格执行，这也是可行的规范。

## 所需依赖
在pom.xml引入maven依赖，注意是否存在依赖冲突。
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```
## Controller层代码
这里使用上一篇的缓存来测试四种基本的REST接口。
```java
import com.zph.programmer.api.dto.BaseResponseDto;
import com.zph.programmer.springboot.cache.CacheService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping(value = "/test", name = "测试")
public class TestCtrl {

    @Autowired
    private final CacheService cacheService;

    public TestCtrl(CacheService cacheService) {
        this.cacheService = cacheService;
    }

    @GetMapping(value = "/testCache", name = "测试Cache get方法")
    public BaseResponseDto<String> testCacheGet(@RequestParam("key") String key) throws Exception {
        return BaseResponseDto.success(HttpStatus.OK.value(), cacheService.get(key));
    }

    @PostMapping(value = "/testCache", name = "测试Cache set方法")
    public BaseResponseDto<String> testCachePost(@RequestBody CacheService.KeyValue keyValue) throws Exception {
        cacheService.set(keyValue.getKey(), keyValue.getValue());
        return BaseResponseDto.success(HttpStatus.OK.value(),HttpStatus.OK.getReasonPhrase());
    }

    @PutMapping(value = "/testCache", name = "测试Cache put方法")
    public BaseResponseDto<String> testCachePut(@RequestBody CacheService.KeyValue keyValue) throws Exception {
        cacheService.set(keyValue.getKey(), keyValue.getValue());
        return BaseResponseDto.success(HttpStatus.OK.value(),HttpStatus.OK.getReasonPhrase());
    }

    @DeleteMapping(value = "/testCache", name = "测试Cache delete方法")
    public BaseResponseDto<String> testCacheDelete(@RequestParam("key") String key) throws Exception {
        cacheService.del(key);
        return BaseResponseDto.success(HttpStatus.OK.value(),HttpStatus.OK.getReasonPhrase());
    }
}
```
这里所有的返回值都用BaseResponseDto包装了一下。

```java
import lombok.Data;
import lombok.NoArgsConstructor;

@NoArgsConstructor
@Data
public class BaseResponseDto<T> {

    private final static  String success="success";

    private final static  String fail="fail";

    private final static  String error="error";

    /**
     * 包含一个整数类型的HTTP响应状态码。
     */
    private Integer code;

    /**
     * 包含文本："success"，"fail"或"error"。
     * HTTP状态响应码在500-599之间为"fail"，在400-499之间为"error"，
     * 其它均为"success"（例如：响应状态码为1XX、2XX和3XX）。
     */
    private String status;

    /**
     * 当状态值为"fail"和"error"时有效，用于显示错误信息。
     * 参照国际化（il8n）标准，它可以包含信息号或者编码，可以只包含其中一个，或者同时包含并用分隔符隔开。
     */
    private String message;

    /**
     * 当状态值为"fail"和"error"时有效，用于显示更多错误信息。主要用于分析问题使用
     */
    private Object moreInfo;

    /**
     * 包含响应的body。当状态值为"fail"或"error"时，data仅包含错误原因或异常名称
     */
    private T data;

    public BaseResponseDto(Integer code,String status,String message, T data){
        this.code=code;
        this.status=status;
        this.data=data;
        this.message=message;
    }

    public static <T> BaseResponseDto<T> success(Integer code,T data){
        return new BaseResponseDto<>(code,success,null,  data);
    }

    public static <T> BaseResponseDto<T> fail(Integer code,String message,T data){
        return new BaseResponseDto<>(code,fail,message,  data);
    }
    public static <T> BaseResponseDto<T> error(Integer code,String message,T data){
        return new BaseResponseDto<>(code,error,message, data);
    }
}
```

# 参考资料

## URI格式规范  

-   URI(Uniform Resource Identifiers) 统一资源标示符
-   URL(Uniform Resource Locator) 统一资源定位符  

URI的格式定义如下:  
URI = scheme "://" authority "/" path [ "?" query ] [ "#" fragment ]  
URL是URI的一个子集(一种具体实现)，对于REST API来说一个资源一般对应一个唯一的URI(URL)。在URI的设计中，我们会遵循一些规则，使接口看起透明易读，方便使用者调用。

-   关于分隔符“/”的使用  

"/"分隔符一般用来对资源层级的划分，例如 http://api.canvas.restapi.org/shapes/polygons/quadrilaterals/squares

对于REST API来说，"/"只是一个分隔符，并无其他含义。为了避免混淆，"/"不应该出现在URL的末尾。例如以下两个地址实际表示的都是同一个资源：  
http://api.canvas.restapi.org/shapes/  
http://api.canvas.restapi.org/shapes  

REST API对URI资源的定义具有唯一性，一个资源对应一个唯一的地址。为了使接口保持清晰干净，如果访问到末尾包含 "/" 的地址，服务端应该301到没有 "/"的地址上。当然这个规则也仅限于REST API接口的访问，对于传统的WEB页面服务来说，并不一定适用这个规则。

-   URI中尽量使用连字符"-"代替下划线"_"的使用  

连字符"-"一般用来分割URI中出现的字符串(单词)，来提高URI的可读性，例如：  
http://api.example.restapi.org/blogs/mark-masse/entries/this-is-my-first-post  

```
使用下划线"_"来分割字符串(单词)可能会和链接的样式冲突重叠，而影响阅读性。  
但实际上，"-"和"_"对URL中字符串的分割语意上还是有些差异的："-"分割的字符串(单词)一般各自都具有独立的含义，可参见上面的例子。而"_"一般用于对一个整体含义的字符串做了层级的分割，方便阅读，例如你想在URL中体现一个ip地址的信息：210_110_25_88
```

-   URI中统一使用小写字母  

根据RFC3986定义，URI是对大小写敏感的，所以为了避免歧义，我们尽量用小写字符。但主机名(Host)和scheme（协议名称:http/ftp/...）对大小写是不敏感的。

-   URI中不要包含文件(脚本)的扩展名  

例如 .php .json 之内的就不要出现了，对于接口来说没有任何实际的意义。如果是想对返回的数据内容格式标示的话，通过HTTP Header中的Content-Type字段更好一些。

-   CRUD的操作不要体现在URI中，HTTP协议中的操作符已经对CRUD做了映射。  

CRUD是创建，读取，更新，删除这四个经典操作的简称。

## HTTP交互设计

### HTTP请求方法的使用

-   GET方法用来获取资源
-   PUT方法用来新增Store类型的资源，更新一个资源
-   PATCH方法用来更新资源的部分属性。
-   POST方法用来创建一个资源，触发执行一个Controller类型资源
-   DELETE方法用于删除资源
-   HEAD：获取资源的元数据。
-   OPTIONS：获取信息，关于资源的哪些属性是客户端可以改变的。
-   TRACE：回显服务器收到的请求，主要用于测试或诊断。

一旦资源被删除，GET/HEAD方法访问被删除的资源时，要返回404
DELETE是一个比较纯粹的方法，我们不能对其做任何的重构或者定义，不可附加其它状态条件，如果我们希望"软"删除一个资源，则这种需求应该由Controller类资源来实现。 

## HTTP响应状态码的使用

- 200 (“OK”) 用于一般性的成功返回
- 200 (“OK”) 不可用于请求错误返回
- 201 (“Created”) 资源被创建
- 202 (“Accepted”) 用于Controller控制类资源异步处理的返回，仅表示请求已经收到。对于耗时比较久的处理，一般用异步处理来完成
- 204 (“No Content”) 此状态可能会出现在PUT、POST、DELETE的请求中，一般表示资源存在，但消息体中不会返回任何资源相关的状态或信息。
- 301 (“Moved Permanently”) 资源的URI被转移，需要使用新的URI访问
- 302 (“Found”) 不推荐使用，此代码在HTTP1.1协议中被303/307替代。我们目前对302的使用和最初HTTP1.0定义的语意是有出入的，应该只有在GET/HEAD方法下，客户端才能根据Location执行自动跳转，而我们目前的客户端基本上是不会判断原请求方法的，无条件的执行临时重定向
- 303 (“See Other”) 返回一个资源地址URI的引用，但不强制要求客户端获取该地址的状态(访问该地址)
- 304 (“Not Modified”) 有一些类似于204状态，服务器端的资源与客户端最近访问的资源版本一致，并无修改，不返回资源消息体。可以用来降低服务端的压力
- 307 (“Temporary Redirect”) 目前URI不能提供当前请求的服务，临时性重定向到另外一个URI。在HTTP1.1中307是用来替代早期HTTP1.0中使用不当的302
- 400 (“Bad Request”) 用于客户端一般性错误返回, 在其它4xx错误以外的错误，也可以使用400，具体错误信息可以放在body中
- 401 (“Unauthorized”) 在访问一个需要验证的资源时，验证错误
- 403 (“Forbidden”) 一般用于非验证性资源访问被禁止，例如对于某些客户端只开放部分API的访问权限，而另外一些API可能无法访问时，可以给予403状态
- 404 (“Not Found”) 找不到URI对应的资源
- 405 (“Method Not Allowed”) HTTP的方法不支持，例如某些只读资源，可能不支持POST/DELETE。但405的响应header中必须声明该URI所支持的方法
- 406 (“Not Acceptable”) 客户端所请求的资源数据格式类型不被支持，例如客户端请求数据格式为application/xml，但服务器端只支持application/json
- 409 (“Conflict”) 资源状态冲突，例如客户端尝试删除一个非空的Store资源
- 412 (“Precondition Failed”) 用于有条件的操作不被满足时
- 415 (“Unsupported Media Type”) 客户所支持的数据类型，服务端无法满足
- 500 (“Internal Server Error”) 服务器端的接口错误，此错误于客户端无关

## 原数据设计

### http headers

-   Content-Type 标示body的数据格式
-   Content-Length body 数据体的大小，客户端可以根据此标示检验读取到的数据是否完整，也可以通过Header判断是否需要下载可能较大的数据体
-   Last-Modified 用于服务器端的响应，是一个资源最后被修改的时间戳，客户端(缓存)可以根据此信息判断是否需要重新获取该资源
-   ETag 服务器端资源版本的标示，客户端(缓存)可以根据此信息判断是否需要重新获取该资源，需要注意的是，ETag如果通过服务器随机生成，可能会存在多个主机对同一个资源产生不同ETag的问题
-   Location 在响应header中使用，一般为客户端感兴趣的资源URI,例如在成功创建一个资源后，我们可以把新的资源URI放在Location中，如果是一个异步创建资源的请求，接口在响应202 (“Accepted”)的同时可以给予客户端一个异步状态查询的地址

-   Cache-Control, Expires, Date 通过缓存机制提升接口响应性能,同时根据实际需要也可以禁止客户端对接口请求做缓存。对于REST接口来说，如果某些接口实时性要求不高的情况下，我们可以使用max-age来指定一个小的缓存时间，这样对客户端和服务器端双方都是有利的。一般来说只对GET方法且返回200的情况下使用缓存，在某些情况下我们也可以对返回3xx或者4xx的情况下做缓存，可以防范错误访问带来的负载。

### 媒体类型(Media Type)

定义如下：
Content-Type: type "/" subtype *( ";" parameter )  
两个实例：  
Content-type: text/html; charset=ISO-8859-4  
Content-type: text/plain; charset="us-ascii"   
type 主类型一般为：application, audio, image, message, model, multipart, text, video。REST接口的主类型一般使用application

* [返回目录](https://zph-programmer.github.io)
    * [上一篇 —— 启用spring-boot-start-cache](04-启用spring-boot-start-cache.md)
    * [下一篇 —— 使用过滤器打印Rest接口日志](06-使用过滤器打印Rest接口日志.md)