# gralde exclude

在引用依赖时经常会有这样的问题：某些间接引用的依赖项是不需要的；产生了依赖冲突。此时需要排除一些依赖。

下面的内容介绍了几种在gradle中排除依赖的方式。

## 在dependency中排除

```gradle

dependencies {

    compile('com.zhyea:ar4j:1.0') {
        //excluding a particular transitive dependency:
        exclude module: 'cglib' //by artifact name
        exclude group: 'org.jmock' //by group
        exclude group: 'org.unwanted', module: 'iAmBuggy' //by both name and group
    }
}

```

## 在全局配置中排除

```gradle
configurations {
    compile.exclude module: 'cglib'
    all*.exclude group:'org.unwanted', module: 'iAmBuggy' 
}

```

禁用传递依赖

禁用传递依赖需要将属性transitive设置为false。可以在dependency中禁用传递依赖，也可以在configuration中全局禁用

```gradle
compile('com.zhyea:ar4j:1.0') {
    transitive = false
}

configurations.all {
    transitive = false
}
```

还可以在单个依赖项中使用@jar标识符忽略传递依赖：

compile 'com.zhyea:ar4j:1.0@jar'

强制使用某个版本

如果某个依赖项是必需的，而又存在依赖冲突时，此时没必要逐个进行排除，可以使用force属性标识需要进行依赖统一。当然这也是可以全局配置的：

```gradle
compile('com.zhyea:ar4j:1.0') {
    force = true
}


configurations.all {
    resolutionStrategy {
    force 'org.hamcrest:hamcrest-core:1.3'
    }
}
```

在打包时排除依赖

先看一个示例：

task zip(type: Zip) {

    into('lib') {

        from(configurations.runtime) {

            exclude '*unwanted*', '*log*'

        }

    }

    into('') {

        from jar

        from 'doc'

    }

}

代码表示在打zip包的时候会过滤掉名称中包含“unwanted”和“log”的jar包。这里调用的exclude方法的参数和前面的例子不太一样，前面的参数多是map结构，这里则是一个正则表达式字符串。

也可以使用在打包时调用include方法选择只打包某些需要的依赖项：

task zip(type: Zip) {

    into('lib') {

        from(configurations.runtime) {

            include '*ar4j*', '*spring*'

        }

    }

    into('') {

        from jar

        from 'doc'

    }

}

始终选择最新的依赖的项

最后这个特性和排除依赖没有什么关系，只是顺带说一下。

在依赖项中使用加号+,可以在构建时检查远程仓库是否存在该依赖的新版本，如果存在新版本则下载选用最新版本。更有用的是可以指定依赖某个大版本下的最新子版本，如1.+表示始终采用依赖的1.x序列的最新版本。

示例如下：

compile 'com.zhyea:ar4j:+'
