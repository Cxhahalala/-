在本地搭建虚拟机集群是足够学习Hbase的。

这篇文章将会介绍如何在本地通过虚拟机的方式来搭建Hbase集群。

# 准备虚拟机

我所使用的linux系统是ubuntu18系统，如果你们的虚拟机系统不是ubuntu系统，可以**修改对应命令**即可。

这一步只需要搭建一个虚拟机即可，不做介绍。

虚拟机的配置2g内存，20g硬盘空间即可，网络使用桥接模式。

# 前置工作

## 使用远程连接工具连接虚拟机

使用远程连接工具可以方便我们操作虚拟机和上传文件。

我使用的是Xshell和Xftp。

Xshell默认使用**ssh连接**，因此需要在虚拟机中安装ssh。

```
#安装ssh服务
sudo apt-get install openssh-server
#允许ssh开机自启动
sudo systemctl enable ssh
#查看ssh状态，如果出现active字段就证明ssh成功启动
sudo ssh status
#查看虚拟机状态，如果为inactive则为已关闭，否则需要关闭防火墙
sudo ufw status
#禁用防火墙
sudo ufw disable
```

在连接虚拟机前ubuntu可以使用net-tools的`ifconfig`查看虚拟机的ip。

如下的ens33中的inet就是我的虚拟机ip地址。

```
hbase@ubuntu:~$ ifconfig
ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.3.218  netmask 255.255.255.0  broadcast 192.168.3.255
        inet6 fe80::3a5d:8626:5cb8:f39b  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:7e:08:e8  txqueuelen 1000  (Ethernet)
        RX packets 14063  bytes 11253522 (11.2 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 5138  bytes 456250 (456.2 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 322  bytes 31557 (31.5 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 322  bytes 31557 (31.5 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

知道ip地址之后就可以使用xshell连接虚拟机了。

## 授予当前用户最高权限

授予当前用户最高权限将简化许多操作。

**切换root用户**

```
sudo su
```

**将当前用户添加到sudo组**

将下面的hbase替换为你自己实际的用户名

```
sudo usermod -aG sudo hbase
```

**切换为正常用户**

```
su - hbase
```

**查看用户权限**

```
sudo -l
```

见到下面内容，则证明已经授予最高权限

```
root@node1:/home/hbase# su - hbase
hbase@node1:~$ sudo -l
Matching Defaults entries for hbase on node1:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User hbase may run the following commands on node1:
    (ALL : ALL) ALL
