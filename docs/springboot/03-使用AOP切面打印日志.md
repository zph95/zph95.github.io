#   使用AOP切面打印日志
以下所有内容均可在[springboot-demo](https://github.com/zph-programmer/springboot)中找到。

## 所需依赖
在pom.xml引入maven依赖，注意是否存在依赖冲突。
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

## 定义一个注解PointLog
```java
import java.lang.annotation.*;

@Target({ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface PointLog {

    String value() default "";
}

```

## 配置切点操作
主要内容是使用切点在方法执行前打印入参日志，执行后打印返回结果日志，抛出异常时打印异常日志。
```java

import lombok.extern.slf4j.Slf4j;
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.*;
import org.springframework.stereotype.Component;

import java.lang.reflect.Method;
import java.time.LocalDateTime;
import java.util.Arrays;

@Aspect
@Slf4j
@Component
public class PointLogAspect {
    /**
     * 注意修改包路径
     */
    @Pointcut("@annotation(com.zph.programmer.springboot.annotation.PointLog)")
    public void pointAspect() {

    }

    /**
     * 前置通知 用于拦截记录日志
     *
     * @param joinPoint 切点
     */
    @Before("pointAspect()")
    public void doLogBefore(JoinPoint joinPoint) {
        try {
            //类名
            String className = joinPoint.getTarget().getClass().getName();
            //请求方法
            String method = joinPoint.getSignature().getName() + "()";
            //方法参数
            String methodParam = Arrays.toString(joinPoint.getArgs());

            //方法描述
            String methodDescription = getMethodDescription(joinPoint);
            String sb = "\n********************************* "+LocalDateTime.now().toString() + "\n" +
                    "ClassName     :  " + className +"\n"+
                    "Method        :  " + method +"\n" +
                    "Description   :  " + methodDescription + "\n" +
                    "RequestParams :  " + methodParam + "\n" +
                    "*********************************  Start\n";
            log.info(sb);
        } catch (Exception e) {
            log.error("拦截记录日志异常:",e);
        }
    }
    /**
     * 返回结果通知 用于拦截记录结果日志
     */
    @AfterReturning(returning = "ret", pointcut = "pointAspect()")
    public void doLogAfterReturning(JoinPoint joinPoint, Object ret) {
        try {
            //类名
            String className = joinPoint.getTarget().getClass().getName();
            //请求方法
            String method = joinPoint.getSignature().getName() + "()";
            //方法描述
            String methodDescription = getMethodDescription(joinPoint);
            String sb = "\n********************************* " + LocalDateTime.now().toString() + "\n" +
                    "ClassName     :  " + className + "\n" +
                    "Method        :  " + method + "\n" +
                    "Description   :  " + methodDescription + "\n" +
                    "Return        :  " + ret.toString() + "\n" +
                    "********************************* End\n";
            log.info(sb);
        } catch (Exception e) {
            log.error("拦截记录日志异常:",e);
        }
    }

    /**
     * 异常通知 用于拦截记录异常日志
     */
    @AfterThrowing(pointcut = "pointAspect()", throwing = "ex")
    public void doLogAfterThrowing(JoinPoint joinPoint, Throwable ex) {
        try {
            //类名
            String className = joinPoint.getTarget().getClass().getName();
            //请求方法
            String method = joinPoint.getSignature().getName() + "()";
            //方法描述
            String methodDescription = getMethodDescription(joinPoint);
            //方法参数
            String methodParam = Arrays.toString(joinPoint.getArgs());
            String sb =  "\n********************************* "+LocalDateTime.now().toString() + "\n" +
                    "ClassName     :  " + className +"\n"+
                    "Method        :  " + method +"\n" +
                    "Description   :  " + methodDescription + "\n" +
                    "RequestParams :  " + methodParam + "\n" +
                    "ExceptionName    :  " + ex.getClass().getName() + "\n" +
                    "ExceptionMessage :  " + ex.getMessage() + "\n" +
                    "********************************* End\n";
            log.error(sb,ex);
        } catch (Exception e1) {
            log.error("拦截记录日志异常 : ",e1);
        }
    }

    /**
     * 获取注解中对方法的描述信息
     *
     * @param joinPoint 切点
     * @return 方法描述
     */
    private static String getMethodDescription(JoinPoint joinPoint) throws Exception {
        String targetName = joinPoint.getTarget().getClass().getName();
        String methodName = joinPoint.getSignature().getName();
        Object[] arguments = joinPoint.getArgs();
        Class targetClass = Class.forName(targetName);
        Method[] methods = targetClass.getMethods();
        String description = "";
        for (Method method : methods) {
            if (method.getName().equals(methodName)) {
                Class[] clazzs = method.getParameterTypes();
                if (clazzs.length == arguments.length) {
                    description = method.getAnnotation(PointLog.class).value();
                    break;
                }
            }
        }
        return description;
    }
}

```

## 使用方式
在要打印日志的方法上使用@PointLog("描述")
```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;
import com.zph.programmer.springboot.annotation.PointLog;

import java.util.HashMap;
import java.util.Map;

@Slf4j
@Component
public class TestService {

    @PointLog("测试注解@PointLog")
    public Map<String,String> testPointLog(String param) throws Exception{
        if(param==null){
            throw new Exception("参数为空");
        }
        else{
            log.info("param={}", param);
            Map<String,String> result=new HashMap<>();
            result.put("200","success");
            return result;
        }
    }
}
```

## 呈现效果
![PointLog效果图](images/PointLog效果图.png)

## 其它
可针对controller专门定义注解，打印url，request参数,response返回值等。

* [返回目录](https://zph-programmer.github.io)
    * [上一篇 —— spring-boot-starter-logging日志配置.md](02-spring-boot-starter-logging日志配置.md)
    * [下一篇 —— 启用spring-boot-start-cache](04-启用spring-boot-start-cache.md)