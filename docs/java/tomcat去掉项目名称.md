
```
需求： 
把发布到Tomcat下的web项目，访问路径去掉项目名称 

实现方式及原理： 

方式一： 
原理：Tomcat的默认根目录是ROOT，实际上ROOT这个项目在实际生产环境是没有用的，所以我们可以用我们的项目覆盖ROOT项目 
操作过程： 
1.删除ROOT下所有文件及文件夹 
2.把我们项目的war包解压后，项目目录下的所有文件和子目录都拷贝到ROOT目录下即可 
或者有更狠的一招：直接删掉ROOT目录，然后把我们的项目打包名称改成ROOT.war，放到webapps下就行 

方式二： 
原理：Tomcat本身可以配置虚拟目录。方法就是在Server.xml中<Engine><Host>节点下加入Context信息。如我们可以配置<Context path="/abc" docBase="D:\app\abc" ...  />，那我们可以通过地址http://localhost:8080/abc来访问我们放在D:\app\下面的abc项目。我们可以把这个path="/abc"修改为path=""。意思就是把abc映射到根目录，访问路径就会变成http://localhost:8080/。 
操作过程： 
按照配置虚拟目录的方式，在<Engine><Host>下添加一个Context节点，具体配置如下： 

1. <Engine name="Catalina" defaultHost="localhost"...> 
2. ... 
3.   <Host name="localhost" appBase="webapps" unpackWARs="true" autoDeploy="true"> 
4.   <Context path="" docBase="Interface" reloadable="true" /> 
5. <!--注：我这里使用的是相对路径，Interface项目是放在Tomcat的webapps目录下的，当然也可以改为绝对路径--> 
6. ... 
7.   </Host> 
8. ... 
9. </Engine> 


访问方式就可以用http://localhost:8080/SearchReqService.asmx?wsdl了 
如果用虚拟目录的方式，地址http://localhost:8080/Interface/SearchReqService.asmx?wsdl也可以访问。 
同样的方式，我们可以为path指定不同的路径，解决访问路径区别项目名称的需要。 


其它，去掉访问的端口号8080.就是利用了HTTP请求访问的端口默认是80的方式实现的，iis也一样。我们只用把Tomcat的HTTP监听端口号改为80（修改<Connector port="8080" protocol="HTTP/1.1"这里的端口号为80）即可。 
```