# 前置步骤
# 如果linux 没有安装gcc，需要先安装gcc

- 如果已经联网
```
yum install gcc
```

- 若没有联网，上传并解压，离线第一步里yum所需安装的依赖包
```
tar -xvf gcc.tar.gz
压缩包里有很多依赖，yum install gcc 显示只要安装着两个
rpm -ivh cpp-4.8.5-36.el7.x86_64.rpm  gcc-4.8.5-36.el7.x86_64.rpm 
```

# 安装redis
- root账户上传redis-4.0.14.tar.gzr置/usr目录，并解压
```
    cd /usr
    rz 
    tar -xvf redis-4.0.14.tar.gz
    mv redis-4.0.14 redis
```

- 编译,编译成功后，进入src文件夹，执行make install进行Redis安装。
```
make 
cd src
make install
```

- 测试redis是否安装成功，启动redis
```
./redis-server ../redis.conf
```

-  更换绑定ip,设置密码,设置后台启动redis

编辑conf文件，更改bing ip地址(本机的IP地址)，修改里面的requirepass，这个本来是注释起来了的，将注释去掉，并将后面对应的字段设置成自己想要的密码,将daemonize属性改为yes（表明需要在后台运行）
```
cd ..
vim redis.conf 

编辑保存

cd src 
./redis-server ../redis.conf
ps -ef|grep redis
```

redis连接命令
redis-cli -h 127.0.0.1 -p 6379 -a "mypass"
127.0.0.1 换成机器ip地址，mypass：密码

- 其它
```
对于Redis中bind的正确的理解是：

bind：是绑定本机的IP地址，（准确的是：本机的网卡对应的IP地址，每一个网卡都有一个IP地址），而不是redis允许来自其他计算机的IP地址。

如果指定了bind，则说明只允许来自指定网卡的Redis请求。如果没有指定，就说明可以接受来自任意一个网卡的Redis请求。

 

举个例子：如果redis服务器（本机）上有两个网卡，每一个网卡对应一个IP地址，例如IP1和IP2。（注意这个IP1和IP2都是本机的IP地址）。

我们的配置文件：bind IP1。  只有我们通过IP1来访问redis服务器，才允许连接Redis服务器，如果我们通过IP2来访问Redis服务器，就会连不上Redis。

 

查看本地的网卡对应的IP地址：使用ifconfig命令。

 

从上面看出我们有两个网卡，也就是我们只能使用：127.0.0.1和172.18.235.206最为bind的地址，不然redis启动不起来。

这就说明了上面例子（bind 10.0.0.1）为什么启动不起来，因为我们没有对应的网卡IP地址。这就说明了bind并不是指定redis中可以接受来自哪些服务器请求的IP地址。

而是：bind用于指定本机网卡对应的IP地址。

附注：

bind 127.0.0.1的解释：（为什么只有本机可以连接，而其他不可以连接）

我们从ifconfig可以看出：lo网卡（对应127.0.0.1IP地址）：是一个回环地址（Local Loopback），也就是只有本地才能访问到这个回环地址，而其他的计算机也只能访问他们自己的回环地址。

那么来自这个lo网卡的计算机只有本机，所以只有本机可以访问，而其他计算机不能访问。

 

bind 172.18.235.206的话，只要通过这个网卡地址（172.18.235.206）来的Redis请求，都可以访问redis。我使用的阿里云的服务器。我在另一个服务器上去请求              redis-cli 阿里云公网IP地址        就会连接到redis服务器。

因为公网地址的请求：都是经过这个eth0的网卡地址（172.18.235.206），从而接收到这个redis请求。

 

当你们不使用那个回环地址，基本上外部的计算机都可以访问本机的Redis服务器。

 

如果我们想限制只有指定的主机可以连接到redis中，我们只能通过防火墙来控制，而不能通过redis中的bind参数来限制。
```