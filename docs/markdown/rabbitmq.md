1. 安装erlang

```
sudo apt-get install erlang-nox
1
```

1. 安装Rabbitmq
   更新源

```
sudo apt-get update
sudo apt-get install rabbitmq-server
12
```

1. 启动、停止、重启、状态rabbitMq命令

```
sudo rabbitmq-server start
sudo rabbitmq-server stop
sudo rabbitmq-server restart
sudo rabbitmqctl status
1234
```

1. 启用rabbitmq自带的一个web插件，可以用来管理消息队列

```
rabbitmq-plugins enable rabbitmq_management
//rabbitmq默认端口号5672，web管理端口号是15672，管理地址为http://ip:15672

```

1. 创建用户，指定用户名以及密码

```
rabbitmqctl add_user admin 123456 //用户名admin,密码123456

```

修改密码：

```
rabbitmqctl  change_password  username'123456'

```

6.添加admin，并赋予administrator权限

```
添加admin用户，密码设置为admin。
sudo rabbitmqctl add_user  admin  admin  

赋予权限
sudo rabbitmqctl set_user_tags admin administrator 

赋予virtual host中所有资源的配置、写、读权限以便管理其中的资源
sudo rabbitmqctl  set_permissions -p / admin '.*' '.*' '.*'
12345678
```