```

## 创建软件安装文件夹

本集群一共需要安装Java,Hadoop,Zookeeper,Hbase，因此创建一个文件夹来存放这些软件是十分有必要的。

本教程软件统一存放在`/opt/moudle` 目录下，软件压缩包统一存放在`/opt/software` 目录下。

**创建目录**

```
sudo mkdir /opt/moudle
sudo mkdir /opt/software
```

**更改文件夹的所有者**

```
sudo chown hbase:hbase /opt/moudle
sudo chown hbase:hbase /opt/software
```

# Java

将Java压缩包通过Xftp传输到/opt/software目录下。

**Java版本为1.8。**

## **解压**

```
tar -zxvf jdk-linux-x64.tar.gz -C /opt/moudle
```

## **配置环境变量**

在/opt/profile.d目录下新建**my_env.sh**

```
cd /opt/profile.d
sudo vim my_env.sh
```

**写入以下内容**

```
#JAVA_HOME
export JAVA_HOME=/opt/moudle/jdk1.8.0_131
export PATH=$PATH:$JAVA_HOME/bin
```

## **加载配置文件**

```
source /etc/profile.d/my_env.sh
```

**检查java**

在控制台输入java -version,出现java的版本则证明Java安装完成

```
hbase@node1:/etc/profile.d$ java -version
java version "1.8.0_131"
Java(TM) SE Runtime Environment (build 1.8.0_131-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.131-b11, mixed mode)
```

# Hadoop

将hadoop的压缩包传到`/opt/software`目录下，Hadoop版本为3.13。

版本若不一致则把下面命令中的版本号替换为自己的版本号即可。

## 解压

```
tar -zxvf hadoop-3.1.3.tar.gz -C /opt/moudle
```

## 配置Hadoop环境变量

修改配置文件my_env.sh

 `cd /etc/profile.d`

```
sudo vim my_env.sh
```

将以下内容粘贴进配置文件



```
#如果你的安装目录和我不一样或版本号不一样，替换为自己的即可
#HADOOP_HOME
export HADOOP_HOME=/opt/moudle/hadoop-3.1.3
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin
```

重新加载配置文件

```
source /etc/profile.d/my_env.sh
```

在控制台输入`hadoop version` 

若能显示安装的Hadoop版本则证明Hadoop的环境变量已经配置完成。

## 为Hadoop单独设置JAVA_HOME

Hadoop的运行需要依赖于Java，即便已经配置了全局环境变量JAVA_HOME，也需要在Hadoop的配置文件中指定JAVA_HOME来指定Java所在的位置来告诉Hadoop使用哪个Java环境。

进入hadoop配置文件夹

```
cd /opt/moudle/hadoop-3.1.3/etc/hadoop
```

修改文件

```
vim hadoop-env.sh
```

找到JAVA_HOME去掉注释并填上自己的JAVA_HOME位置

```
# The java implementation to use. By default, this environment
# variable is REQUIRED on ALL platforms except OS X!
export JAVA_HOME=/opt/moudle/jdk1.8.0_131
```

# Zookeeper

将zookeeper压缩包传输到/opt/software目录下

进入/opt/software文件夹将zookeeper压缩包解压到`/opt/moudle` 

```
tar -zxvf apache-zookeeper-3.6.4-bin.tar.gz -C /opt/moudle/
```

进入`/opt/moudle`修改文件夹名

```
mv apache-zookeeper-3.6.4-bin zookeeper-3.6.4
```

## **设置数据存储文件夹**

```
mkdir /opt/moudle/zookeeper-3.6.4/zkData
```

## **修改配置文件名**

```
/opt/moudle/zookeeper-3.6.4/conf
mv zoo_sample.cfg zoo.cfg
```

## **修改配置文件**

```
vim zoo.cfg
```

第一修改数据目录，第二修改占用端口。

zookeeper默认占用8080端口，建议修改。

```
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
#设置数据存储目录
dataDir=/opt/moudle/zookeeper-3.6.4/zkData
# the port at which the clients will connect
clientPort=2181
#zookeeper默认占用8080端口，建议改为其它端口，我这里是8090端口
admin.serverPort=8090
```

# Hbase

## 解压

使用远程连接工具将Hbase压缩包上传到/opt/software目录下。

将Hbase压缩包解压到/opt/moudle目录下。

```
cd /opt/software
tar -zxvf hbase-2.4.5-bin.tar.gz -C /opt/moudle/ 
```

## 配置环境变量

```
cd /etc/profile.d/
sudo vim my_env.sh
```

将下面内容插入进my_env.sh

```
#HBASE_HOME
export HBASE_HOME=/opt/moudle/hbase-2.4.5
export PATH=$PATH:$HBASE_HOME/bin
```

在控制台中输入hbase version如果能看到hbase的版本就证明hbase环境变量成功

## 配置JAVA_HOME

```
cd /opt/moudle/hbase-2.4.5/conf
#找到配置文件的export JAVA_HOME解开注释并填充为自己实际的JAVA_HOME
export JAVA_HOME=/opt/moudle/jdk1.8.0_131
```

# 克隆虚拟机

克隆虚拟机要注意的地方是需要

**创建完整克隆**

克隆两台即可

# 修改主机名和Hosts文件

**推荐给自己的每台虚拟机都设置一个静态ip，否则虚拟机一旦重启或换网络ip就会变化需要重新修改hosts文件**

在克隆虚拟机后，需要为每台虚拟机分配独立的主机名和ip地址。确保每台虚拟机都有唯一的标识以便Hadoop,Zookeeper,Hbase识别不同的节点。

## 修改主机名

当前虚拟机的主机名设置为node1，后续两台分别设置为node2与node3，现在先设置node1。

`sudo hostnamectl set-hostname node1`  #在第一台机器上运行

`sudo hostnamectl set-hostname node2`  #在第二台虚拟机上运行

`sudo hostnamectl set-hostname node3`  #在第三台虚拟机上运行

之后可以使用hostname查询当前主机的主机名。

## 编辑hosts文件

**这个命令三台虚拟机都需要执行**

```
sudo vim /etc/hosts
```

使用`hostnamectl` 命令即可查看当前主机名

**修改hosts文件**

```
127.0.0.1       localhost
127.0.1.1       ubuntu

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
#这个是需要加入的内容，当前主机的主机名为node1，ip为192.168.3.219，后面的node2与node3的ip可以先乱写，后面克隆完虚拟机后再修改
192.168.244.163 node1
192.168.244.48 node2
192.168.244.250 node3
```

# 配置SSH

Hadoop启动集群需要通过ssh登录到其它节点来启动Hadoop集群，因此需要为每一个节点都配置ssh。

ssh已经安装，无需重新安装。

## 生成SSH密钥对

### node1,node2,node3都需要执行

```
ssh-keygen -t rsa -P "" -f ~/.ssh/id_rsa
```

## 将公钥添加到其它节点

### **三个节点都要执行，确保任意两个节点都可以ssh无密码登录**

**hbase替换为你们自己用户名**

```
ssh-copy-id -i ~/.ssh/id_rsa.pub hbase@node1  # 将公钥添加到node1
ssh-copy-id -i ~/.ssh/id_rsa.pub hbase@node2  # 将公钥添加到 node2
ssh-copy-id -i ~/.ssh/id_rsa.pub hbase@node3  # 将公钥添加到 node3
```

## SSH登录

第一次登录需要密码

三台机器任意两台可以正常切换则ssh配置成功

```
ssh node1
ssh node2
ssh node3
```

在第一次输入密码后可以免密登录即可

# 配置Hadoop集群

**我这里把Hadoop的master节点设为node1**

### core-site.xml

**core-site.xml是三个节点都要修改**

这个配置文件是指定namenode的地址的，我们的集群需要一个namenode来管理datanode。

```
cd /opt/moudle/hadoop-3.1.3/etc/hadoop/
vim core-site.xml
```

将下面内容插入到configuration标签中。：

**设置namenode地址**

```
  <property>
        <name>fs.defaultFS</name>
        <value>hdfs://node1:9000</value>
  </property>
