# 前置
机器环境:centos

# 下载jdk

jdk文件太大，网络下载，链接可能失效

wget https://javadl.oracle.com/webapps/download/GetFile/1.8.0_261-b12/a4634525489241b9a9e1aa73d9e118e6/linux-i586/jdk-8u261-linux-x64.tar.gz -O jdk-8u261-linux-x64.tar.gz

以下下用的jdk-8u251

# 安装jdk
- 卸载自带的openjdk
```
yum -y remove java-1.8.0-openjdk.x86_64
```

- 上传jdk至usr目录，并解压
```
cd /usr
tar -xvf jdk-8u251-linux-x64.tar.gz

```

- 配置环境变量：
```
vim /etc/profile
添加
export JAVA_HOME=/usr/jdk1.8.0_251
export CLASSPATH=$:CLASSPATH:$JAVA_HOME/lib/
export PATH=$PATH:$JAVA_HOME/bin
保存
刷新环境变量文件： 
source /etc/profile
```

- 将默认java命令链接到安装的jdk
```
alternatives --install /usr/bin/java java /usr/jdk1.8.0_251/bin/java 2
alternatives --config java
```

- 提示有两个安装的jdk可供选择，选择刚安装的2

There are 2 programs which provide 'java'.

  Selection    Command
-----------------------------------------------
*+ 1           java-1.8.0-openjdk.x86_64 (/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.181-7.b13.el7.x86_64/jre/bin/java)
   2           /usr/jdk1.8.0_251/bin/java

Enter to keep the current selection[+], or type selection number:


# 安装tomcat
- 根目录创建app文件夹，rz上传并解压

```
tar -xvf apache-tomcat-8.5.56.tar.gz 
cd tomcat
cd bin
./startup.sh
ps -ef|grep tomcat
```

# 启动应用
- 上传war包至/app/tomcat/webapps目录下，启动tomcat

# Tomcat访问路径去掉发布项目的项目名称

实现方式及原理： 

方式一： 
原理：Tomcat的默认根目录是ROOT，实际上ROOT这个项目在实际生产环境是没有用的，所以我们可以用我们的项目覆盖ROOT项目 
操作过程： 
1.删除ROOT下所有文件及文件夹 
2.把我们项目的war包解压后，项目目录下的所有文件和子目录都拷贝到ROOT目录下即可 
或者有更狠的一招：直接删掉ROOT目录，然后把我们的项目打包名称改成ROOT.war，放到webapps下就行 

方式二： 
原理：Tomcat本身可以配置虚拟目录。方法就是在Server.xml中<Engine><Host>节点下加入Context信息。如我们可以配置<Context path="/abc" docBase="D:\app\abc" ... />，那我们可以通过地址http://localhost:8080/abc来访问我们放在D:\app\下面的abc项目。我们可以把这个path="/abc"修改为path=""。意思就是把abc映射到根目录，访问路径就会变成http://localhost:8080/。 
操作过程： 
按照配置虚拟目录的方式，在<Engine><Host>下添加一个Context节点，具体配置如下： 
Xml代码  收藏代码

    <Engine name="Catalina" defaultHost="localhost"...>  
    ...  
        <Host name="localhost"  appBase="webapps" unpackWARs="true" autoDeploy="true">  
        <Context path="" docBase="Interface" reloadable="true" />  
    <!--注：我这里使用的是相对路径，Interface项目是放在Tomcat的webapps目录下的，当然也可以改为绝对路径-->  
    ...  
        </Host>  
    ...  
    </Engine>  


访问方式就可以用http://localhost:8080/SearchReqService.asmx?wsdl了 
如果用虚拟目录的方式，地址http://localhost:8080/Interface/SearchReqService.asmx?wsdl也可以访问。 
同样的方式，我们可以为path指定不同的路径，解决访问路径区别项目名称的需要。 


