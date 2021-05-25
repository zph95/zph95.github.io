1. 查看文件夹占用

​      du -h --max-depth=1



1，lsof -i:端口号

2，netstat -tunlp|grep 端口号

这两个命令都可以查看端口被什么进程占用



3. ssh 用户名@IP地址 -p 端口号

4. zip -r 和unzip

5. 每次监控到tomcat的硬盘空间变小达到阈值，手动登陆服务器，切换到tomcat的logs下，手动清空

   ```
   echo " "  > catalina.out
   ```
truncate -s 0 catalina.out





4, 调整分区

sudo deepin-editor /etc/fstab

编辑fstab文件


5, windows linux子系统命令

Windows10 20.04开启WSL2并安装Ubuntu20.04 LTS后启动显示　　参考的对象类型不支持尝试的操作的解决方案：

在管理员模式下运行命令后重启即可

netsh winsock reset



linux 本地文件上传到服务器


scp /home/XXX/file.1txt  liujia@172.16.252.32:/user/XXX


从服务器下载文件


scp XXX@172.16.252.32:/user/XXX/file1.txt /home/XXX

命令


scp XXX@172.16.252.32:/user/XXX/ /home/XXX


cp报错：not a regular file


原因是 这样是相当于下载文件夹，而非文件。


解决办法是 加参数 -r 


scp -r XXX@172.16.252.32:/user/XXX/ /home/XXX


inux 远程拷贝 ：scp
 
scp 文件名  root@远程ip:/路径/ 
 
将本地home目录下的test.tar的文件拷贝到远程主机192.168.1.23的/home/adm/目录下，则命令为：scp /home/test.tar root@192.168.1.23:/home/adm/  回车后输入密码就可以了 
 
scp提供了几个选项  在scp后加就行了 
 
    -p 拷贝文件的时候保留源文件建立的时间。 
    -q 执行文件拷贝时，不显示任何提示消息。 
    -r 拷贝整个目录   [www.2cto.com](http://www.2cto.com/)  
    -v 拷贝文件时，显示提示信息。 
 


Linux 下scp传文件时错误 scp: /usr/tools: not a regular file 不能成功传送 解决方案

1：有可能没权限 chmod 777
2:  在使用scp时加上-r 参数
scp -r root@192.168.16.5:/usr/tools/xxxx



<c t r l −c>9：停止命令执行
<c t r l −d>：结束传输或屏幕输入
<c t r l −s>：临时停止输出
<c t r l −q>：恢复输出
<c t r l −u>：擦除光标以前的
<c t r l −k>：擦除光标以后的
<backspace>：纠正错误
<c t r l −r>：在以前的命令中搜索



命令在普通模式下可以用而在sudo下不可以用

出现这种情况的原因，主要是因为当 sudo以管理权限执行命令的时候，linux将PATH环境变量进行了重置，当然这主要是因为系统安全的考虑，以防用户执行可引起灾难性的程序。


这个配置信息存储在了/etc/sudoers这个文件是，当指行sudo命令的时候系统寻找的是 secure_path下的目录，所有在 /etc/profile ~/.bashrc下对PATH做的配置都会被忽略。所以就出现加上 sudo xxx 找不到命令的情况，解决办法用很多种了，


可以将要执行sudo的命令软链接到 secure_path的目录下，或者修改 secure_path变量。



liunx 文件操作命令

这里不仅仅指的是普通文件，也包括目录等文件
cp：复制文件或目录
mv：移动文件和文件换名
rm：删除文件或目录
ln：在文件间建立连接
f i n d：查找特定的文件
l o c a t e：查找特定的文件
which：查看命令的路径
touch：改变文件的时间参数


显示文件内容的命令

cat：显示和合并文件
paste：横向合并文件，将多个文件对应行合并
more：分屏显示文件
l e s s ：分屏显示文件
head：显示文件的前几行
t a i l ：显示文件的最后几行

chmod：改变文件或目录的许可权限
chown：改变文件的所有权
chgrp：改变用户分组