```

### hdfs-site.xml

**hdfs-site.xml的也要三个节点都修改**

此配置文件来配置HDFS系统数据存储路径。

我选择把namenode的数据存储在`/opt/moudle/hadoop-3.1.3/hadoop_data/hdfs/namenode` ，

datanode的数据存储在`/opt/moudle/hadoop-3.1.3/hadoop_data/hdfs/datanode` 

**创建数据存储文件夹**

```
cd /opt/moudle/hadoop-3.1.3
mkdir -p ./hadoop_data/hdfs/namenode
mkdir -p ./hadoop_data/hdfs/datanode
```

**编辑hdfs-site.xml**

```
cd /opt/moudle/hadoop-3.1.3/etc/hadoop
vim hdfs-site.xml
```

将下面内容插入进configuration标签中

```
#副本数量设置为3   
<property>
                <name>dfs.replication</name>
                <value>3</value>
        </property>
#namenode数据存储文件夹
        <property>
                <name>dfs.namenode.name.dir</name>
                <value>/opt/moudle/hadoop-3.1.3/hadoop_data/hdfs/namenode</value>
        </property>
#datanode数据存储文件夹
        <property>
                <name>dfs.datanode.data.dir</name>
                <value>/opt/moudle/hadoop-3.1.3/hadoop_data/hdfs/datanode</value>
        </property>
```

### yarn-site.xml

**三个节点都要修改**

配置资源调度。

```
vim yarn-site.xml
```

将下面内容复制到configuration标签中。

```
<!-- 设置 ResourceManager 的主机名，通常是集群的主节点，主节点为node1 -->
  <property>
    <name>yarn.resourcemanager.hostname</name>
    <value>node1</value> <!-- 主节点 -->

  </property>

  <!-- 启用 NodeManager 的 shuffle 服务 -->
  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
  </property>

  <property>
    <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
    <value>org.apache.hadoop.mapred.ShuffleHandler</value>
  </property>

  <!-- 设置 YARN 的资源调度器 -->
  <property>
    <name>yarn.resourcemanager.scheduler.class</name>
    <value>org.apache.hadoop.yarn.server.resourcemanager.scheduler.capacity.CapacityScheduler</value>
  </property>

  <!-- 设置 ResourceManager 的 Web UI 端口 -->
  <property>
    <name>yarn.resourcemanager.webapp.address</name>
    <value>node1:8088</value>
  </property>
```

### mapred-site.xml

确保 HBase 与 Hadoop MapReduce 框架集成

```
vim mapred-site.xml
```

将下面内容复制到configuration标签中。

```
         <!-- 指定 MapReduce 框架为 YARN -->
  <property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
  </property>

  <!-- 设置 MapReduce Application Master 运行的队列 -->
  <property>
    <name>mapreduce.job.queuename</name>
    <value>default</value>
  </property>

  <!-- 设置 ResourceManager 的地址 -->
  <property>
    <name>yarn.app.mapreduce.am.resource.mb</name>
    <value>1024</value> <!-- Application Master 使用的内存大小（MB） -->
  </property>

  <!-- 启用 MapReduce shuffle 服务 -->
  <property>
    <name>mapreduce.jobhistory.address</name>
    <value>node1:10020</value>
  </property>

  <property>
    <name>mapreduce.jobhistory.webapp.address</name>
    <value>node1:19888</value>
  </property>
