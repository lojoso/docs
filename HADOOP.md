#### Hadoop结合Zookeeper(高可用)、Kerberos(鉴权)  

##### 1. 解决[阿里云主机数据盘初始化](./阿里云数据盘挂载.md)

##### 2. 配置节点间的免密登录  

配置各主机的host文件，将hostname与ip互相记录  
借助keygen生成sshkey(最简单就是一路回车)
> ssh-keygen -t rsa 

```text  
> ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:dT4HFvW3yOQORw/hS0q1uPA/APEXns/1+vmTUC78pq4 root@hadoop-node-00
The key's randomart image is:
+---[RSA 2048]----+
|         .  .=.  |
|          o =.=. |
|         o.++@  +|
|         .=+X.B.=|
|        S  *+*+=.|
|            ** o |
|             += .|
|              .*.|
|           Eooo.=|
+----[SHA256]-----+

```
证书将生成在/root/.ssh/id_rsa目录下
而后将公钥拷贝进authorized_keys文件中。
> cd /root/.ssh/  
> cp id_rsa.pub authorized_keys

使用ssh-copy-id命令进行公钥传递，拷贝过程根据提示输入对应[hostname]主机的密码
> ssh-copy-id -i [hostname]

试验ssh命令+[hostname] 进行免密连接

##### 3. 每个节点[安装JDK环境](./LINUX_JDK.md)

##### 4. 准备[Zookeeper集群搭建](./ZOOKEEPER.md)

##### 5. 开始安装hadoop

* 5.1. 准备hadoop安装包(文中记录为hadoop 3.2.1)，拷贝到其中一台主机
* 5.2. 使用tar命令进行解压

> tar -zxvf hadoop-3.2.1.tar.gz

* 5.3. 编辑环境变量 /etc/profile 添加hadoop的环境变量(例)

> vim /etc/profile  
> source /etc/profile

```text

JAVA_HOME=/opt/jdk1.8.0_151
ZOOKEEPER_HOME=/opt/apache-zookeeper-3.6.0
HADOOP_HOME=/opt/hadoop-3.2.1
PATH=$PATH:$JAVA_HOME/bin:$ZOOKEEPER_HOME/bin:$HADOOP_HOME/bin
export JAVA_HOME ZOOKEEPER_HOME HADOOP_HOME PATH

```

> hadoop配置均为xml格式  
> configuration: 为根  
> configuration.property: 为配置项  
> configuration.property.name: 配置项名称声明  
> configuration.property.value: 配置项对应值  

* 5.4. 进入hadoop目录下的 etc/hadoop/ 中开始配置  

