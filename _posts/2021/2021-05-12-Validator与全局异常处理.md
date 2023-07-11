---

title:  "Validator与全局异常处理"
date:   2021-05-11 14:29:36 +0800
categories: springboot
typora-root-url: ..
---

#  Validator与全局异常处理

开发过程中，后台的参数校验是必不可少的。如果对每一个参数都写if,else判断就太不优雅了。

 JSR-303 (Java Specification Requests）指定的Validator规范可以很好的解决该问题。

**Spring Validator**和**Hibernate Validator**是两套Validator，可以混着用。

官方文档：

- Spring Validator : [Core Technologies (spring.io)](https://docs.spring.io/spring-framework/docs/5.2.9.RELEASE/spring-framework-reference/core.html#validation-beanvalidation)
- Hibernate Validator: [Hibernate Validator 7.0.1.Final - Jakarta Bean Validation Reference Implementation: Reference Guide (jboss.org)](https://docs.jboss.org/hibernate/stable/validator/reference/en-US/html_single/#validator-gettingstarted)

## 所需依赖

```xml
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

以下转载于：[SpringBoot 参数校验的方法 - 木白的菜园 - 博客园 (cnblogs.com)](https://www.cnblogs.com/mooba/p/11276062.html)

## Validator注解

### 1. Bean Validation 中内置的 constraint

|   注解   |   作用   |
| ---- | ---- |
|@Valid	|被注释的元素是一个对象，需要检查此对象的所有字段值|
|@Null	|被注释的元素必须为 null|
|@NotNull	|被注释的元素必须不为 null|
|@AssertTrue	|被注释的元素必须为 true|
|@AssertFalse	|被注释的元素必须为 false|
|@Min(value)	|被注释的元素必须是一个数字，其值必须大于等于指定的最小值|
|@Max(value)	|被注释的元素必须是一个数字，其值必须小于等于指定的最大值|
|@DecimalMin(value)	|被注释的元素必须是一个数字，其值必须大于等于指定的最小值|
|@DecimalMax(value)	|被注释的元素必须是一个数字，其值必须小于等于指定的最大值|
|@Size(max, min)	|被注释的元素的大小必须在指定的范围内|
|@Digits (integer, fraction)	|被注释的元素必须是一个数字，其值必须在可接受的范围内|
|@Past	|被注释的元素必须是一个过去的日期|
|@Future	|被注释的元素必须是一个将来的日期|
|@Pattern(value)	|被注释的元素必须符合指定的正则表达式|


### 2. Hibernate Validator 附加的 constraint

|   注解   |   作用   |
| ---- | ---- |
|@Email	|被注释的元素必须是电子邮箱地址|
|@Length(min=, max=)	|被注释的字符串的大小必须在指定的范围内|
|@NotEmpty	|被注释的字符串的必须非空|
|@Range(min=, max=)	|被注释的元素必须在合适的范围内|
|@NotBlank	|被注释的字符串的必须非空|
|@URL(protocol=, host=,    port=, regexp=, flags=)	|被注释的字符串必须是一个有效的url|
|@CreditCardNumber |被注释的字符串必须通过Luhn校验算法，银行卡，信用卡等号码一般都用Luhn计算合法性|
|@ScriptAssert(lang=, script=, alias=)	|要有Java Scripting API 即JSR 223("Scripting for the JavaTM Platform")的实现|
|@SafeHtml (whitelistType=,additionalTags=)	|classpath中要有jsoup包|

hibernate补充的注解中，最后3个不常用，可忽略。
主要区分下@NotNull，  @NotEmpty，  @NotBlank 3个注解的区别：

|   注解   |   作用   |
| ---- | ---- |
|@NotNull           |任何对象的value不能为null|
|@NotEmpty       |集合对象的元素不为0，即集合不为空，也可以用于字符串不为null|
|@NotBlank        |只能用于字符串不为null，并且字符串trim()以后length要大于0|

##### 其它-@NonNull

@NotNull 是 JSR-303（Bean的校验框架）的注解，用于运行时检查一个属性是否为空，如果为空则不合法。
@NonNull 是JSR-305（缺陷检查框架）的注解，是告诉编译器这个域不可能为空，当代码检查有空值时会给出一个风险警告，目前这个注解只有IDEA支持

## 对controller层参数校验

在controller层的参数校验可以分为两种场景：

1. 单个参数校验
2. 实体类参数校验

### 1.单个参数校验

```java
@RestController
@Validated
public class UserController {

    @GetMapping("/getUser")
    public String getUserStr(@NotNull(message = "name 不能为空") String name,
                             @Max(value = 99, message = "不能大于99岁") Integer age) {
        return "name: " + name + " ,age:" + age;
    }
}
```

当处理`GET`请求时或只传入少量参数的时候，我们可能不会建一个bean来接收这些参数，就可以像上面这样直接在`controller`方法的参数中进行校验。

> 注意：这里一定要在方法所在的controller类上加入`@Validated`注解，不然没有任何效果。

使用这种参数验证的controller方法，会抛出`ConstraintViolationException`异常。

### 2.实体类参数校验

当处理post请求或者请求参数较多的时候我们一般会选择使用一个bean来接收参数，然后在每个需要校验的属性上使用参数校验注解：

```java
@Data
public class UserInfo {

    /**
     * 用户名
     */
    @NotNull
    @Size(min = 2, max = 30,message = "用户名长度应在2到30之间")
    private String userName;

    /**
     * 真实姓名
     */
    @NotNull(message = "realName cannot be null")
    private String realName;

    /**
     * 用户密码
     */
     @NotNull(message = "userPassword cannot be null")
    private String userPassword;
}

```

需要注意的是，如果想让`UserInfo`中的参数注解生效，还必须在Controller参数中使用`@Validated`注解：

```java
@PostMapping("/getUser")
    public String getUserStr(@RequestBody @Validated({GroupA.class, Default.class}) UserInfo user, BindingResult bindingResult){
        validData(bindingResult);
        return "name: " + user.getName() + ", age:" + user.getAge();
    }

    private void validData(BindingResult bindingResult) {
        if (bindingResult.hasErrors()) {
            StringBuffer sb = new StringBuffer();
            for (ObjectError error : bindingResult.getAllErrors()) {
                sb.append(error.getDefaultMessage());
            }
            throw new ValidationException(sb.toString());
        }
    }
```

## 验证多个对象

一个功能方法上处理多个模型对象时，需添加多个验证结果对象:

```java

    @RequestMapping("/addPeople")  
    public @ResponseBody String addPeople(@Validated People p,BindingResult result,@Validated Person p2,BindingResult result2)  
    {  
        if(result.hasErrors())  
        {  
            return "0";  
        }  
        if(result2.hasErrors())  
        {  
            return "-1";  
        }  
        return "1";  
    }  

```

这种参数校验方式的校验结果会被放到`BindingResult`中。如果不加BindingResult，可以通过抛出异常的方式在`GlobalExceptionHandler`中统一处理。



## 全局异常处理

如果有很多使用这种参数验证的controller方法，我们希望在一个地方对参数校验异常进行统一处理，可以使用**统一异常捕获**，这需要借助`@ControllerAdvice`注解来实现，当然在springboot中我们就用`@RestControllerAdvice`（内部包含@ControllerAdvice和@ResponseBody的特性）。

当参数校验异常的时候，该统一异常处理类在控制台打印信息的同时把*bad request*的字符串和`HttpStatus.BAD_REQUEST`所表示的状态码`400`返回给调用方（用`@ResponseBody`注解实现，表示该方法的返回结果直接写入HTTP response body 中）。其中：

- `@ControllerAdvice`：控制器增强，使@ExceptionHandler、@InitBinder、@ModelAttribute注解的方法应用到所有的 @RequestMapping注解的方法。
- `@ExceptionHandler`：异常处理器，此注解的作用是当出现其定义的异常时进行处理的方法。

```java
@Slf4j
@RestControllerAdvice
public class GlobalExceptionHandler {
    /**
     * 返回状态码--系统异常
     */
    public static final int STATUS_ERROR = 999;

    /**
     * 返回状态码--参数错误
     */
    public static final int PARAM_ERROR = 996;

    /**
     * 默认错误提示消息
     */
    public static final String DEFAULT_ERROR_MESSAGE = "系统繁忙，请稍后重试";

    public static final String PARAM_ERROR_MESSAGE = "参数错误！";


    @ExceptionHandler(ConstraintViolationException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public BaseResponseDto<String> resolveConstraintViolationException(ConstraintViolationException ex) {

        Set<ConstraintViolation<?>> constraintViolations = ex.getConstraintViolations();
        String errorMessage = constraintViolations.stream().map(x -> x.getMessage()).collect(Collectors.joining(","));
        return BaseResponseDto.fail(PARAM_ERROR, errorMessage, null);

    }

    /**
     * 方法参数校验
     */
    @ExceptionHandler(value = {BindException.class, ValidationException.class, MethodArgumentNotValidException.class})
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public BaseResponseDto<String> handleParameterVerificationException(Exception e) {
        log.error(" handleParameterVerificationException has been invoked", e);
        String msg = null;
        /// BindException
        if (e instanceof BindException) {
            // getFieldError获取的是第一个不合法的参数(P.S.如果有多个参数不合法的话)
            FieldError fieldError = ((BindException) e).getFieldError();
            if (fieldError != null) {
                msg = fieldError.getDefaultMessage();
            }
            /// MethodArgumentNotValidException
        } else if (e instanceof MethodArgumentNotValidException) {
            BindingResult bindingResult = ((MethodArgumentNotValidException) e).getBindingResult();
            // getFieldError获取的是第一个不合法的参数(P.S.如果有多个参数不合法的话)
            FieldError fieldError = bindingResult.getFieldError();
            if (fieldError != null) {
                msg = fieldError.getDefaultMessage();
            }
            /// ValidationException 的子类异常ConstraintViolationException
        } else if (e instanceof ConstraintViolationException) {
            /*
             * ConstraintViolationException的e.getMessage()形如
             *     {方法名}.{参数名}: {message}
             *  这里只需要取后面的message即可
             */
            msg = e.getMessage();
            if (msg != null) {
                int lastIndex = msg.lastIndexOf(':');
                if (lastIndex >= 0) {
                    msg = msg.substring(lastIndex + 1).trim();
                }
            }
            /// ValidationException 的其它子类异常
        } else {
            msg = "处理参数时异常";
        }
        return BaseResponseDto.fail(PARAM_ERROR, msg, e.getMessage());
    }


    @ExceptionHandler(Exception.class)
    @ResponseBody
    public BaseResponseDto<String> resolveException(Exception ex) {
        log.error("Exception:", ex);
        return BaseResponseDto.fail(STATUS_ERROR, DEFAULT_ERROR_MESSAGE, ex.getMessage());
    }

}
```

### 校验模式

在上面的例子中，我们使用`BindingResult`验证不通过的结果集合，但是通常按顺序验证到第一个字段不符合验证要求时，就可以直接拒绝请求了。这就涉及到两种**校验模式**的配置：

1. 普通模式（默认是这个模式）: 会校验完所有的属性，然后返回所有的验证失败信息
2. 快速失败模式: 只要有一个验证失败，则返回
   如果想要配置第二种模式，需要添加如下配置类：

```java
import org.hibernate.validator.HibernateValidator;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import javax.validation.Validation;
import javax.validation.Validator;
import javax.validation.ValidatorFactory;

@Configuration
public class ValidatorConf {
    @Bean
    public Validator validator() {
        ValidatorFactory validatorFactory = Validation.byProvider( HibernateValidator.class )
                .configure()
                .failFast( true )
                .buildValidatorFactory();
        Validator validator = validatorFactory.getValidator();

        return validator;
    }
}
```

### 参数校验分组

在实际开发中经常会遇到这种情况：想要用一个实体类去接收多个controller的参数，但是不同controller所需要的参数又有些许不同，而你又不想为这点不同去建个新的类接收参数。比如有一个`/setUser`接口不需要`id`参数，而`/getUser`接口又需要该参数，这种时候就可以使用**参数分组**来实现。

1. 定义表示组别的`interface`，在`@Validated`中指定使用哪个组；

```java
public interface GroupA {
}

@RestController
public class UserController {
    @PostMapping("/getUser")
    public String getUserStr(@RequestBody @Validated({GroupA.class, Default.class}) UserInfo user, BindingResult bindingResult){
        validData(bindingResult);
        return "name: " + user.getName() + ", age:" + user.getAge();
    }

    @PostMapping("/setUser")
    public String setUser(@RequestBody @Validated UserInfo user, BindingResult bindingResult) {
        validData(bindingResult);
        return "name: " + user.getName() + ", age:" + user.getAge();
    }
}
```

其中`Default`为`javax.validation.groups`中的类，表示参数类中其他没有分组的参数，如果没有，`/getUser`接口的参数校验就只会有标记了`GroupA`的参数校验生效。

- @Validated没有添加groups属性时，默认验证没有分组的验证属性。
- 不分配groups，默认每次都要进行验证。
- 对一个参数需要多种验证方式时，也可通过分配不同的组达到目的。



```java
@Data
public class UserInfo {
    @NotNull( groups = {GroupA.class}, message = "id cannot be null")
    private Integer id;

    @NotEmpty(groups={First.class})  
	@Size(min=3,max=8,groups={Second.class})  
	private String name;  


    @NotNull(message = "sex cannot be null")
    private String sex;

    @Max(value = 99L)
    private Integer age;
}
```

​	2. 组序列

默认情况下，不同组别的约束验证是无序的，然而在某些情况下，约束验证的顺序却很重要。

例：

（1）第二个组中的约束验证依赖于一个稳定状态来运行，而这个稳定状态是由第一个组来进行验证的。

（2）某个组的验证比较耗时，CPU 和内存的使用率相对比较大，最优的选择是将其放在最后进行验证。因此，在进行组验证的时候尚需提供一种有序的验证方式，这就提出了组序列的概念。
一个组可以定义为其他组的序列，使用它进行验证的时候必须符合该序列规定的顺序。在使用组序列验证的时候，如果序列前边的组验证失败，则后面的组将不再给予验证。

分组接口类 （通过@GroupSequence注解对组进行排序）：

```java
public interface First {  

}  

public interface Second {  

}  

import javax.validation.GroupSequence;  
@GroupSequence({First.class,Second.class})  
public interface Group {  

}

public class People {  
	//在First分组时，判断不能为空  
	@NotEmpty(groups={First.class})  
	private String id;  
  
	//name字段不为空，且长度在3-8之间  
	@NotEmpty(groups={First.class})  
	@Size(min=3,max=8,groups={Second.class})  
	private String name;  
}


@Controller  
public class FirstController {  
      
    @RequestMapping("/addPeople")  
    //不需验证ID  
    public @ResponseBody String addPeople(@Validated({Group.class}) People p,BindingResult result)  
    {  
        if(result.hasErrors())  
        {  
            return "0";  
        }  
        return "1";  
    }  
} 
```

### 级联参数校验

当参数bean中的属性又是一个复杂数据类型或者是一个集合的时候，如果需要对其进行进一步的校验需要考虑哪些情况呢？

```java
@Data
public class UserInfo {
    @NotNull( groups = {GroupA.class}, message = "id cannot be null")
    private Integer id;

    @NotNull(message = "username cannot be null")
    private String name;

    @NotNull(message = "sex cannot be null")
    private String sex;

    @Max(value = 99L)
    private Integer age;
   
    @NotEmpty
    private List<Parent> parents;
}
```

比如对于`parents`参数，`@NotEmpty`只能保证list不为空，但是list中的元素是否为空、User对象中的属性是否合格，还需要进一步的校验。这个时候我们可以这样写:

```java
    @NotEmpty
    private List<@NotNull @Valid UserInfo> parents;
```

然后再继续在`UserInfo`类中使用注解对每个参数进行校验。

但是我们再回过头来看看，在controller中对实体类进行校验的时候使用的`@Validated`，在这里只能使用`@Valid`否则会报错。但是在这里我想说的是使用`@Valid`就没办法对`UserInfo`进行分组校验。

### 自定义参数校验

虽然JSR303和Hibernate Validator已经提供了很多校验注解，但是当面对复杂参数校验时，还是不能满足我们的要求，这时候我们就需要自定义校验注解。这里我们再回到上面的例子介绍一下自定义参数校验的步骤。`private List<@NotNull @Valid UserInfo> parents`这种在容器中进行参数校验是`Bean Validation2.0`的新特性，假如没有这个特性，我们来试着自定义一个**List数组中不能含有null元素**的注解。这个过程大概可以分为两步：

1. 自定义一个用于参数校验的注解，并为该注解指定校验规则的实现类
2. 实现校验规则的实现类

#### 自定义注解

定义`@ListNotHasNull`注解， 用于校验 List 集合中是否有null 元素

```java
@Target({ElementType.ANNOTATION_TYPE, ElementType.METHOD, ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
//此处指定了注解的实现类为ListNotHasNullValidatorImpl
@Constraint(validatedBy = ListNotHasNullValidatorImpl.class)
public @interface ListNotHasNull {

    /**
     * 添加value属性，可以作为校验时的条件,若不需要，可去掉此处定义
     */
    int value() default 0;

    String message() default "List集合中不能含有null元素";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};

    /**
     * 定义List，为了让Bean的一个属性上可以添加多套规则
     */
    @Target({METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER})
    @Retention(RUNTIME)
    @Documented
    @interface List {
        ListNotHasNull[] value();
    }
}
```

> 注意：message、groups、payload属性都需要定义在参数校验注解中不能缺省

#### 注解实现类

该类需要实现`ConstraintValidator`

```java
import org.springframework.stereotype.Service;
import javax.validation.ConstraintValidator;
import javax.validation.ConstraintValidatorContext;
import java.util.List;

public class ListNotHasNullValidatorImpl implements ConstraintValidator<ListNotHasNull, List> {

    private int value;

    @Override
    public void initialize(ListNotHasNull constraintAnnotation) {
        //传入value 值，可以在校验中使用
        this.value = constraintAnnotation.value();
    }

    public boolean isValid(List list, ConstraintValidatorContext constraintValidatorContext) {
        for (Object object : list) {
            if (object == null) {
                //如果List集合中含有Null元素，校验失败
                return false;
            }
        }
        return true;
    }
}
```

然后我们就能在之前的例子中使用该注解了：

```java
@NotEmpty
@ListNotHasNull
private List<@Valid UserInfo> userInfoList;
```

