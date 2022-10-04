# [Linux 下修改Tomcat使用的JVM内存大小](https://www.cnblogs.com/baihuitestsoftware/articles/6382906.html)



JAVA_OPTS="-Xms1024m -Xmx4096m -Xss1024K -XX:PermSize=512m -XX:MaxPermSize=2048m"

 

正文：

 

常见的内存溢出有以下两种:

**java.lang.OutOfMemoryError: PermGen space**

**java.lang.OutOfMemoryError: Java heap space**

 

\---------------------------------------------------------

这里以tomcat环境为例，其它WEB服务器如jboss,weblogic等是同一个道理。


**一、****java.lang.OutOfMemoryError: PermGen space**

PermGen space的全称是Permanent Generation space,是指内存的永久保存区域,
这块内存主要是被JVM存放Class和Meta信息的,Class在被Loader时就会被放到PermGen space中,
它和存放类实例(Instance)的Heap区域不同,GC(Garbage Collection)不会在主程序运行期对
PermGen space进行清理，所以如果你的应用中有很多CLASS的话,就很可能出现PermGen space错误,
这种错误常见在web服务器对JSP进行pre compile的时候。如果你的WEB APP下都用了大量的第三方jar, 其大小
超过了jvm默认的大小(4M)那么就会产生此错误信息了。
解决方法： 手动设置MaxPermSize大小
建议：将相同的第三方jar文件移置到tomcat/shared/lib目录下，这样可以达到减少jar 文档重复占用内存的目的。

 

**二、****java.lang.OutOfMemoryError: Java heap space**
JVM堆的设置是指java程序运行过程中JVM可以调配使用的内存空间的设置.JVM在启动的时候会自动设置Heap size的值，
其初始空间(即-Xms)是**物理内存的****1/64**，最大空间(-Xmx)是**物理内存的****1/4**。可以利用JVM提供的-Xmn -Xms -Xmx等选项可
进行设置。Heap size 的大小是Young Generation 和Tenured Generaion 之和。
提示：在JVM中如果98％的时间是用于GC且可用的Heap size 不足2％的时候将抛出此异常信息。
提示：Heap Size 最大不要超过可用物理内存的80％，一般的要将-Xms和-Xmx选项设置为相同，而-Xmn为1/4的-Xmx值。 
解决方法：手动设置Heap size

\----------------------------------------------------------

 

Linux下修改JVM内存大小:

要添加在tomcat 的bin 下catalina.sh 里，位置cygwin=false前 。注意引号要带上,红色的为新添加的.

\# OS specific support. $var _must_ be set to either true or false.
JAVA_OPTS="-Xms256m -Xmx512m -Xss1024K -XX:PermSize=128m -XX:MaxPermSize=256m"
cygwin=false

 

windows下修改JVM内存大小:

情况一:解压版本的Tomcat, 要通过startup.bat启动tomcat才能加载配置

要添加在tomcat 的bin 下catalina.bat 里

rem Guess CATALINA_HOME if not defined
set CURRENT_DIR=%cd%后面添加,红色的为新添加的.

set JAVA_OPTS=-Xms256m -Xmx512m -XX:PermSize=128M -XX:MaxNewSize=256m -XX:MaxPermSize=256m -Djava.awt.headless=true

 

情况二:安装版的Tomcat下没有catalina.bat

windows服务执行的是bin/tomcat.exe.他读取注册表中的值,而不是catalina.bat的设置.

修改注册表HKEY_LOCAL_MACHINE/SOFTWARE/Apache Software Foundation/Tomcat Service Manager/Tomcat5/Parameters/JavaOptions
原值为
-Dcatalina.home="C:/ApacheGroup/Tomcat 5.0"
-Djava.endorsed.dirs="C:/ApacheGroup/Tomcat 5.0/common/endorsed"
-Xrs

加入 -Xms300m -Xmx350m 
重起tomcat服务,设置生效

 

\---------------------------------------------------------

**各参数的比例****:**

