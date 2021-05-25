### 拉取最新版的 Ubuntu 镜像

```
$ docker pull ubuntu
```

或者：

```
$ docker pull ubuntu:latest
```

[![img](https://www.runoob.com/wp-content/uploads/2019/11/docker-ubuntu1.png)](https://www.runoob.com/wp-content/uploads/2019/11/docker-ubuntu1.png)

### 3、查看本地镜像

```
$ docker images
```

[![img](https://www.runoob.com/wp-content/uploads/2019/11/docker-ubuntu2.png)](https://www.runoob.com/wp-content/uploads/2019/11/docker-ubuntu2.png)

在上图中可以看到我们已经安装了最新版本的 ubuntu。

### 4、运行容器，并且可以通过 exec 命令进入 ubuntu 容器,指定端口映射,可以指定名称

```
$ docker run -itd --name ubuntu-test ubuntu
sudo docker run -itd -p 8099:8080 --name fiqs_front-test fiqs_front
 docker exec -it ubuntu-test /bin/bash
```

[![img](https://www.runoob.com/wp-content/uploads/2019/11/docker-ubuntu3.png)](https://www.runoob.com/wp-content/uploads/2019/11/docker-ubuntu3.png)

### 5、安装成功

最后我们可以通过 **docker ps** 命令查看容器的运行信息：



配置系统环境变量 vi /etc/environment 在文件后添加

修改用户的环境变量 vi /etc/profile  另起一行添加 

export JAVA_HOME=/export/servers/jdk1.8.0_261
export JRE_HOME=/export/servers/jdk1.8.0_261/jre
export CLASSPATH=$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/jre/lib
export PATH=$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$PATH:$HOME/bin



source /etc/environment 



解决linux下vim乱码的情况：(修改vimrc的内容）

全局的情况下：即所有用户都能用这个配置

文件地址：/etc/vimrc

在文件中添加：

```
set fileencodings=utf-8,ucs-bom,gb18030,gbk,gb2312,cp936
set termencoding=utf-8
set encoding=utf-8
```





# 1.启动镜像并做出修改

docker run -it centos /bin/bash

[root@afcaf46e8305 /]#
 注意afcaf46e8305是产生的容器ID，前面运行的时候不要-d后台运行了，不然无法进入容器交互执行模式：

安装vim并且退出容器：
 yum install -y vim
 exit

# 2.把容器打包成镜像

docker commit afcaf46e8305 centos-vim

# 3.查看镜像centos-vim

docker images | grep centos-vim
 查看镜像的详细信息：
 docker inspect centos-vim:afcaf46e8305

# 4.使用centos-vim这个镜像

docker run -it centos-vim /bin/bash
 发现可以直接使用vim了，而不需要重新安装：
 vim --version





使用docker经常会遇到这样的问题，基础镜像几百兆，在容器中安装了几个软件，然后commit到镜像。后来删除了一些内容，再次commit成镜像。发现。根本不会变小，而且会越来越大。

其实，commit，顾名思义，就是把当次的修改提交。体现在docker镜像中，就是新的一层。

![imagepng](https://imgconvert.csdnimg.cn/aHR0cDovL2F0dGFjaC5pbWFkaWFvcy5jb20vL2ZpbGUvMjAxOS8wOC9jYzczYzI5OGE2MTA0MmNhOTY5MDM0ODJkZjdiYmI3MF9pbWFnZS5wbmc?x-oss-process=image/format,png)

> 在 Dockerfile 中， 每一条指令都会创建一个镜像层，继而会增加整体镜像的大小。而commit也是层的增加。

这其实也很好理解，例如git，你对某个文件增加了一行，又删除了一这一行，虽然最新版文件看起来没有了，但其实历史还是被记录下来。

手里的这个环境并没有原始的Dockerfile，并不知道从第一版到现在做了什么。所以干脆从零开始，把当前的容器直接**做成基础镜像**。

不在废话，直接开始：

1. 查看当前目录，删除不需要内容(容器中)

   ```bash
   #进入根目录
   cd / 
   #查看各个目录体积
   du -h -d 1
   1234
   ```

2. 一顿删除操作猛如虎

3. 打包当前容器

   ```bash
   # 根目录下：
   tar --exclude=/proc --exclude=/sys --exclude=base_img.tar -cvf base_img.tar .
   12
   ```

4. 退出容器，拷贝压缩包

   ```bash
   # 退出
   exit
   
   # copy
   docker cp [容器id]:/base_img.tar .
   12345
   ```

5. 导入容器

   ```bash
   # 导入
   cat base_img.tar|docker import - base_img
   12
   ```

6. 对比：

   ```bash
   # 直观上体积减少了
   docker images
   
   # history,只有一个记录：Imported from -
   docker history [新镜像id]
   ```



## docker 配置host

在mac开发的时候，docker容器没有配置hosts，但是mac本机配置了hosts，这个本机的hosts配置对docker容器里面的所有容器都适用，但是到了linux的时候反而不适用了

可以通过下面两种方法把hosts配置到docker容器上

一、启动容器的时候加上“--add-host”把hosts配置上

```
# docker run --add-host=www.baidu.com:127.0.0.1 xxx -it /bin/bash
```

上面的容器启动之后，会把 “www.baidu.com 127.0.0.1” 这个配置写到容器的 /etc/hosts中

进入到容器中验证下

```
# docker ps -a
# docker exec xxxx -it /bin/bash    #xxx是上面执行后的容器id，CONTAINER ID# cat /etc/hosts
```

二、通过 docker-compose.yaml 文件启动

通过 extra_hosts 将hosts配置到容器中

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
php72:
  container_name: "php72"
  hostname: "php72"
  image: "xxxx"
  extra_hosts:
    - "www.baidu.com:127.0.0.1"    - "www.google.com:127.0.0.1"
  volumes:
    - xxx:xxx
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)