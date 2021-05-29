---
layout: post
title:  "MobaXterm使用记录"
date:   2021-05-29 22:29:36 +0800
categories: linux
typora-root-url: ..
---
# MobaXterm使用记录

由于再也不想使用各种盗版破解软件了，想找一个全功能的终端软件替代。

MobaXterm是一款免费可商用的 SSH 客户端，可以在公司环境下使用，但是**软件使用者必须自己到官网下载，不允许在公司内批量分发本软件**。

它的优点：

- 直接的便携版

- 内建多标签和多终端分屏

- 内建SFTP文件传输

- 内建X server，可远程运行X窗口程序

- 直接支持VNC/RDP/Xdmcp等远程桌面

- 默认的UTF-8编码

- 更加友好的串口连接设置

- 操作更明确，更少的“神秘技巧”

  下载地址——[MobaXterm Xserver with SSH, telnet, RDP, VNC and X11 - Home Edition (mobatek.net)](https://mobaxterm.mobatek.net/download-home-edition.html)

## 1.密钥生成方式

- 点击菜单Tools-MobaKeyGen(SSH key generator); 

![image-20210304102209454](/assets/images/mobaXterm密钥生成.png) 

- 选择密钥类型RSA，设置为2048位。  
  ![RSA，2048](/assets/images\key-generator.png)

- 点击Generate，生成密钥（鼠标在进度条上左右移动，加快生成速度, 这不是开玩笑，真的有用！( •̀ ω •́ )✧）。  
  ![密钥生成](/assets/images\generate-key.png)

  

- 点击“Save private key”保存密钥，可以选择设置密码（key passphrase), 确认并记住密码。



![保存](/assets/images\save-private-key.png)

### 其它：使用命令生成公钥和私钥

```bash
 ssh-keygen -t rsa     ##-t rsa可以省略，默认就是生成rsa类型的密钥
```

   **说明：**命令执行后会有提示，输入三次回车即可，执行完成后会在当前用户的.ssh目录下生成两个文件：id_rsa、id_rsa.pub文件，前者是私钥文件，后者是公钥文件（拷贝到其他主机只需要拷贝这个文件的内容）

## 2.使用密钥

**将公钥复制到被登陆的主机上的 ~/.ssh/authorized_keys 文件中**

   拷贝公钥有两种方法，其原理都相同：

   **方式一：使用 ssh-copy-id 直接拷贝**

   使用 ssh-copy-id 进行拷贝公钥非常方便，只需要指定目标主机和目标主机的用户即可。

```bash
ssh-copy-id -i ~/.ssh/id_rsa.pub root@ip
```

   执行这条命令后会自动将登录主机的公钥文件内容追加至目标主机中指定用户（root）.ssh目录下的authorized_keys文件中。这个过程是全自动的，非常方便。

 

   **方法二：自己创建文件进行拷贝**

1) 在登录主机（客户机）上创建authorized_keys文件并将公钥追加到该文件。

​    先cd到登录机使用的用户下的 .ssh 目录，方便操作

```bash
cat id_rsa.pub >> authorized_keys
chmod 600 authorized_keys     ##修改文件权限为600，该文件有规定如果属组其他人出现可写则文件就不会生效
```

2) 在被登录机的指定用户家目录下创建 .ssh 目录

```bash
mkdir .ssh
chmod 700 .ssh     ##将目录权限改为700该目录的权限必须是700才有效
```

3) 将登录机创建的authorized_keys文件拷贝到被登录机，使用scp

```bash
scp authorized_keys root@ip:/root/.ssh/
```



**客户端点击session, 配置会话,填写登录的ip, 用户名，和端口。**

 指定上一步生成的private key路径进行密钥登录。

![image-20210304103113717](/assets/images\session.png)

## 3.sftp文件目录

如果在上一步配置session时，配置了SFTP protocol，那么在可使用sftp协议的服务器上，通过ssh连接后，在右边标签页scp会出现文件目录树，可通过图标上传，下载，创建文件夹。勾选下面的follow terminal folder，可使文件目录跟随终端的工作目录。6



![image-20210304103737013](/assets/images\scp.png)

## 4.rz和sz命令

rz、sz 采用 ZModem 协议，MobaXterm 采用插件实现，先到官网

[MobaXterm Xserver with SSH, telnet, RDP, VNC and X11 - Plugins (mobatek.net)](https://mobaxterm.mobatek.net/plugins.html)

下载[ lrzsz 插件](https://mobaxterm.mobatek.net/plugins/Lrzsz.mxt3)，并放入 MobaXterm.exe 可执行文件相同的目录中。

使用
与 xshell、SecureCRT 不同的是

MobaXterm 使用 rz、sz 命令进行下载、上传，不会直接弹出窗口，需要***ctrl + 鼠标右键*** 再点击Send file using Z-modem上传文件或Receive file using Z-modem选择一个目录接收文件

> ***下载***
>
> *sz filename*
>
> *ctrl + 鼠标右键*
>
> *Receive file using Z-modem*

 

> ***上传***
>
> *rz*
>
> *ctrl + 鼠标右键*
>
> *Send file using Z-modem*
>
> *选择上传文件*

![image-20210304104236710](images\szrzfile.png)

## 5. 连接WSL

很多时候需要Linux环境，但是由于某些工具，也不能完全抛弃Windows，需要双系统来回切换很是麻烦，用虚拟机的话电脑配置又太低性能损耗大。直到知道了WSL(Windows Subsystem for Linux) 适用于Linux的Windows子系统，网上有多这个东西的安装教程。mobaXterm也可以连接WSL，配色和字体渲染也更好看，

如果安装桌面环境的话，还能使用xserver。

- 使用方式：

  添加session，选择WSL，默认打开的是shell。如果使用xfce4桌面，可以选择xfce4 desktop.

![连接WSL](/assets/images\WSL.png)

- 效果

  ![ubuntu](/assets/images\WSL-ubuntu.png)

  [返回目录](https://zph-programmer.github.io)

  

  * [下一篇 ——WSL安装桌面环境](02-WSL安装桌面环境.md)

## 参考资料

**1、公私钥简介与原理**

 公钥和私钥都属于非对称加密算法的一个实现，这个加密算法的信息交换过程是：

1) 持有公钥的一方（甲）在收到持有私钥的一方（乙）的请求时，甲会在自己的公钥列表中查找是否有乙的公钥，如果有则使用一个随机字串使用公钥加密并发送给乙。

2) 乙收到加密的字串使用自己的私钥进行解密，并将解密后的字串发送给甲。

3) 甲接收到乙发送来的字串与自己的字串进行对比，如过通过则验证通过，否则验证失败。

 非对称加密算法不能使用相同的密钥进行解密，也就是说公钥加密的只能使用私钥进行解密。



**2、用途**

ssh使用私钥登录大致步骤就是：主机A（客户端）创建公钥私钥，并将公钥复制到主机B（被登陆机）的指定用户下，然后主机A使用保存私钥的用户登录到主机B对应保存公钥的用户。 

ssh密钥登录可以实现免密登录，免密登陆有很多用途：例如scp免认证、rsync备份免交互等一切使用ssh认证的地方均可以免交互，也就实现了自动化。

 