Xmx 与PermSize的和不可超过JVM可获得的总内存

PermSize不可大于Xmx

================

**如何设置****Tomcat****的****JVM****虚拟机内存大小**

可以给Java虚拟机设置使用的内存，但是如果你的选择不对的话，虚拟机不会补偿。可通过命令行的方式改变虚拟机使用内存的大小。如下表所示有两个参数用来设置虚拟机使用内存的大小。
参数
描述
-Xms
JVM初始化堆的大小
-Xmx
JVM堆的最大值
这 两个值的大小一般根据需要进行设置。初始化堆的大小执行了虚拟机在启动时向系统申请的内存的大小。一般而言，这个参数不重要。但是有的应用程序在大负载的 情况下会急剧地占用更多的内存，此时这个参数就是显得非常重要，如果虚拟机启动时设置使用的内存比较小而在这种情况下有许多对象进行初始化，虚拟机就必须 重复地增加内存来满足使用。由于这种原因，我们一般把-Xms和-Xmx设为一样大，而堆的最大值受限于系统使用的物理内存。一般使用数据量较大的应用程 序会使用持久对象，内存使用有可能迅速地增长。当应用程序需要的内存超出堆的最大值时虚拟机就会提示内存溢出，并且导致应用服务崩溃。因此一般建议堆的最 大值设置为可用内存的最大值的80%。
Tomcat默认可以使用的内存为128MB，在较大型的应用项目中，这点内存是不够的，需要调大。
Windows下，在文件/bin/catalina.bat，Unix下，在文件/bin/catalina.sh的前面，增加如下设置：
JAVA_OPTS='-Xms【初始化内存大小】 -Xmx【可以使用的最大内存】'
需要把这个两个参数值调大。例如：
JAVA_OPTS='-Xms256m -Xmx512m'
表示初始化内存为256MB，可以使用的最大内存为512MB。
另 外需要考虑的是Java提供的垃圾回收机制。虚拟机的堆大小决定了虚拟机花费在收集垃圾上的时间和频度。收集垃圾可以接受的速度与应用有关，应该通过分析 实际的垃圾收集的时间和频率来调整。如果堆的大小很大，那么完全垃圾收集就会很慢，但是频度会降低。如果你把堆的大小和内存的需要一致，完全收集就很快， 但是会更加频繁。调整堆大小的的目的是最小化垃圾收集的时间，以在特定的时间内最大化处理客户的请求。在基准测试的时候，为保证最好的性能，要把堆的大小 设大，保证垃圾收集不在整个基准测试的过程中出现。
如果系统花费很多的时间收集垃圾，请减小堆大小。一次完全的垃圾收集应该不超过 3-5 秒。如果垃圾收集成为瓶颈，那么需要指定代的大小，检查垃圾收集的详细输出，研究 垃圾收集参数对性能的影响。一般说来，你应该使用物理内存的 80% 作为堆大小。当增加处理器时，记得增加内存，因为分配可以并行进行，而垃圾收集不是并行的。
Tomcat 5常用优化和配置
1、JDK内存优化：
Tomcat默认可以使用的内存为128MB,Windows下,在文件{tomcat_home}/bin/catalina.bat，Unix下，在文件{tomcat_home}/bin/catalina.sh的前面，增加如下设置：
JAVA_OPTS='-Xms[初始化内存大小] -Xmx[可以使用的最大内存]
一般说来，你应该使用物理内存的 80% 作为堆大小。
2、连接器优化：
在tomcat配置文件server.xml中的配置中，和连接数相关的参数有：
maxThreads：
Tomcat使用线程来处理接收的每个请求。这个值表示Tomcat可创建的最大的线程数。默认值150。
acceptCount：
指定当所有可以使用的处理请求的线程数都被使用时，可以放到处理队列中的请求数，超过这个数的请求将不予处理。默认值10。
minSpareThreads：
Tomcat初始化时创建的线程数。默认值25。
maxSpareThreads：
一旦创建的线程超过这个值，Tomcat就会关闭不再需要的socket线程。默认值75。
enableLookups：
是否反查域名，默认值为true。为了提高处理能力，应设置为false
connnectionTimeout：
网络连接超时，默认值60000，单位：毫秒。设置为0表示永不超时，这样设置有隐患的。通常可设置为30000毫秒。
maxKeepAliveRequests：
保持请求数量，默认值100。
bufferSize：
输入流缓冲大小，默认值2048 bytes。
compression：
压缩传输，取值on/off/force，默认值off。
其中和最大连接数相关的参数为maxThreads和acceptCount。如果要加大并发连接数，应同时加大这两个参数。web server允许的最大连接数还受制于*作系统的内核参数设置，通常Windows是2000个左右，Linux是1000个左右。
3、tomcat中如何禁止和允许列目录下的文件
在{tomcat_home}/conf/web.xml中，把listings参数设置成false即可，如下：
<servlet>
...
< init-param>
< param-name>listings</param-name>
< param-value>false</param-value>
< /init-param>
...
< /servlet>
4、tomcat中如何禁止和允许主机或IP地址访问
<Host name="localhost" ...>
...
< Valve className="org.apache.catalina.valves.RemoteHostValve"
allow="*.mycompany.com,www.yourcompany.com"/>
< Valve className="org.apache.catalina.valves.RemoteAddrValve"
deny="192.168.1.*"/>
...
< /Host>
服务器的配置
JAVA_OPTS='-server -Xms512m -Xmx768m -XX:NewSize=128m -XX:MaxNewSize=192m -XX:SurvivorRatio=8'

 

 

