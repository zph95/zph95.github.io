# 如果linux 没有安装gcc，需要先安装gcc 参考reids里安装gcc

- 完成下载后解压
```
tar -xvf Python-3.7.0.tgz
```
- 创建一个新的空文件夹，用于存放py3的程序
```
mkdir python3.7
```

- 执行配置文件，编译，安装（分两步进行）
```
cd Python-3.7.0
./configure --prefix=../python3.7
make && make install
```

- 建立软链接

```
ln -s /python3.7/bin/python3.7 /usr/bin/python3
ln -s /python3.7/bin/pip3.7 /usr/bin/pip3
```

- pip3 安装需要需要的包


# 报错问题

安装时报错ModuleNotFoundError: No module named '_ctypes'的解决办法
1、安装libffi-devl

```
yum install libffi-devel
如果没有联网 手动安装所需依赖包，（yum install中要求的包） 
rpm -ivh libffi-devel-3.0.13-18.el7.x86_64.rpm 

```
2、从"make && make install "重新安装



## 更新pip

```
pip3 install pip-20.3.3-py2.py3-none-any.whl
pip3 install setuptools-40.8.0-py2.py3-none-any.whl
pip3 install Cython-3.0a5-cp37-cp37m-manylinux1_x86_64.whl 
pip3 install pycparser-2.20-py2.py3-none-any.whl 
pip3 install cffi-1.14.4-cp37-cp37m-manylinux1_x86_64.whl 
```




# pip 安装第三方包

3，如何离线给python安装模块
windows（此方法需要找一个安装好的环境导出whl文件，然后导入到新环境中去）:

1，获取whl文件

pip freeze > requirements.txt    #pip freee的意思是查看当前python安装了哪些库，保存在requirements.txt 中
pip download  -r requirements.txt   -d  ./pip_packages    #从当前环境的网络中下载requestments.txt中写的包，下载到当前目录下的pip_packages目录中，这时候你会发现，里面有很多依赖，还有一些whl文件
当然从网上直接下载也是可以的，网址https://pypi.python.org/pypi/，友情提示，炒鸡慢

2，把模块文件导入到新环境中，如果python和pip已经加入到环境变量中了，你随意在哪个文件夹下执行如下命令都可以，速度超级快哦

pip3 install --no-index --find-links=d:\packages -r requirements.txt 
## 单独安装gevent-20.12.1

```
tar -xvf gevent-20.12.1.tar.gz

```







# --find-links指定的是包文件的存放地址，-r指定的是txt文件的位置


# zlip问题 zipimport.ZipImportError: can't decompress data; zlib not available

问题分析

从错误信息分析，就是缺少了zlib的解压缩类库，安装即可

    yum -y install zlib*



# ssl module in Python is not available的解决方

pip3 install -U --force-reinstall --no-binary :all: gevent

yum install -y openssl-devel
# 其它
gunicorn 不支持windows