其它，去掉访问的端口号8080.就是利用了HTTP请求访问的端口默认是80的方式实现的，iis也一样。我们只用把Tomcat的HTTP监听端口号改为80（修改<Connector port="8080" protocol="HTTP/1.1"这里的端口号为80）即可。 

# 多tomcat启动
- 多tomcat启动,修改端口

[https://blog.csdn.net/qq_42685333/article/details/87873983]


# war包能不能删除
    war不能在tomcat运行时删除，否则会删除自动解压的工程。 你可以停止tomcat后删除war。
    当你重新部署的时候，如果有与war文件相同的文件夹，就不会重新部署。
    
        因为tomcat在运行期, 会实时监控webapps目录下的war文件，如果有新增的war，就去解压它; 有删除war，就连同项目一起删除 .
        所以，如果您要删除，可以先关闭tomcat再删除，这样不会有影响的

 


    Tomcat/webapps下的WAR包和同名已解压项目，如何加载？

1、首先你要明白什么时候war包才会解压
2、当tomcat启动时候，会去查看webapps目錄下的所有war包，同时查看是否有该war包对应的，已解压的同名文件夹，
3、如果已经存在，就不会再解压，也不會覆蓋該工程下已经被修改过的文件.
4、只有当你删除war包对应的同名文件夹（即 你的工程 ）后，启动tomcat时才会再進行解压war文件动作





# Hot Swap failed:add method not implemented

This new feature encapsulates the ability to substitute modified code in a running application through the debugger APIs. 
——'HotSwapping' using JVM:http://www.jug.mk/blogs/ipenov/entry/hotswapping_using_jvm

**目前HotSwap只支持对方法body的修改，不支持对类和方法签名的修改（比如修改类名，方法名，方法参数，添加或者删除一个方法，增加、删除类文件等，是不能够热部署到服务上的。这时候需要停止服务器重新部署后再启动，就不会出现上面的提示了等**）。考虑这些限制，也是有理由的，替换类定义，就需要新类和旧类之间有一个关联，这里关联就是类的全名（或许还有其他信息），类名都改了，就不知道替换哪个类了。至于方法签名的修改，应该是考虑到运行时方法的调用，通过方法签名替换已有的方法调用。 

网上很多人误解热部署和热加载的概念,所以造成乱配置的行为,这里提示一下.
热部署:就是容器状态在运行的情况下部署或者重新部署整个项目.在这种情况下一般整个内存会清空,重新加载.简单来说就是Tomcat或者其他的web服务器会帮我们重新加载项目.这种方式可能会造成sessin丢失等情况.
热加载:就是容器状态在运行的情况下重新加载改变编译后的类.在这种情况下内存不会清空,sessin不会丢失,但容易造成内存溢出,或者找不到方法。因为内存无法转变成对像. 一般改变类的结构和模型就会有异常，在已经有的变量和方法中改变是不会出问题的。在中模式最好是在调试过程中使用，免得整个项目加载.
debug模式都支持热加载.很方便使用.
——IDEA TOMCAT WEB开发 SSH开发 修改类不重启 热部署 热加载 IDEA8：http://3000pzj.javaeye.com/blog/503222

部署在项目开发过程中是常有的事，特别是debug的时候。但是如果每次fix一个bug都要把整个项目重新部署一遍以便测试fix的效果或者继续debug其他bug，那对开发人员来说无疑是一大噩梦。不过谁都不想噩梦连连，有了JVM的hotSwap以及Intellij Idea对debug，hotSwap的支持，从此美梦相伴（夸张了点:)）。今天通过这篇文章介绍一下通过对Intellij Idea热部署的设置达到最方便的最高效的debug效果。
我想在介绍具体设置之前，不妨了解一些背景知识和概念。
HotSwap：“HotSwap”是JPDA（Java Platform Debugger Architecture）中的一个特性，JPDA增强是自Java 2 SDK1.4新增的功能。HotSwap允许将JVM中的类定义替换为新的类定义，这就允许开发人员在debug时，将修改过的class替换JVM中旧有的class，无需重新启动服务器。