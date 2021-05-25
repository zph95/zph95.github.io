# 前置步骤
- root账户上传mysql-5.7.30-linux-glibc2.12-x86_64.tar置/usr目录，并解压
```
    cd /usr
    rz 
    tar -xvf mysql-5.7.30-linux-glibc2.12-x86_64.tar
```
- 创建mysql用户组和用户并修改权限
```
groupadd mysql
useradd -r -g mysql mysql
```

- 创建数据目录并赋予权限
```
mkdir -p  /app/mysqldata              #创建目录
chown mysql:mysql -R /app/mysqldata 
```

-   配置my.cnf
```
vim /etc/my.cnf
 内容如下
```

```
[mysqld]
bind-address=0.0.0.0
port=3306
user=mysql
basedir=/usr/mysql
datadir=/app/mysqldata
socket=/tmp/mysql.sock
log-error=/app/mysqldata/mysql.err
pid-file=/app/mysqldata/mysql.pid
sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
#character config
character_set_server=utf8
symbolic-links=0
explicit_defaults_for_timestamp=true
[client]
default-character-set=utf8mb4
#
# include all files from the config directory
#
!includedir /etc/my.cnf.d
```

# 初始化数据库
- 安装
```
cd /usr/mysql/bin
./mysqld --defaults-file=/etc/my.cnf --basedir=/usr/mysql/ --datadir=/app/mysqldata/ --user=mysql --initialize
```
<<<<<<< HEAD

- 查看密码(YON9gWmdfsgp)
=======
- 查看密码(iuosTxrrw9>t)
>>>>>>> c439d3d9e925e0785e37bb1af2e8d71b6d102f4a
```
cat /app/mysqldata/mysql.err
```

# 启动mysql，并更改root 密码
- 先将mysql.server放置到/etc/init.d/mysql中
```
cp /usr/mysql/support-files/mysql.server /etc/init.d/mysql
```
- 启动
```
service mysql start
 
ps -ef|grep mysql

```

- 修改密码

首先登录mysql，前面的那个是随机生成的。
```
cd /usr/mysql/bin
./mysql -u root -p   #bin目录下
输入随机密码
SET PASSWORD = PASSWORD('asdfasdcxz');
ALTER USER 'root'@'localhost' PASSWORD EXPIRE NEVER;
FLUSH PRIVILEGES;  
```
- 使root账户能远程登录
```
use mysql                                            #访问mysql库
update user set host = '%' where user = 'root';      #使root能再任何host访问
FLUSH PRIVILEGES;                                    #刷新
```

- 建立软链接
```
ln -s  /usr/mysql/bin/mysql    /usr/bin
```

- 建库 执行建表脚本

在命令行下(已连接数据库,此时的提示符为 mysql> ),输入 source /app/fiqsdb.sql; (注意路径不用加引号的) 回车即可
```
create database fiqs;
use fiqs;
source /app/fiqsdb.sql;

create database fiqs_quartz;
use fiqs_quartz;
source /app/quartz_init.sql;
```



# mysql 8.0版本和5.7 区别
 - 对于group by 字段不再隐式排序，如需要排序，必须显式加上order by。



 

# MySQL字符串和数字比较的坑：比较时会把字符串类型转成整数类型，从首字母开始，遇到非数字类型后终止。

    原文：
    
    Comparison operations result in a value of 1 (TRUE), 0 (FALSE), or NULL. These operations work for both numbers and strings. Strings are automatically converted to numbers and numbers to strings as necessary.
    
    也就是说在比较的时候，String是可能会被转为数字的。
    
    对于数据开头的字符串，转成数字后会自动丢弃后面的字母部分，只留下纯数字进行比较。
    
    对于没有数字的那些字符串，与数值进行比较的时候，就只剩下0去和其他数值进行比较了


​     

    注：其实字符串和数值比较最大的坑在于：它会导致查询不能用到索引，直接就影响了查询的效率。
    
    字符串比较大小是逐位从高位到低位逐个比较（按ascii码），所以字符串类型的数字18<2
