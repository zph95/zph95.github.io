自动清理日志步骤

常用命令：
crontab -l 查看已有的定时任务
crontab -e 新增编辑定时任务
wq 保存并退出
/sbin/service crond reload 重新载入定时任务，新增后需重载才能生效

例子：

```

* * */3 * * echo > /home/wy/www/provider.pms.com/logs/catalina.out
* * */3 * * ech0 > cat /home/wy/www/web.pms.com/logs/catalina.out

```

说明：
cron 表达式 * * * * *
第一个分钟
第二个小时
第三个天
第四个月
第五个周 
例子中标识每三天执行一次，其它需求需自行百度查询

第一步：crontab -e进入编辑页面
第二步：新增或编辑定时任务，参考【例子】
第三步：wq 保存
第四步：/sbin/service crond reload 重载



catalina.out文件写满了整个磁盘。尝试过修改tomcat下面的logging.properties文件。但是无效。最终通过修改输出文件为/dev/null解决。这种方式是把输出日志输出到空设备，它的缺点就是看不到输出日志了。

具体操作就是，修改catalina.sh文件的CATALINA_OUT值如下：

```kotlin
if [ -z "$CATALINA_OUT" ] ; then
  #CATALINA_OUT="$CATALINA_BASE"/logs/catalina.out
  CATALINA_OUT=/dev/null
fi
```

1.重定向清空文件

```bash
[root@hb logs]# du -h catalina.out  查看文件大小
[root@hb logs]# > catalina.out   重定向清空文件
[root@hb logs]# du -h catalina.out  查看文件大小
123
```

2.重定向true命令清空文件

```bash
[root@hb logs]# du -h catalina.out
[root@hb logs]# true > catalina.out 
[root@hb logs]# du -h catalina.out 
123
```

3.使用cat/cp/dd命令及/dev/null设备来清空文件

```bash
[root@hb logs]# du -h  catalina.out
[root@hb logs]# cat /dev/null > catalina.out
[root@hb logs]#  du -h  catalina.out

[root@hb logs]# du -h  catalina.out
[root@hb logs]# cp /dev/null > catalina.out
[root@hb logs]#  du -h  catalina.out

[root@hb logs]# du -h  catalina.out
[root@hb logs]# dd if=/dev/null of=catalina.out
[root@hb logs]#  du -h  catalina.out
1234567891011
```

4.echo命令清空文件
 echo -n " " > catalina.out ==》加上"-n"参数，默认情况下会"\n"，也就是回车符

```bash
[root@hb logs]# du -h  catalina.out
[root@hb logs]# echo -n  " " > catalina.out
[root@hb logs]#  du -h  catalina.out
123
```

5.truncate命令清空文件
 truncate -s 0 catalina.out -s参数是设置文件的大小，清空文件的话，就设定为0

```bash
[root@hb logs]# du -h  catalina.out
[root@hb logs]# truncate -s 0 catalina.out
[root@hb logs]#  du -h  catalina.out
```