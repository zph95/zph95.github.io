---
title:  "xxl-job 使用mariadb的一个连接错误"
date:   2023-10-17 11:12:00 +0800
tags: 
    - xxl-job
    - mariadb
    - socketTimeout
toc: true 
toc_label: "Contents Table" 
toc_icon: "cog"
---

## mariadb 

MariaDB 是一个开源的关系型数据库管理系统（RDBMS），它是 MySQL 数据库的一个分支和替代品。MariaDB 的目标是提供与 MySQL 兼容的功能，并在此基础上提供更多新功能和改进。

MariaDB Connector/J is used to connect applications developed in Java to MariaDB and MySQL databases using the standard JDBC API. The library is LGPL licensed.

## 看看两个官网链接

[faq](https://mariadb.com/kb/en/mariadb-faq-mariadb-connectorj/)

We are using MariaDB Connector/J against our production AWS Aurora RDS and noticed that default socketTimeout is set to 10 seconds against Aurora...

We do have transaction layer that times out at 60 seconds. So was thinking to up it to match our higher layer timeout.

[wiki](https://mariadb.com/kb/en/about-mariadb-connector-j/)

### socketTimeout

Defined the network socket timeout (SO_TIMEOUT) in milliseconds. Value of 0 disables this timeout.
If the goal is to set a timeout for all queries, since MariaDB 10.1.1, the server has permitted a solution to limit the query time by setting a system variable, max_statement_time. The advantage is that the connection then is still usable.
Default: 0 (standard configuration) or 10000ms (using "aurora" failover configuration).
since 1.1.7

## Mysql

[mysql-connector-j](https://dev.mysql.com/doc/connector-j/8.1/en/connector-j-connp-props-networking.html)

### connectTimeout

Timeout for socket connect (in milliseconds), with 0 being no timeout.

Default Value0Since Version3.0.1

### socketTimeout

Timeout, specified in milliseconds, on network socket operations. Value "0" means no timeout.

Default Value0Since Version3.0.1

## 复习一下timeout区别

1. Connect Timeout（连接超时）：连接超时是在建立与数据库服务器的连接时等待的最大时间。如果在连接超时时间内无法建立连接，则会抛出连接超时异常。连接超时通常用于控制连接建立的时间，防止长时间的连接尝试占用资源。

2. Socket Timeout（套接字超时）：套接字超时是在已建立连接的情况下，等待从数据库服务器读取或写入数据的最大时间。如果在套接字超时时间内没有收到响应或无法完成数据读写操作，则会抛出套接字超时异常。套接字超时通常用于控制数据传输的时间，防止长时间的阻塞操作。

3. Statement Timeout（语句超时）：语句超时是在执行数据库查询或更新语句时等待的最大时间。如果在语句超时时间内没有完成执行操作，则会抛出语句超时异常。语句超时通常用于控制单个数据库操作的执行时间，防止长时间的执行操作占用资源。

4. Transaction Timeout（事务超时）：事务超时是在执行数据库事务期间等待的最大时间。如果在事务超时时间内没有完成事务的提交或回滚操作，则会抛出事务超时异常。事务超时通常用于控制事务的执行时间，防止长时间的事务操作导致锁定和资源占用的问题。

## MySQL和mariadb这点细微区别会造成什么影响？

之前将XXL-job从mysql改成连接aurora，一段时间之后生产报了个错， 本地复现一下。

#spring.datasource.url=jdbc:mariadb:aurora//***:3306/xxl_job?serverTimezone=UTC&log=true&socketTimeout=60000
spring.datasource.url=jdbc:mariadb:aurora//:3306/xxl_job?serverTimezone=UTC
spring.datasource.username=local
spring.datasource.password=local
spring.datasource.driver-class-name=org.mariadb.jdbc.Driver


```java
private static Logger logger = LoggerFactory.getLogger(JobScheduleHelperTest.class);


@Test
public void doExecute() throws InterruptedException {
Thread scheduleThreadA = new Thread(() -> testSelectForUpdate("A"));
Thread scheduleThreadB = new Thread(() -> {
testSelectForUpdate("B");
});
Thread scheduleThreadC = new Thread(() -> {
testSelectForUpdate("C");
});
scheduleThreadA.start();
scheduleThreadB.start();
scheduleThreadC.start();
Thread.sleep(1000000);
}

private void testSelectForUpdate(String threadNum){
boolean scheduleThreadToStop=false;
while (!scheduleThreadToStop) {

// Scan Job
long start = System.currentTimeMillis();

Connection conn = null;
Boolean connAutoCommit = null;
PreparedStatement preparedStatement = null;

boolean preReadSuc = true;
try {

conn = XxlJobAdminConfig.getAdminConfig().getDataSource().getConnection();
connAutoCommit = conn.getAutoCommit();
conn.setAutoCommit(false);
logger.info("try to get xxl_job_lock:{}",threadNum);
preparedStatement = conn.prepareStatement("select * from xxl_job_lock where lock_name = 'schedule_lock' for update");
preparedStatement.execute();
logger.info("get xxl_job_lock succeed:{}",threadNum);
Thread.sleep(11*1000L);
} catch (Exception e) {
if (!scheduleThreadToStop) {
logger.error(">>>>>>>>>>> xxl-job, JobScheduleHelper#scheduleThread error:{}",threadNum, e);
}
} finally {

// commit
if (conn != null) {
try {
conn.commit();
} catch (SQLException e) {
if (!scheduleThreadToStop) {
logger.error(e.getMessage(), e);
}
}
try {
conn.setAutoCommit(connAutoCommit);
} catch (SQLException e) {
if (!scheduleThreadToStop) {
logger.error(e.getMessage(), e);
}
}
try {
conn.close();
} catch (SQLException e) {
if (!scheduleThreadToStop) {
logger.error(e.getMessage(), e);
}
}
}

// close PreparedStatement
if (null != preparedStatement) {
try {
preparedStatement.close();
} catch (SQLException e) {
if (!scheduleThreadToStop) {
logger.error(e.getMessage(), e);
}
}
}
}
}
}

```


```log
java.sql.SQLInvalidAuthorizationSpecException: (conn=4295954) Communications link failure with primary host aurora01.cmex.corp:3306. Connection timed out
	at org.mariadb.jdbc.internal.util.exceptions.ExceptionFactory.createException(ExceptionFactory.java:66)
	at org.mariadb.jdbc.internal.util.exceptions.ExceptionFactory.create(ExceptionFactory.java:158)
	at org.mariadb.jdbc.MariaDbStatement.executeExceptionEpilogue(MariaDbStatement.java:266)
	at org.mariadb.jdbc.ClientSidePreparedStatement.executeInternal(ClientSidePreparedStatement.java:229)
	at org.mariadb.jdbc.ClientSidePreparedStatement.execute(ClientSidePreparedStatement.java:149)
	at com.zaxxer.hikari.pool.ProxyPreparedStatement.execute(ProxyPreparedStatement.java:44)
	at com.zaxxer.hikari.pool.HikariProxyPreparedStatement.execute(HikariProxyPreparedStatement.java)
	at com.xxl.job.admin.core.thread.JobScheduleHelperTest.testSelectForUpdate(JobScheduleHelperTest.java:55)
	at com.xxl.job.admin.core.thread.JobScheduleHelperTest.lambda$doExecute$2(JobScheduleHelperTest.java:28)
	at java.lang.Thread.run(Thread.java:748)
Caused by: java.sql.SQLException: Communications link failure with primary host aurora***. Connection timed out
on HostAddress{host='aurora***', port=3306},master=true.  Driver has reconnect connection
	at org.mariadb.jdbc.internal.failover.AbstractMastersListener.throwFailoverMessage(AbstractMastersListener.java:563)
	at org.mariadb.jdbc.internal.failover.FailoverProxy.handleFailOver(FailoverProxy.java:391)
	at org.mariadb.jdbc.internal.failover.FailoverProxy.executeInvocation(FailoverProxy.java:324)
	at org.mariadb.jdbc.internal.failover.FailoverProxy.invoke(FailoverProxy.java:294)
	at com.sun.proxy.$Proxy100.executeQuery(Unknown Source)
	at org.mariadb.jdbc.ClientSidePreparedStatement.executeInternal(ClientSidePreparedStatement.java:220)
	... 6 common frames omitted
Caused by: java.sql.SQLNonTransientConnectionException: Read timed out
	at org.mariadb.jdbc.internal.protocol.AbstractQueryProtocol.handleIoException(AbstractQueryProtocol.java:2091)
	at org.mariadb.jdbc.internal.protocol.AbstractQueryProtocol.readPacket(AbstractQueryProtocol.java:1541)
	at org.mariadb.jdbc.internal.protocol.AbstractQueryProtocol.getResult(AbstractQueryProtocol.java:1520)
	at org.mariadb.jdbc.internal.protocol.AbstractQueryProtocol.executeQuery(AbstractQueryProtocol.java:318)
	at sun.reflect.GeneratedMethodAccessor56.invoke(Unknown Source)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.mariadb.jdbc.internal.failover.impl.MastersReplicasListener.invoke(MastersReplicasListener.java:233)
	at org.mariadb.jdbc.internal.failover.FailoverProxy.executeInvocation(FailoverProxy.java:301)
	... 9 common frames omitted
Caused by: java.net.SocketTimeoutException: Read timed out
	at java.net.SocketInputStream.socketRead0(Native Method)
	at java.net.SocketInputStream.socketRead(SocketInputStream.java:116)
	at java.net.SocketInputStream.read(SocketInputStream.java:171)
	at java.net.SocketInputStream.read(SocketInputStream.java:141)
	at java.io.FilterInputStream.read(FilterInputStream.java:133)
	at org.mariadb.jdbc.internal.io.input.ReadAheadBufferedStream.fillBuffer(ReadAheadBufferedStream.java:131)
	at org.mariadb.jdbc.internal.io.input.ReadAheadBufferedStream.read(ReadAheadBufferedStream.java:104)
	at org.mariadb.jdbc.internal.io.input.StandardPacketInputStream.getPacketArray(StandardPacketInputStream.java:247)
	at org.mariadb.jdbc.internal.io.input.StandardPacketInputStream.getPacket(StandardPacketInputStream.java:218)
	at org.mariadb.jdbc.internal.protocol.AbstractQueryProtocol.readPacket(AbstractQueryProtocol.java:1539)

```

### 多机部署竞争锁

上面通过期待哦给你三个线程的方式模拟多机部署竞争锁

xxl-job通过mysql悲观锁实现分布式锁，从而避免多个服务器同时调度任务：

- 通过setAutoCommit(false)，关闭自动提交

- 通过select lock for update语句，其他事务无法获取到锁，显示排她锁。

- 进行定时调度任务的逻辑（这部分代码省略，在下面进行分析）

- 最后在finally块中commit()提交事务，并且setAutoCommit，释放for update的排他锁。


理论上，XXL-job只会有一台机器在执行调度。如果一台机器获取锁后，调度超过了10s， 其它两台机器在等待锁获取的时候就会出现sockettimeout报错。

### 为什么要设置成60S

本来可以改为0的，但是官网说了为了利用failover，最好不要设置0。至于为什么要设置成60？
这里还有个锁等待时间，

Mysql数据库采用InnoDB模式，默认参数:innodb_lock_wait_timeout设置锁等待的时间是50s，一旦数据库锁超过这个时间就会报错。

执行 SHOW VARIABLES LIKE '%timeout'， 可以看到Aurora和MYSQL一直，都是50s。那么socket Timeout 设置成60s，可以保证select for update 完整尝试获取锁。

锁等待超时报错是这样:

```log
java.sql.SQLTransientConnectionException: (conn=4299316) Lock wait timeout exceeded; try restarting transaction
	at org.mariadb.jdbc.internal.util.exceptions.ExceptionFactory.createException(ExceptionFactory.java:79)
	at org.mariadb.jdbc.internal.util.exceptions.ExceptionFactory.create(ExceptionFactory.java:158)
	at org.mariadb.jdbc.MariaDbStatement.executeExceptionEpilogue(MariaDbStatement.java:266)
	at org.mariadb.jdbc.ClientSidePreparedStatement.executeInternal(ClientSidePreparedStatement.java:229)
	at org.mariadb.jdbc.ClientSidePreparedStatement.execute(ClientSidePreparedStatement.java:149)
	at com.zaxxer.hikari.pool.ProxyPreparedStatement.execute(ProxyPreparedStatement.java:44)
	at com.zaxxer.hikari.pool.HikariProxyPreparedStatement.execute(HikariProxyPreparedStatement.java)
	at com.xxl.job.admin.core.thread.JobScheduleHelperTest.testSelectForUpdate(JobScheduleHelperTest.java:55)
	at com.xxl.job.admin.core.thread.JobScheduleHelperTest.lambda$doExecute$1(JobScheduleHelperTest.java:25)
	at java.lang.Thread.run(Thread.java:748)
Caused by: org.mariadb.jdbc.internal.util.exceptions.MariaDbSqlException: Lock wait timeout exceeded; try restarting transaction
	at org.mariadb.jdbc.internal.util.exceptions.MariaDbSqlException.of(MariaDbSqlException.java:34)
	at org.mariadb.jdbc.internal.protocol.AbstractQueryProtocol.exceptionWithQuery(AbstractQueryProtocol.java:194)
	at org.mariadb.jdbc.internal.protocol.AbstractQueryProtocol.exceptionWithQuery(AbstractQueryProtocol.java:177)
	at org.mariadb.jdbc.internal.protocol.AbstractQueryProtocol.executeQuery(AbstractQueryProtocol.java:321)
	at sun.reflect.GeneratedMethodAccessor55.invoke(Unknown Source)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.mariadb.jdbc.internal.failover.impl.MastersReplicasListener.invoke(MastersReplicasListener.java:233)
	at org.mariadb.jdbc.internal.failover.FailoverProxy.executeInvocation(FailoverProxy.java:301)
	at org.mariadb.jdbc.internal.failover.FailoverProxy.invoke(FailoverProxy.java:294)
	at com.sun.proxy.$Proxy99.executeQuery(Unknown Source)
	at org.mariadb.jdbc.ClientSidePreparedStatement.executeInternal(ClientSidePreparedStatement.java:220)
	... 6 common frames omitted
Caused by: java.sql.SQLException: Lock wait timeout exceeded; try restarting transaction
	at org.mariadb.jdbc.internal.protocol.AbstractQueryProtocol.readErrorPacket(AbstractQueryProtocol.java:1695)
	at org.mariadb.jdbc.internal.protocol.AbstractQueryProtocol.readPacket(AbstractQueryProtocol.java:1557)
	at org.mariadb.jdbc.internal.protocol.AbstractQueryProtocol.getResult(AbstractQueryProtocol.java:1520)
	at org.mariadb.jdbc.internal.protocol.AbstractQueryProtocol.executeQuery(AbstractQueryProtocol.java:318)
	... 14 common frames omitted
```
