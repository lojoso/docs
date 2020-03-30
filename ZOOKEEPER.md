Zookeeper集群搭建

[官方指南](https://zookeeper.apache.org/doc/r3.6.0/zookeeperStarted.html)

- 1. 准备zookeeper的安装包 (注:非source版)
- 2. 使用tar命令将压缩包进行解压缩

> tar -zxvf  apache-zookeeper-x.x.x-bin.tar.gz

- 3. 解压缩后进入zookeeper目录下的conf文件夹对文件zoo_sample.cfg进行配置，并重命名为zoo.cfg

> cd /xxx/zookeeper/conf  
> vim zoo_sample.cfg

```text
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial 
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just 
# example sakes.
dataDir=/data/zookeeper/data
# the port at which the clients will connect
clientPort=2181

server.1=[zoo1-hostname]:2888:3888
server.2=[zoo2-hostname]:2888:3888
server.3=[zoo3-hostname]:2888:3888
```
> mv zoo_sample.cfg zoo.cfg

* server.[n] n代表zookeeper集群的id  
* 2888为zookeeper各节点通信使用  
* 3888为选举leader使用  

而后对各个节点zookeeper的data目录添加myid文件，标识id

> echo "[n]" > /path/to/zookeeper/data/myid

最后进入zookeeper的bin目录中，启动server，对各个节点均手动启动。

> ./path/to/zookeeper/bin/zkServer.sh start

可看到正常启动，使用status命令可查看节点server状态

> ./path/to/zookeeper/bin/zkServer.sh status

```text
> ./zkServer.sh status

ZooKeeper JMX enabled by default
Using config: /opt/apache-zookeeper-3.6.0-bin/bin/../conf/zoo.cfg
Client port found: 2181. Client address: localhost.
Mode: follower

```



