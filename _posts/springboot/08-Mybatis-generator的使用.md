---
layout: post
title:  "Mybatis-generator的使用"
date:   2021-05-28 14:29:36 +0800
categories: springboot
---

#  Mybatis-generator的使用
以下所有内容均可在[springboot-demo](https://github.com/zph-programmer/springboot)中找到。

## 所需依赖
在maven pom文件中引入所需依赖
```xml    
<dependency>
    <groupId>org.mybatis.generator</groupId>
    <artifactId>mybatis-generator-core</artifactId>
    <version>1.4.0</version>
</dependency>
```

## 配置generator.xml文件
这里面classPathEntry是驱动程序位置，上一篇文章中添加的sqlite-jdbc应该已经通过maven下好，可以在.m2文件里面搜到，已经复制出来放在项目中。此外，plugin type另外指定了一个自定义插件MybatisGeneratorPlugin。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
    <!-- 驱动程序路径 -->
    <classPathEntry location="lib/sqlite-jdbc-3.34.0.jar" />

    <context id="SQLiteTables" targetRuntime="MyBatis3Simple">

        <plugin type="com.zph.programmer.springboot.utils.MybatisGeneratorPlugin"/>
        <!-- 注释 -->
        <commentGenerator>
            <property name="suppressAllComments" value="true"/><!-- 是否取消注释 -->
            <property name="suppressDate" value="false"/> <!-- 是否生成注释代时间戳-->
        </commentGenerator>
        <!-- 数据库连接地址 -->
        <jdbcConnection driverClass="org.sqlite.JDBC"
                        connectionURL="jdbc:sqlite:springboot-web/src/main/resources/static/sqlite/sqlite.db"
                        userId=""
                        password="">
            <property name="useInformationSchema" value="true" />
        </jdbcConnection>
        <javaTypeResolver >
            <property name="useJSR310Types" value="true"/>
            <property name="forceBigDecimals" value="false" />
        </javaTypeResolver>
<!-- 实体类 -->
        <javaModelGenerator targetPackage="com.zph.programmer.springboot.po" targetProject="springboot-web/src/main/java/">
            <property name="enableSubPackages" value="true" />
            <property name="trimStrings" value="true" />
        </javaModelGenerator>
       <!-- xml文件 --> 
        <sqlMapGenerator targetPackage="mapper"  targetProject="springboot-web/src/main/resources/static">
            <property name="enableSubPackages" value="true" />
        </sqlMapGenerator>
 <!-- mapper文件 -->
        <javaClientGenerator type="XMLMAPPER" targetPackage="com.zph.programmer.springboot.dao"
                             targetProject="springboot-web/src/main/java/">
            <property name="enableSubPackages" value="true" />
        </javaClientGenerator>

        <table tableName="rest_call_log_record" domainObjectName="RestCallLogRecord"
               enableDeleteByPrimaryKey="false"
               enableUpdateByPrimaryKey="true"
               enableSelectByPrimaryKey="true"
               enableInsert="true">
            <columnOverride column="cost_time" javaType="Long"/>
        </table>

    </context>
</generatorConfiguration>
```

## 使用generator自定义插件
因为有一些定制化功能改造，这里使用MybatisGeneratorPlugin自定义插件。这个自定义插件主要结合了lombok,去掉了自带的getter，setter等方法。

```java
import org.mybatis.generator.api.GeneratedXmlFile;
import org.mybatis.generator.api.IntrospectedColumn;
import org.mybatis.generator.api.IntrospectedTable;
import org.mybatis.generator.api.PluginAdapter;
import org.mybatis.generator.api.dom.java.Field;
import org.mybatis.generator.api.dom.java.Interface;
import org.mybatis.generator.api.dom.java.Method;
import org.mybatis.generator.api.dom.java.TopLevelClass;
import org.mybatis.generator.api.dom.xml.XmlElement;
import java.time.LocalDate;
import java.util.List;

public class MybatisGeneratorPlugin extends PluginAdapter {
    @Override
    public boolean validate(List<String> list) {
        return true;
    }

    @Override
    public boolean modelBaseRecordClassGenerated(TopLevelClass topLevelClass, IntrospectedTable introspectedTable) {
        //添加domain的import
        topLevelClass.addImportedType("lombok.Data");
        //添加domain的注解
        topLevelClass.addAnnotation("@Data");

        //添加Accessors的import
        topLevelClass.addImportedType("lombok.experimental.Accessors");
        //添加的注解
        topLevelClass.addAnnotation("@Accessors(chain = true)");
        // 获取表注释
        String remarks = introspectedTable.getRemarks();
        //添加domain的注释
        topLevelClass.addJavaDocLine("/**");
        topLevelClass.addJavaDocLine(" * " + remarks);
        topLevelClass.addJavaDocLine("* Created by Mybatis Generator on " + LocalDate.now());
        topLevelClass.addJavaDocLine("*/");

        return true;
    }

    //@Override
    public boolean clientGenerated(Interface interfaze, TopLevelClass topLevelClass, IntrospectedTable introspectedTable) {
        //Mapper文件的注释
        interfaze.addJavaDocLine("/**");
        interfaze.addJavaDocLine("* Created by Mybatis Generator on " +  LocalDate.now());
        interfaze.addJavaDocLine("*/");
        return true;
    }

    @Override
    public boolean modelFieldGenerated(Field field, TopLevelClass topLevelClass, IntrospectedColumn introspectedColumn, IntrospectedTable introspectedTable, ModelClassType modelClassType) {
        // 获取列注释
        String remarks = introspectedColumn.getRemarks();
        field.addJavaDocLine("/**");
        field.addJavaDocLine(" * " + remarks);
        field.addJavaDocLine(" */");
        return true;
    }
    @Override
    public boolean modelSetterMethodGenerated(Method method, TopLevelClass topLevelClass, IntrospectedColumn introspectedColumn, IntrospectedTable introspectedTable, ModelClassType modelClassType) {
        //不生成getter
        return false;
    }

    @Override
    public boolean modelGetterMethodGenerated(Method method, TopLevelClass topLevelClass, IntrospectedColumn introspectedColumn, IntrospectedTable introspectedTable, ModelClassType modelClassType) {
        //不生成setter
        return false;
    }

    @Override
    public boolean sqlMapGenerated(GeneratedXmlFile sqlMap, IntrospectedTable introspectedTable) {
        sqlMap.setMergeable(false);//覆盖xml
        return super.sqlMapGenerated(sqlMap, introspectedTable);
    }
    @Override
    public boolean clientSelectAllMethodGenerated(Method method, Interface interfaze, IntrospectedTable introspectedTable) {
        return false;//去掉selectAll
    }
    @Override
    public boolean sqlMapSelectAllElementGenerated(XmlElement var1, IntrospectedTable var2){
        return false;//去掉selectAll
    }
}

```

### 执行mybatis自定义插件
生成了RestCallLogRecord.java，RestCallLogRecordMapper.java,MapperRestCallLogRecordMapper.xml文件
```java
import org.mybatis.generator.api.MyBatisGenerator;
import org.mybatis.generator.config.Configuration;
import org.mybatis.generator.config.xml.ConfigurationParser;
import org.mybatis.generator.internal.DefaultShellCallback;

import java.io.InputStream;
import java.util.ArrayList;
import java.util.List;

public class MyBatisGeneratorRun {
    public static void main(String[] args) throws Exception{
        MyBatisGeneratorRun app = new MyBatisGeneratorRun();

        System.out.println(app.getClass().getResource("/").getPath());
        String generatorFilePath="static/generatorConfig.xml";
        app.generator(generatorFilePath);
    }

    public void generator(String generatorFilePath) throws Exception{

        List<String> warnings = new ArrayList<String>();
        boolean overwrite = true;
        InputStream resourceAsStream = this.getClass().getClassLoader().getResourceAsStream(generatorFilePath);
        ConfigurationParser cp = new ConfigurationParser(warnings);
        Configuration config = cp.parseConfiguration(resourceAsStream);
        DefaultShellCallback callback = new DefaultShellCallback(overwrite);
        MyBatisGenerator myBatisGenerator = new MyBatisGenerator(config, callback, warnings);
        myBatisGenerator.generate(null);

        for(String warning:warnings){
            System.out.println(warning);
        }
    }
}
```
## 其它参考
[mybatis-generator官网配置介绍文档](http://mybatis.org/generator/configreference/xmlconfig.html)
### 领域模型中的实体类分为四种类型：VO、DTO、DO、PO
```
PO：
persistant object持久对象

最形象的理解就是一个PO就是数据库中的一条记录。
好处是可以把一条记录作为一个对象处理，可以方便的转为其它对象。

BO：
business object业务对象
主要作用是把业务逻辑封装为一个对象。这个对象可以包括一个或多个其它的对象。
比如一个简历，有教育经历、工作经历、社会关系等等。
我们可以把教育经历对应一个PO，工作经历对应一个PO，社会关系对应一个PO。
建立一个对应简历的BO对象处理简历，每个BO包含这些PO。
这样处理业务逻辑时，我们就可以针对BO去处理。

VO ：
value object值对象
ViewObject表现层对象

主要对应界面显示的数据对象。对于一个WEB页面，或者SWT、SWING的一个界面，用一个VO对象对应整个界面的值。(包含界面所有值的对象)

DTO ：
Data Transfer Object数据传输对象
主要用于远程调用等需要大量传输对象的地方。
比如我们一张表有100个字段，那么对应的PO就有100个属性。
但是我们界面上只要显示10个字段，
客户端用WEB service来获取数据，没有必要把整个PO对象传递到客户端，
这时我们就可以用只有这10个属性的DTO来传递结果到客户端，这样也不会暴露服务端表结构.到达客户端以后，如果用这个对象来对应界面显示，那此时它的身份就转为VO

POJO ：
plain ordinary java object 简单java对象
个人感觉POJO是最常见最多变的对象，是一个中间对象，也是我们最常打交道的对象。

一个POJO持久化以后就是PO
直接用它传递、传递过程中就是DTO
直接用来对应表示层就是VO

DAO：
data access object数据访问对象
这个大家最熟悉，和上面几个O区别最大，基本没有互相转化的可能性和必要.

主要用来封装对数据库的访问。通过它可以把POJO持久化为PO，用PO组装出来VO、DTO
```

* [返回目录](https://zph-programmer.github.io)
    * [上一篇 —— SQLite数据库](07-SQLite数据库.md)
    * [下一篇 —— Springboot使用mybatis连接SQLite](09-Springboot使用mybatis连接SQLite.md)