---------------------其他可供参考----------------------------

CentOS7下面[**测试**](http://lib.csdn.net/base/softwaretest)可用，特此记录。 

## Tomcat自动开机启动配置

1、/etc/rc.d/rc.local 文件最后加上下面俩行脚本。 

1. export JAVA_HOME=/home/jdk/jdk1.7.0_79 
2. /home/tomcat/apache-tomcat-7.0.47/bin/startup.sh start 

JAVA_HOME 是你jdk的安装路径

/home/tomcat/apache-tomcat-7.0.47  是tomcat的安装目录

 

2、修改rc.local文件为可执行，如： chmod +x rc.local

 

3、重启机器：shutdown -r now

 

### 配置内存等相关设置

[**Linux**](http://lib.csdn.net/base/linux)下修改JVM内存大小:
要添加在tomcat 的bin 下catalina.sh 里，位置cygwin=false前 。注意引号要带上,export的为新添加的.
\# OS specific support. $var _must_ be set to either true or false.
export JAVA_OPTS="-Xms512m -Xmx1024m -XX:PermSize=128m -XX:MaxPermSize=256m"
cygwin=false

### atomikos

在项目中使用atomikos时,如果在同一个环境中部署两个以上这种项目，则可能会报出com.atomikos.icatch.SysException: Error in init(): Log already in use异常，这个信息是因为atomikos在默认情况下是将console_file_name和log_base_name设置为默认值：tm.out和tmlog0.log,并且会将这两个文件上锁，导致其他线程无法访问，所以当多个项目都未指定这一名称时就会出现上述异常信息
解决办法：
在每一个项目中都指定atomikos的文件名称，修改jta.properties文件中的
com.atomikos.icatch.console_file_name
com.atomikos.icatch.log_base_name
两个属性的值，保证每个项目的名称都不一样
例如：
第一个项目中使用默认值，则自动生成为tm.out、tm.out.lck和tmlog0.log、tmlog.log.lck四个文件；
第二个项目中在jta.properties文件中指定属性名称：
com.atomikos.icatch.console_file_name = rm.out
com.atomikos.icatch.log_base_name = rmlog.log
启动服务时就会自动生成rm.out、rm.out.lck和rmlog0.log、rmlog.log.lck四个文件；
这时两个项目使用的文件就不会产生冲突
问题解决
-----------
实际解决办法：另外解压一个tomcat，配置一下就好了。
