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



## Linux中zip压缩和unzip解压缩命令详解

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





1、把/home目录下面的mydata目录压缩为mydata.zip
 zip -r mydata.zip mydata #压缩mydata目录
 2、把/home目录下面的mydata.zip解压到mydatabak目录里面
 unzip mydata.zip -d mydatabak
 3、把/home目录下面的abc文件夹和123.txt压缩成为abc123.zip
 zip -r abc123.zip abc 123.txt
 4、把/home目录下面的wwwroot.zip直接解压到/home目录里面
 unzip wwwroot.zip
 5、把/home目录下面的abc12.zip、abc23.zip、abc34.zip同时解压到/home目录里面
 unzip abc\*.zip
 6、查看把/home目录下面的wwwroot.zip里面的内容
 unzip -v wwwroot.zip
 7、验证/home目录下面的wwwroot.zip是否完整
 unzip -t wwwroot.zip
 8、把/home目录下面wwwroot.zip里面的所有文件解压到第一级目录
 unzip -j wwwroot.zip

主要参数

-c：将解压缩的结果
 -l：显示压缩文件内所包含的文件
 -p：与-c参数类似，会将解压缩的结果显示到屏幕上，但不会执行任何的转换
 -t：检查压缩文件是否正确
 -u：与-f参数类似，但是除了更新现有的文件外，也会将压缩文件中的其它文件解压缩到目录中
 -v：执行是时显示详细的信息
 -z：仅显示压缩文件的备注文字
 -a：对文本文件进行必要的字符转换
 -b：不要对文本文件进行字符转换
 -C：压缩文件中的文件名称区分大小写
 -j：不处理压缩文件中原有的目录路径
 -L：将压缩文件中的全部文件名改为小写
 -M：将输出结果送到more程序处理
 -n：解压缩时不要覆盖原有的文件
 -o：不必先询问用户，unzip执行后覆盖原有文件
 -P：使用zip的密码选项
 -q：执行时不显示任何信息
 -s：将文件名中的空白字符转换为底线字符
 -V：保留VMS的文件版本信息
 -X：解压缩时同时回存文件原来的UID/GID