```

### workers

**只需要修改node1的workers即可**

修改此配置文件，告诉主节点(node1)哪些是从节点，配置完成后只需要在主节点启动hadoop那么hadoop就会自动ssh登录到从节点并启动hadoop服务。

```
vim workers
```

将node1到node3全部加入作为datanode

```
node1
node2
node3
```

## 启动Hadoop集群

### 在启动hadoop集群之前首先要把hadoop的namenode格式化

```
#格式化namenode
$HADOOP_HOME/bin/hdfs namenode -format
```

## 启动hdfs和yarn

```
#启动hdfs
$HADOOP_HOME/sbin/start-dfs.sh
#启动yarn
$HADOOP_HOME/sbin/start-yarn.sh
```

**使用jps查看当前进程**

分别在node1,node2,node3执行jps，看到下面内容则证明hadoop集群启动成功。

```
#node1执行jps
hbase@node1:/opt/moudle/hadoop-3.1.3$ jps
5009 ResourceManager
4787 SecondaryNameNode
4566 DataNode
4376 NameNode
5497 Jps
5178 NodeManager
#node2执行jps
hbase@node2:/opt/moudle/hadoop-3.1.3/etc/hadoop$ jps
2600 DataNode
2845 Jps
2749 NodeManager
#node3执行jps
hbase@node3:/opt/moudle/hadoop-3.1.3/etc/hadoop$ jps
2582 DataNode
2731 NodeManager
2828 Jps
```

# 配置Zookeeper集群

在 HBase 集群中，ZooKeeper 主要用于分布式协调和管理。

- **Master 和 RegionServer 的协调与选举**。
- **Region 的分配和自动恢复**。
- **确保分布式锁和一致性**。
- **集群发现和节点状态管理**

## **为每个节点创建** `myid` **文件**

在每个 Zookeeper 节点的 `dataDir` 目录下创建一个 `myid` 文件，文件中只包含一个数字，用于标识该节点的 ID。每个节点的 `myid` 文件内容应该不同,node1为 1 ，node2为2 ，node3为 3

```
echo "1" > /opt/zookeeper-3./zkData/myid  # 在 node1 上执行
echo "2" > /opt/zookeeper-3./zkData/myid  # 在 node2 上执行
echo "3" > /opt/zookeeper-3./zkData/myid  # 在 node3 上执行
```

## **zoo.cfg**

**此配置三个节点都要修改**

**修改此配置文件确保Zookeeper集群之间的通信。**

```
cd /opt/moudle/zookeeper-3.6.4/conf/
vim zoo.cfg
#添加以下内容
#定义zookeeper集群个节点的服务器和通信端口
server.1=node1:2888:3888
server.2=node2:2888:3888
server.3=node3:2888:3888
```

## **启动Zookeeper集群**

```
#zookeeper启动需要在每个节点上都启动,在三个节点上分别启动zookeeper
/opt/moudle/zookeeper-3.6.4/bin/zkServer.sh start
#检查zookeeper状态,每个node都执行
/opt/moudle/zookeeper-3.6.4/bin/zkServer.sh status
```

**查看集群状态出现一个leader和两个follower即为zookeeper集群启动成功**

```
#node1状态
hbase@node1:/opt/moudle/zookeeper-3.6.4/conf$ /opt/moudle/zookeeper-3.6.4/bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /opt/moudle/zookeeper-3.6.4/bin/../conf/zoo.cfg
Client port found: 2181. Client address: localhost. Client SSL: false.
Mode: follower
#node2状态
hbase@node2:/opt/moudle/zookeeper-3.6.4/conf$ /opt/moudle/zookeeper-3.6.4/bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /opt/moudle/zookeeper-3.6.4/bin/../conf/zoo.cfg
Client port found: 2181. Client address: localhost. Client SSL: false.
Mode: leader
#node3状态
hbase@node3:/opt/moudle/zookeeper-3.6.4/conf$ /opt/moudle/zookeeper-3.6.4/bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /opt/moudle/zookeeper-3.6.4/bin/../conf/zoo.cfg
Client port found: 2181. Client address: localhost. Client SSL: false.
Mode: follower
```

## **检查集群状态（可不做）**

可以通过 ZooKeeper 客户端命令连接到 ZooKeeper 集群，检查各节点是否正常工作：

```
/opt/moudle/zookeeper-3.6.4/bin/zkCli.sh -server node1:2181
```

进入 ZooKeeper 客户端后，可以执行 `stat` 命令检查集群状态：

```
stat /
```

# 配置HBase集群

配置HBase集群的前提是Hadoop集群和Zookeeper集群正常启动成功。

## 创建hbase存储数据目录

**此步骤只在node1上执行**

在hdfs文件系统中创建目录存储hbase数据

```
hdfs dfs -mkdir /hbase
hdfs dfs -chown hbase:hbase /hbase
```

## 创建hbase使用的临时目录

**此步骤三台机器都要执行**

```
mkdir /opt/moudle/hbase-2.4.5/tmp
```

## hbase-site.xml

修改此配置文件定义HBase 集群的行为和与其他组件（如 ZooKeeper 和 Hadoop）的集成方式。

**此配置文件三台虚拟机都需要配置**

```
cd cd /opt/moudle/hbase-2.4.5/conf/
vim hbase-site.xml
```

将hbase-site.xml中的property全部删除，下面内容全部复制进去。

```
    <!-- Zookeeper 节点列表 -->
    <property>
        <name>hbase.zookeeper.quorum</name>
        <value>node1,node2,node3</value>   
    </property>
    <!-- HBase 连接的 Hadoop 文件系统 -->
  <property>
    <name>hbase.rootdir</name>
    <value>hdfs://node1:9000/hbase</value> <!-- HDFS 上存储 HBase 数据的目录 -->
  </property>
  <!-- HBase Master Web UI 端口 -->
  <property>
    <name>hbase.master.port</name>
    <value>16000</value>
  </property>
   <!-- Zookeeper 的端口号 -->
  <property>
    <name>hbase.zookeeper.property.clientPort</name>
    <value>2181</value>
  </property>
