# 一、下载
apache-zookeeper-3.5.8-bin.tar.gz

wget http://archive.apache.org/dist/zookeeper/zookeeper-3.5.8/apache-zookeeper-3.5.8-bin.tar.gz

# 二、解压
tar apache-zookeeper-3.5.8-bin.tar.gz
# 三、修改配置
cd zookeeper
cp conf/zoo_sample.cfg conf/zoo.cfg
vim conf/zoo.cfg

# 四、启动
cd bin/
./zkServer.sh start

# 五、查看状态
./zkServer.sh status

