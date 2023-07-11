---

title:  "springboot-test和dbunit单元测试方案"
date:   2021-05-10 20:29:36 +0800
categories: springboot
typora-root-url: ..
---

## springboot-test和dbunit单元测试方案

参考链接：[Testing (spring.io)](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#testing)

在pom文件中添加：maven依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

## 一般的测试方法

简单的，对于springboot项目，只需要引入spring-boot-starter-test，就可以使用@SpringBootTest注解编写测试类。

```java
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
public class BaseTest {

 /**
     * 注入发送邮件的接口
     */
    @Autowired
    private MailService mailService;

    /**
     * 测试发送文本邮件
     */
    @Test
    public void sendmail() {
        mailService.sendSimpleMail("@qq.com", "主题：你好文本邮件", "内容：发送文本邮件");
    }

    @Test
    public void sendmailHtml() {
        mailService.sendHtmlMail("@qq.com", "主题：你好html网页邮件", "<h2>内容：第一封html网页邮件</h2>");
    }
}
```

## MockMvc测试Rest接口

除了注入service,mapper来测试方法springboot-test也提供了MockMvc来测试http请求。
MockMvc是由springboot-test包提供，实现了对Http请求的模拟，能够直接使用网络的形式，转换到Controller的调用，使得测试速度快、不依赖网络环境。同时提供了一套验证的工具，结果的验证十分方便。

接口MockMvcBuilder，提供一个唯一的build方法，用来构造MockMvc。主要有两个实现：StandaloneMockMvcBuilder和DefaultMockMvcBuilder，分别对应两种测试方式，即独立安装和集成Web环境测试（并不会集成真正的web环境，而是通过相应的Mock API进行模拟测试，无须启动服务器）。MockMvcBuilders提供了对应的创建方法standaloneSetup方法和webAppContextSetup方法，在使用时直接调用即可。


```java
@Slf4j
public class CacheCtrlTest extends BaseTest {
    @Autowired
    private WebApplicationContext webApplicationContext;
    private MockMvc mockMvc;

    @BeforeEach
    public void setUp() {
        mockMvc = MockMvcBuilders.webAppContextSetup(webApplicationContext).build();//建议使用这种
    }


    /**
     * 1、mockMvc.perform执行一个请求。
     * 2、MockMvcRequestBuilders.get("XXX")构造一个请求。
     * 3、ResultActions.param添加请求传值
     * 4、ResultActions.accept(MediaType.TEXT_HTML_VALUE))设置返回类型
     * 5、ResultActions.andExpect添加执行完成后的断言。
     * 6、ResultActions.andDo添加一个结果处理器，表示要对结果做点什么事情
     * 比如此处使用MockMvcResultHandlers.print()输出整个响应结果信息。
     * 5、ResultActions.andReturn表示执行完成后返回相应的结果。
     */
    @Test
    public void cacheTest() {
        try {
            CacheService.KeyValue keyValue = new CacheService.KeyValue("test_cache", "测试缓存");
            String requestJson = JsonUtils.toJSONString(keyValue);
            //对post测试

            ResultActions resultActions = mockMvc.perform(
                    MockMvcRequestBuilders.post("/test/testCache")
                            .contentType(MediaType.APPLICATION_JSON)
                            .characterEncoding("UTF-8")
                            .content(requestJson)
            ).andExpect(MockMvcResultMatchers.status().isOk());
            resultActions.andReturn().getResponse().setCharacterEncoding("UTF-8");
            resultActions.andDo(MockMvcResultHandlers.print());
            //断言，判断返回的值是否正确
            String content = resultActions.andReturn().getResponse().getContentAsString();
            Assertions.assertEquals("{\"code\":200,\"status\":\"success\",\"message\":null,\"moreInfo\":null,\"data\":\"OK\",\"success\":true}",
                    content);

            //对get测试
            resultActions = mockMvc.perform(
                    MockMvcRequestBuilders.get("/test/testCache")
                            .contentType(MediaType.APPLICATION_JSON)
                            .characterEncoding("UTF-8")
                            .param("key", keyValue.getKey())
            ).andExpect(MockMvcResultMatchers.status().isOk());
            //设置编码
            resultActions.andReturn().getResponse().setCharacterEncoding("UTF-8");
            resultActions.andDo(MockMvcResultHandlers.print());
            content = resultActions.andReturn().getResponse().getContentAsString();
            Assertions.assertEquals("{\"code\":200,\"status\":\"success\",\"message\":null,\"moreInfo\":null,\"data\":\"测试缓存\",\"success\":true}",
                    content);

            CacheService.KeyValue keyValue2 = new CacheService.KeyValue("test_cache", "更新缓存");
            String requestJson2 = JsonUtils.toJSONString(keyValue2);
            //对put测试
            MvcResult mvcResultPut = mockMvc.perform(
                    MockMvcRequestBuilders.put("/test/testCache")
                            .contentType(MediaType.APPLICATION_JSON)
                            .characterEncoding("UTF-8")
                            .content(requestJson2)
            ).andExpect(MockMvcResultMatchers.status().isOk())
                    .andDo(MockMvcResultHandlers.print())
                    .andReturn();
            content = mvcResultPut.getResponse().getContentAsString();
            Assertions.assertEquals("{\"code\":200,\"status\":\"success\",\"message\":null,\"moreInfo\":null,\"data\":\"OK\",\"success\":true}",
                    content);
            //对delete测试
            MvcResult mvcResultDelete = mockMvc.perform(
                    MockMvcRequestBuilders.delete("/test/testCache")
                            .contentType(MediaType.APPLICATION_JSON)
                            .characterEncoding("UTF-8")
                            .param("key", keyValue.getKey())
            ).andExpect(MockMvcResultMatchers.status().isOk())
                    .andDo(MockMvcResultHandlers.print())
                    .andReturn();
            content = mvcResultDelete.getResponse().getContentAsString();
            Assertions.assertEquals("{\"code\":200,\"status\":\"success\",\"message\":null,\"moreInfo\":null,\"data\":\"OK\",\"success\":true}",
                    content);

        } catch (Exception e) {
            log.error("缓存测试失败 e", e);
            Assertions.fail("缓存测试失败", e);
        }
    }
}

```



这种测试程序运行时，依赖数据库数据。容易受到历史数据，脏数据影响。

  ```mermaid
  graph LR;
  	构造调用-->|输入参数|pg([待测试程序])-->|输出结果|观察比较;
  	db[(数据库)]-->|获取数据|pg([待测试程序]);
  	
  	
  ```

## 基于dbunit+h2内存数据库的测试

dbunit是junit的一个扩展，对于数据库驱动的项目，它能在进行单元测试之前，使得数据库维持一个给定的状态。

H2是一个开源的嵌入式数据库引擎，它是一个用Java开发的类库，可直接嵌入到应用程序中，与应用程序一起打包发布，不受平台限制。

单元测试使用内存数据库时：测试类启动时会创建内存数据库，并在停止时销毁。

---

官网[About DbUnit](http://www.dbunit.org/)

H2数据库地址：[H2 Database Engine](http://www.h2database.com/html/main.html)

---

  ```mermaid
  graph LR;
  	编写测试类--->|构造输入参数|pg([待测试程序])-->|输出结果|比较Json;
  	准备xml格式数据-->|初始化|h2[(内存数据库)]-->|获取数据|pg([待测试程序]);
  ```

### maven依赖

值得注意的是，unitils的高版本目前（2021-06-18）还未支持h2内存数据库，不要引入高版本。

```xml
<dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <version>1.4.200</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <scope>test</scope>
            <groupId>org.unitils</groupId>
            <artifactId>unitils-dbunit</artifactId>
            <version>3.4.3</version>
        </dependency>
        <dependency>
            <scope>test</scope>
            <groupId>org.unitils</groupId>
            <artifactId>unitils-io</artifactId>
            <version>3.4.3</version>
        </dependency>
        <dependency>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <artifactId>commons-logging</artifactId>
                    <groupId>commons-logging</groupId>
                </exclusion>
            </exclusions>
            <groupId>org.unitils</groupId>
            <artifactId>unitils-database</artifactId>
            <version>3.4.3</version>
        </dependency>
        <dependency>
            <scope>test</scope>
            <groupId>org.unitils</groupId>
            <artifactId>unitils-spring</artifactId>
            <version>3.4.3</version>
        </dependency>
```



### unitls.properties

在src/test/resources/下添加unitils.properties文件。

```properties
#启用的unitils 模块
unitils.modules=database,dbunit,hibernate,spring,
#配置扩展模块
#unitils.module.dbunit.className=com.zph.programmer.springboot.utils.MySqlDbUnitModule
#配置数据库连接
database.driverClassName=org.h2.Driver
database.url=jdbc:h2:mem:test_mem:public
database.dialect=mysql
database.userName=sa
database.password=
database.schemaNames=public
# The database maintainer is disabled by default.
#数据库维护策略  在每次运行时可更新数据库 根据dbMaintainer.script.locations设置的sql文件进行更新
#当以往文件改变 将更新此文件到数据库  未改变的sql文件将不变
#命名格式   <index>_<some name>.sql
updateDataBaseSchema.enabled=true
#This table is by default not created automatically
#数据库表生成策略
dbMaintainer.autoCreateExecutedScriptsTable=true
dbMaintainer.keepRetryingAfterError.enabled=false
dbMaintainer.script.locations=./src/test/resources/sqlscript
#配置数据集工厂
DbUnitModule.DataSet.factory.default=org.unitils.dbunit.datasetfactory.impl.MultiSchemaXmlDataSetFactory
DbUnitModule.ExpectedDataSet.factory.default=org.unitils.dbunit.datasetfactory.impl.MultiSchemaXmlDataSetFactory
#配置数据库加载策略
DbUnitModule.DataSet.loadStrategy.default=org.unitils.dbunit.datasetloadstrategy.impl.CleanInsertLoadStrategy
DatabaseModule.Transactional.value.default=rollback
# XSD generator
#配置数据集结构模式XSD生成路径
dataSetStructureGenerator.xsd.dirName=resources/xsd
```



### 测试基类和配置方式

添加TestConf.java和TestApplicaiton.java

```java
import org.springframework.boot.test.context.TestConfiguration;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.PropertySource;
import org.unitils.database.UnitilsDataSourceFactoryBean;

import javax.sql.DataSource;

@PropertySource("classpath:unitils.properties")
@TestConfiguration
public class TestConf {
    //使用unitils配置的数据库
    DataSource dataSource = (DataSource) new UnitilsDataSourceFactoryBean().getObject();

    public TestConf() throws Exception {
    }

    @Bean(name = "dataSource")
    public DataSource dbunitDataSource() throws Exception {
        return dataSource;
    }
}
```

如果项目使用Configuration配置的数据库，需要避免将原数据库bean实例化。这里没有使用是因为配置SQLlite数据库是，只是配置了properties，借用springboot的约定，没有显示指定dataesource时，springboot会默认读取配置创建。上面TestConf 在测试配置中指定了dbunits配置的数据库。

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.FilterType;
import org.springframework.core.type.classreading.MetadataReader;
import org.springframework.core.type.classreading.MetadataReaderFactory;
import org.springframework.core.type.filter.TypeFilter;

import java.io.IOException;

@Slf4j
@ComponentScan(basePackages = "com.zph.programmer.springboot"
        , excludeFilters = {
        @ComponentScan.Filter(type = FilterType.CUSTOM, classes = TestApplication.TypeExFilter.class)
})
public class TestApplication {
    /**
     * 避免以下的Bean被实例化
     */
    public static class TypeExFilter implements TypeFilter {
        @Override
        public boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory) throws IOException {
            String n = metadataReader.getClassMetadata().getClassName();
           /* if (n.equals(DataSourceConf.class.getName())) {
                return true;
            }
            */
            return false;

        }
    }
}
```



TestRunner.java

```java
import org.junit.internal.runners.InitializationError;
import org.junit.runner.notification.RunNotifier;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.unitils.UnitilsJUnit4TestClassRunner;


public class TestRunner extends UnitilsJUnit4TestClassRunner {

    private final SpringRunner springRunner;

    public TestRunner(Class<?> clazz) throws org.junit.runners.model.InitializationError, InitializationError {
        super(clazz);
        springRunner = new SpringRunner(clazz);
    }

    @Override
    public void run(RunNotifier notifier) {
        super.run(notifier);
    }

    @Override
    protected Object createTest() throws Exception {
        //使用SpringJUnit4ClassRunner.createTest(), 兼容@Bean @Autowired
        return springRunner.createTest();
    }

    public static class SpringRunner extends SpringJUnit4ClassRunner {
        public SpringRunner(Class<?> clazz) throws org.junit.runners.model.InitializationError {
            super(clazz);
        }

        @Override
        public Object createTest() throws Exception {
            return super.createTest();
        }
    }
}
```

基类BaseUnitilsTest.java

```java

@RunWith(TestRunner.class)
@SpringBootTest(classes = {TestConf.class, TestApplication.class},
        webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@ActiveProfiles("utfake")
@Slf4j
public class BaseUnitilsTest {
    @Before
    public void before() {
        MockitoAnnotations.initMocks(this);
    }

    @After
    public void after() {
        ReflectTestUtils.revert();
    }

    @Ignore
    @Test
    public void t() {

    }
}
```

## 使用方式

xml文件准备REST_CALL_LOG_RECORD表数据:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<dataset>
    <REST_CALL_LOG_RECORD id="13" method="GET" uri="http://127.0.0.1:8180/v2/api-docs" request=""
                          response="" status="200" cost_time="385" is_valid="1" created_time="2021-06-14 16:44:03"
                          modified_time="2021-06-14 16:44:03"/>
</dataset>
```

@DataSet：测试之前初始化数据库

@ExpectedDataset:测试之后比对数据库数据

另外：JsonComparator是第三方工具类，读取json文件比较结果。

```java
@Slf4j
public class TestServiceTest extends BaseUnitilsTest {
    @Resource
    private TestService testService;
    /**
     * dbunit need junit4 test
     */
    @Test
    @DataSet(value = {"rest_call_log_record.init.xml"})
    @ExpectedDataSet(value = {"rest_call_log_record.init.xml"})
    public void findRestLogById() {

        RestCallLogRecord record = testService.findRestLogById(13);
        JsonComparator.newInstance("findRestLogById.expected.json").compareAssert(record);
    }
}
```