<!--启用分布式zookeeper集群-->
  <property>
    <name>hbase.cluster.distributed</name>
    <value>true</value>
  </property>
<!--hbase内置了zookeeper,但我们要使用已经配置好的zookeeper集群，配置此项不会启动hbase自带的zookeeper-->
  <property>
    <name>hbase.zookeeper.useExternal</name>
    <value>true</value>
</property>
<!--hbase临时使用目录-->
  <property>
    <name>hbase.tmp.dir</name>
    <value>/opt/moudle/hbase-2.4.5/tmp</value>
  </property>

  <property>
    <name>hbase.unsafe.stream.capability.enforce</name>
    <value>false</value>
  </property>
```

## regionservers

**此文件（**`regionservers`）用于配置 HBase 集群的 RegionServer 节点列表。在默认情况下，HBase 集群的 Master 节点是启动 HBase 集群的节点。

我选择 `node1` 作为 HBase 的 Master 节点，同时将 `node1`、`node2` 和 `node3` 都配置为 RegionServer 节点。因此，整个 HBase 集群的架构如下：

- **Master 节点**：`node1` 负责管理集群的元数据、Region 分配、负载均衡等。
- **RegionServer 节点**：`node1`、`node2` 和 `node3` 都作为 RegionServer，用于处理数据存储和读写请求

**只在hbase的master节点配置即可**

```
vim regionservers 
```

写入以下内容

```
node1
node2
node3
```

## 启动Hbase集群

进入到node1的hbase安装目录并执行启动脚本

```
cd /opt/moudle/hbase-2.4.5
```

`bin/start-hbase.sh`
进入hbase shell 



```
bin/hbase shell

status
#看到下面，没有error就行
1 active master, 0 backup masters, 3 servers, 0 dead, 0.6667 average load
```

## 测试

```
                                                                                                                                                            hbase:002:0> create 'test','cf'
Created table test
Took 0.7585 seconds                                                                                                                                                                             
=> Hbase::Table - test
hbase:003:0> put 'test', 'row1', 'cf:a', 'value1'
Took 0.1894 seconds                                                                                                                                                                             
hbase:004:0> get 'test', 'row1'
COLUMN                                            CELL                                                                                                                                          
 cf:a                                             timestamp=2024-09-13T06:26:06.503, value=value1                                                                                               
1 row(s)
Took 0.0602 seconds                                                                                                                                                                             
hbase:005:0> scan 'test'
ROW                                               COLUMN+CELL                                                                                                                                   
 row1                                             column=cf:a, timestamp=2024-09-13T06:26:06.503, value=value1                                                                                  
1 row(s)
Took 0.0153 seconds                                                                                                                                                                             
hbase:006:0> delete 'test', 'row1', 'cf:a'
Took 0.0211 seconds                                                                                                                                                                             
hbase:007:0> disable 'test'
Took 0.6791 seconds                                                                                                                                                                             
hbase:008:0> drop 'test'
Took 0.3612 seconds                                                                                                                                                                             
hbase:009:0> list
TABLE                                                                                                                                                                                           
0 row(s)
Took 0.0241 seconds                                                                                                                                                                             
=> []
```