通过systemctl命令可以将sshd服务加到开机自启动列表里。实现开机自动启动sshd服务。

> **[root@localhost ~]# systemctl enable sshd**

启动ssh：

```bash
 systemctl enable sshd
```



在sshd_config文件中存放了端口、控制策略等信息。

> **[root@localhost ~]# vi /etc/ssh/sshd_config**



禁止空密码用户登录。

PermitEmptyPasswords no

开启密码登录授权（默认即开启）

PasswordAuthentication yes

禁止root账户使用ssh登录，这种设置通常用于互联网服务器，防止提权后用root账户登录搞破坏。

PermitRootLogin no


