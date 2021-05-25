# SQLite数据库
有的时候 我们开发不需要一定要用到mysql、oracle等数据库，Sqlite也是我们的一种选择。

## 所需依赖
在maven pom文件中引入所需依赖,添加必要的驱动包
```xml
 <!-- https://mvnrepository.com/artifact/org.xerial/sqlite-jdbc -->
<dependency>
    <groupId>org.xerial</groupId>
    <artifactId>sqlite-jdbc</artifactId>
    <version>3.34.0</version>
</dependency>
```

## 创建一个SQLite数据库
值得注意的是，这里我们是在resources目录下创建了sqlite.db文件，意味着jar包每次启动，war包每次部署都是启用了新的sqlite.db数据库。当然，我们打包之前对sqlite.db执行的初始化脚本还是在的。除了表结构，初始数据，程序运行时产生的数据都可以会被重置。如果想不重置的话，我们可以在程序外部建立sqlite.db数据库，建立之前检查是否存在。但这里，我想要的是一个每次启动时初始化的，干净的数据库。
```java
import java.sql.Connection;
import java.sql.DriverManager;
import lombok.extern.slf4j.Slf4j;

@Slf4j
public class SqliteUtils {
    private static void createDateBase(String dbFileName) {
        try {
            //建立一个数据库名database.db的连接，如果不存在就在当前目录下创建之
            String url="jdbc:sqlite:"+dbFileName;
            Connection conn = DriverManager.getConnection(url);
            conn.close(); //结束数据库的连接
        } catch (Exception e) {
            log.error("创建sqlite数据库失败!",e);
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        try {
            //连接SQLite的JDBC
            Class.forName("org.sqlite.JDBC");
        }catch (Exception e){
            log.error("驱动未找到");
        }
        createDateBase("springboot-web/src/main/resources/static/sqlite/sqlite.db");
    }
}
```

## 创建表和触发器
这里建立了一张rest接口日志表，用于记录上一篇中过滤器获取的Rest接口信息。值得注意的有三点：
1. id自增。
2. 使用了is_valid作为逻辑删除标志位.
3. 每次insert时获取created_time，并使用触发器使得每次update时更新modified_time  

这三点可以作为任何表结构的基础。
![创建表](images/SQLiteDB.png)
```sql
CREATE TABLE rest_call_log_record (
	id INTEGER NOT NULL PRIMARY KEY AUTOINCREMENT,
	user_id INTEGER,
	method TEXT NOT NULL,
	uri TEXT NOT NULL,
	request TEXT,
	response TEXT,
	status INTEGER DEFAULT 0 NOT NULL,
	cost_time INTEGER,
	is_valid INTEGER DEFAULT 1 NOT NULL,
	created_time TIMESTAMP default (datetime('now', 'localtime')),
	modified_time TIMESTAMP default (datetime('now', 'localtime'))
);

CREATE TRIGGER rest_call_log_record_update  after update on rest_call_log_record
BEGIN
	update rest_call_log_record set modified_time =datetime('now', 'localtime') where id=old.id;
END
```

## 其它参考
[官方文档](https://www.sqlite.org/docs.html)

## Sqlite使用场景

### 小型网站

SQLite适用于中小规模流量的网站.

日访问在10万以下的网站可以很好的支持,适用于读多写少的操作,如管理员在后台添加数据,其他访客多为浏览.

10万/天是一个临界值,事实上在100万的数据量之下,SQLite的表现还是可以的,再往上就不适合了.

使用它无需单独购买数据库服务,无需服务器进程,配置成本几乎为零,加上数据的导入导出都是复制文件,维护难度也几乎为零,迁移到别的服务器无需任何配置即可支持,加上其读取的速度非常快,省去了远程数据库的链接,能够极大提升网站访问速度.

### 嵌入式设备

SQLite适用于手机, PDA, 机顶盒, 以及其他嵌入式设备. 作为一个嵌入式数据库它也能够很好的应用于客户端程序.

因为其轻量,小巧,不怎么占用内存,数据的读写性能好,加上嵌入式设备数据量并不大,不需要频繁的维护,所以比较适合.


### 数据库教学

SQLite 支持 SQL92（SQL2）标准的大多数查询语言的功能。

其无配置,无依赖,小巧,单一文件的特性让它的安装和使用非常简单,非常适合用来讲解SQL语句.

学生可以在很短的时候使用并操作SQLite,不受系统和商业限制等影响,学习的结果可以通过邮件或者云文件等形式发送给老师进行评分.

可以通过它快速实现一个最小化应用,适合学生快速了解SQLite,以及SQL语法,从而实现数据库的触类旁通,了解其他数据库系统的设计实现原则.

### 本地应用程序

其单一磁盘文件的特性,并且不支持远程连接,使其适用于本地的应用程序,如PC客户端软件.

常用的应用类型为金融分析工具、CAD 包、档案管理程序等等. (手机上的通讯录也是用此开发的)

没有远程,意味着适用于内部或者临时的数据库,用来处理一些数据,让程序更加灵活.

### 不适用场景

很明显其适合小型网站,相对的就不适合高流量网站,也不适合超大的数据集,在其缺点也提到,不适合高并发访问.

* [返回目录](https://zph-programmer.github.io)
    * [上一篇 —— 使用过滤器打印Rest接口日志](06-使用过滤器打印Rest接口日志.md)
    * [下一篇 —— Mybatis-generator的使用](08-Mybatis-generator的使用.md)