->>>>>> [官方参考](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/HDFSHighAvailabilityWithNFS.html)  
->>>>>> [相关资料](https://juejin.im/post/5e462864e51d4526ce613a8b)

* 5.4.1 core-site.xml 文件配置

|属性名称|值|说明|
|-|-|-|
|fs.defaultFS|hdfs://[hostname]:[port]||
|hadoop.tmp.dir|/path/to/hadoop/temp|临时文件目录|
|ha.zookeeper.quorum|zoo1:2181,zoo2:2181,……|高可用zookeeper集群配置|

* 5.4.2 hdfs-site.xml 文件配置

|属性名称|值|说明|
|-|-|-|
|fs.ha.automatic-failover.enabled|true|开启高可用选举|
|dfs.nameservices|[cluster]|自定义hdfs集群名称|
|dfs.ha.namenodes.[cluster]|[node-n],……|集群列表声明|
|dfs.namenode.rpc-address.[cluster].[node-n]|[node-n-host]:[port]|集群rpc声明对应hostname，port例: 8020|
|dfs.namenode.http-address.[cluster].[node-n]|[node-n-host]:[port]|集群http声明对应hostname，port例: 9870|
|dfs.namenode.shared.edits.dir|qjournal://[node-n-host]:[port];……/[cluster]|port例: 8485|
|dfs.journalnode.edits.dir|/path/to/journalnode/data|本地journalnode存放位置|
|dfs.client.failover.proxy.provider.dqcloud|org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider|官方配置|
|dfs.ha.fencing.methods|sshfence|官方配置|
|dfs.ha.fencing.ssh.private-key-files|/path/to/private_key| 私钥位置，root下默认为/root/.ssh/id_rsa |
|dfs.ha.fencing.ssh.connect-timeout|30000|超时时间|
|dfs.journalnode.http-address|0.0.0.0:8480||
|dfs.journalnode.rpc-address|0.0.0.0:8485||
|dfs.replication|[n]|副本个数|
|dfs.namenode.name.dir|/path/to/namenode|namenode目录|
|dfs.datanode.data.dir|/path/to/datanode|datanode目录|
|dfs.webhdfs.enabled|true|webhdfs开启使用(http端口)|
|dfs.namenode.datanode.registration.ip-hostname-check|false|返回hostname，非ip|
|dfs.permissions|false||
|dfs.web.ugi|supergroup|webhdfs编辑权限组|

* 5.4.3 yarn-site.xml 文件配置

|属性名称|值|说明|
|-|-|-|
|yarn.resourcemanager.ha.enabled|true|yarn高可用开启|
|yarn.resourcemanager.cluster-id|[cluster]|yarn resource-manager 集群名称|
|yarn.resourcemanager.ha.id|[node-n]|yarn声明本机id|
|yarn.resourcemanager.ha.rm-ids|[node-n],……|集群id列表|
|yarn.resourcemanager.hostname.[node-n]|[node-n-host]|对应host声明|
|yarn.resourcemanager.webapp.address.[node-n]|[node-n-host]:[port]|port例: 8088|
|hadoop.zk.address|zoo1:2181,zoo2:2181,……|zookeeper列表|
|yarn.nodemanager.aux-services|mapreduce_shuffle|默认配置|

* 5.4.4 mapred-site.xml 文件配置

|属性名称|值|说明|
|-|-|-|
|mapreduce.framework.name|yarn|MapReduce调度方法，YARN|
|mapreduce.application.classpath|/opt/hadoop-3.2.1/share/hadoop/common/\*,<br/>/opt/hadoop-3.2.1/share/hadoop/common/lib/\*,<br/>/opt/hadoop-3.2.1/share/hadoop/hdfs/\*,<br/>/opt/hadoop-3.2.1/share/hadoop/hdfs/lib/\*,<br/>/opt/hadoop-3.2.1/share/hadoop/mapreduce/\*,<br/>/opt/hadoop-3.2.1/share/hadoop/mapreduce/lib/\*,<br/>/opt/hadoop-3.2.1/share/hadoop/yarn/\*,<br/>/opt/hadoop-3.2.1/share/hadoop/yarn/lib/*||

* 5.4.5 workers 文件中添加所有主机hostname

* 5.5 环境变量修改

* 5.5.1 hadoop-env.sh 添加JAVA_HOME

```text
export JAVA_HOME=/path/to/java
```

* 5.5.2 若hadoop部署时使用root，则需修改hadoop/sbin中的 start-dfs.sh、stop-dfs.sh、start-yarn.sh、stop-yarn.sh

```text
start-dfs.sh stop-dfs.sh

HDFS_NAMENODE_USER=root
HDFS_DATANODE_USER=root
HDFS_JOURNALNODE_USER=root
HDFS_ZKFC_USER=root
```

```text
start-yarn.sh stop-yarn.sh

YARN_RESOURCEMANAGER_USER=root
YARN_NODEMANAGER_USER=root
```

* 5.6 创建并准备好配置中的各个目录结构

> /path/to/hadoop/namenode  
> /path/to/hadoop/temp  
> /path/to/hadoop/datanode  
> /path/to/journalnode/data  

* 5.7 使用scp命令同步hadoop配置等到各个主机，并做部分修改

> YARN resource-manager配置yarn.resourcemanager.ha.id修改id  
> 非ResourceManager节点删掉yarn.resourcemanager.ha.id属性  

* 5.8 启动hadoop集群

* 5.8.1 确保zookeeper集群已经启动

* 5.8.2 每个节点启动JournalNode

> hdfs --daemon start

* 5.8.3 主节点执行NameNode的format, ★并将namenode目录下的内容同步到各节点相同位置

> hadoop namenode -format

* 5.8.4 含有NameNode的节点执行zookeeper初始化

> hdfs zkfc -formatZK

* 5.8.5 每个节点停止JournalNode

> hdfs --daemon stop

* 5.8.6 启动整个hadoop集群

> /path/to/hadoop/sbin/start-all.sh


* 5.9 结束








