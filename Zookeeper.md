# Zookeeper

[TOC]

## <u>文档标识</u>

| 文档名称 | Zookeeper |
| -------- | --------- |
| 版本号   | <V1.0.0>  |

## <u>文档修订历史</u>

| 版本   | 日期       | 描述   | 文档所有者 |
| ------ | ---------- | ------ | ---------- |
| V1.0.0 | 2022.12.29 | create | 杨丝雨     |
|        |            |        |            |
|        |            |        |            |

## <u>目录结构</u>

| 目录名     | 作用             |
| ---------- | ---------------- |
| bin        | 存放系统脚本     |
| conf       | 存放配置文件     |
| contrib    | zk附加功能支持   |
| dist-maven | maven仓库文件    |
| docs       | zk文档           |
| lib        | 依赖的第三方库   |
| recipes    | 经典场景样例代码 |
| src        | zk源码           |

## <u>端口说明</u>

| 端口 | 作用                                   | remarks |
| ---- | -------------------------------------- | ------- |
| 2181 | 对cline端提供服务                      |         |
| 3888 | 选举leader使用                         |         |
| 2888 | 集群内机器通讯使用（Leader监听此端口） |         |

## <u>相关文档参考</u>

[Zookeeper官网]: https://zookeeper.apache.org/
[Zookeeper文档]: https://juejin.cn/post/6911981919974457358
[Zookeeper官方下载地址]: https://zookeeper.apache.org/releases.html



## Zookeeper介绍与使用场景

**官网概述：** Apache ZooKeeper致力于开发和维护可实现高度可靠的**分布式协调**的开源服务器。

> 分布式问题：
>
> 1.分布式协作算法很复杂，实现起来很困难；
>
> 2.分布式系统中更容易出现资源竞争或死锁现象； 
>
> 3.由应用实现分布式协作会导致部署上的困难；

### 概述

------

- Zookeeper是一个高性能、分布式的开源的协作服务；
- 提供一系列简单的功能，分布式应用可以在此基础上实现例如数据发布/订阅、负载均衡、命名服务、分布式协调/通知、集群管理、Leader选举、分布式锁和分布式队列等；

### 由来

------

Zookeeper最早起源于雅虎研究院的一个研究小组。在当时，研究人员发现，在雅虎内部很多大型系统基本都需要依赖一个类似的系统来进行分布式协调，但是这些系统往往都存在分布式单点问题。所以，雅虎的开发人员就试图开发一个通用的无单点问题的分布式协调框架，以便让开发人员将精力集中在处理业务逻辑上。

关于“ZooKeeper”这个项目的名字，其实也有一段趣闻。在立项初期，考虑到之前内部很多项目都是使用动物的名字来命名的（例如著名的Pig项目),雅虎的工程师希望给这个项目也取一个动物的名字。时任研究院的首席科学家RaghuRamakrishnan开玩笑地说：“在这样下去，我们这儿就变成动物园了！”此话一出，大家纷纷表示就叫动物园管理员吧一一一因为各个以动物命名的分布式组件放在一起，雅虎的整个分布式系统看上去就像一个大型的动物园了，而Zookeeper正好要用来进行分布式环境的协调一一于是，Zookeeper的名字也就由此诞生了。

### 特性

------

- 高可用性：能避免单点故障，最多可在n-1个节点故障状态下工作；
- 顺序一致性：来自客户端的更新将按照发送的顺序被写入到zookeeper中；
- 原子性：更新操作要么成功要么失败，没有中间状态；
- 单一系统映像：无论客户端连到哪一个 ZooKeeper 服务器上，客户端看到的服务端数据视图都不会是较旧的数据视图；
- 实时性：zookeeper保证客户端将在一个时间间隔范围内获得服务器的更新信息，或者服务器失效的信息。但由于网络延时等原因，zookeeper不能保证两个客户端能同时得到刚更新的数据，如果需要最新数据，最好在读数据之前调用sync()接口。

### 总体架构

------

Zookeeper服务可单机也可集群（2n+1个服务允许n个失效）部署。

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ee39fa49dafb4f8eb18ab5af691491ea~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image)

- 每个Server在内存中存储一份数据，并维护当前服务状态，服务与服务保持通信；
- Zookeeper启动时，将从实例中选举一个Leader（Paxos协议）；
- Leader提供写服务和读服务，Follower提供读服务；
- 集群间通过Zab（Zookeeper Atomic Broadcast）协议保证数据一致性；
- 一个更新操作成功，当且仅当大多数Server在内存中成功修改数据；

### 角色

------

Zookeeper引入了Leader、Follower 和 Observer 三种角色。Leader既提供写服务又能提供读服务。除了Leader外，Follower和Observer只能提供读服务。

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/41ece1f9a5c04992a800cb00eb12f527~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image)

### 数据模型

------

![img](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/26aaceac639743c9a8581786eaf11051~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image)

Zookeeper采用多层次化的目录结构，类似文件系统的数据结构。Zookeeper结构由节点组成，数据存储也基于节点，这种节点叫做ZNode。ZNode数据保存在内存中，意味着Zookeeper可以实现高吞吐量和低延迟。

![img](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2fef15c652244abb8cf290c98f1192c4~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image)

- Data：ZNode存储的数据信息。
- ACL：记录ZNode的访问权限，即哪些人或哪些IP可以访问本节点。
- Stat：包含ZNode的各种元数据，比如事务ID、版本号、时间戳、大小等等。
- Child：当前节点的子节点引用，类似于二叉树的左孩子右孩子。

## 安装

### 单机安装

------

> `Zookeeper` 需要 `java` 环境，本文不赘述 `java` 环境的部署。(这里不在叙述java环境部署，请自行执行安装java脚本)

#### 1、下载并解压安装包

- 进入 Zookeeper 官网下载：https://zookeeper.apache.org/releases.html 选择一个安装包下载。

解压安装包

```shell
wget https://dlcdn.apache.org/zookeeper/zookeeper-3.8.0/apache-zookeeper-3.8.0-bin.tar.gz
tar xf apache-zookeeper-3.8.0-bin.tar.gz
```

#### 2、配置

- `cd` 到 `zookeeper` 的根目录，创建 `data` 目录用于数据存储，创建 `logs` 目录用于存储日志。

```bash
cd apache-zookeeper-3.8.0-bin/
mkdir data
mkdir logs
```

- `cd` 到 `conf` 配置文件目录，拷贝 `zoo_sample.cfg` 为 `zoo.cfg`。

```bash
cd apache-zookeeper-3.8.0-bin/conf
# 配置文件拷贝
cp zoo_sample.cfg zoo.cfg
```

- 使用 `vim` 命令编辑 `zoo.cfg` 文件，修改 `dataDir` 的值为刚刚创建的 `data` 目录的绝对路径，添加 `dataLogDir` 日志目录指定。

```bash
dataDir=/root/apache-zookeeper-3.8.0-bin/data
dataLogDir=/root/apache-zookeeper-3.8.0-bin/logs
```

#### 3、运行

- 启动

```bash
$bin/zkServer.sh start
ZooKeeper JMX enabled by default
Using config: /root/apache-zookeeper-3.8.0-bin/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
```

- 查询状态

```bash
$bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /root/apache-zookeeper-3.8.0-bin/bin/../conf/zoo.cfg
Client port found: 2181. Client address: localhost. Client SSL: false.
Mode: standalone
```

- 停止

```bash
$bin/zkServer.sh stop
ZooKeeper JMX enabled by default
Using config: /root/apache-zookeeper-3.8.0-bin/bin/../conf/zoo.cfg
Stopping zookeeper ... STOPPED
```



### 伪集群安装

------

#### 1、新建数据储存目录，日志储存目录（本地演示没有创建是直接创建zk目录）

```bash
mkdir /root/apache-zookeeper-3.8.0-bin/zk{1..3}
ll ./
总用量 36
drwxr-xr-x 2 1000 ftp   4096 2月  25 2022 bin
drwxr-xr-x 2 1000 ftp    135 12月 29 10:41 conf
drwxr-xr-x 3 root root    23 12月 29 10:18 data
drwxr-xr-x 5 1000 ftp   4096 2月  25 2022 docs
drwxr-xr-x 2 root root  4096 12月 29 10:12 lib
-rw-r--r-- 1 1000 ftp  11358 2月  25 2022 LICENSE.txt
drwxr-xr-x 3 root root    78 12月 29 10:42 logs
-rw-r--r-- 1 1000 ftp   2084 2月  25 2022 NOTICE.txt
-rw-r--r-- 1 1000 ftp   2335 2月  25 2022 README.md
-rw-r--r-- 1 1000 ftp   3570 2月  25 2022 README_packaging.md
drwxr-xr-x 3 root root    63 12月 29 10:43 zk1
drwxr-xr-x 3 root root    63 12月 29 10:43 zk2
drwxr-xr-x 3 root root    63 12月 29 10:43 zk3
```

#### 2、创建集群配置文件

```bash
# zoo1.cfg
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/root/apache-zookeeper-3.8.0-bin/zk1
clientPort=2181

server.1=192.168.10.168:2888:3888
server.2=192.168.10.168:2889:3889
server.3=192.168.10.168:2890:3890

# zoo2.cfg
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/root/apache-zookeeper-3.8.0-bin/zk2
clientPort=2182

server.1=192.168.10.168:2888:3888
server.2=192.168.10.168:2889:3889
server.3=192.168.10.168:2890:3890

# zoo3.cfg
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/root/apache-zookeeper-3.8.0-bin/zk3
clientPort=2183

server.1=192.168.10.168:2888:3888
server.2=192.168.10.168:2889:3889
server.3=192.168.10.168:2890:3890
```

#### 3、创建myid文件

```bash
touch zk{1..3}/myid
echo "1" > zk1/myid
echo "2" > zk2/myid
echo "3" > zk3/myid
```

#### 4、启动集群

```bash
# zk1
bin/zkServer.sh start ./conf/zoo1.cfg
ZooKeeper JMX enabled by default
Using config: ./conf/zoo1.cfg
Starting zookeeper ... STARTED
# zk2
bin/zkServer.sh start ./conf/zoo2.cfg
ZooKeeper JMX enabled by default
Using config: ./conf/zoo2.cfg
Starting zookeeper ... STARTED
# zk3
bin/zkServer.sh start ./conf/zoo3.cfg
ZooKeeper JMX enabled by default
Using config: ./conf/zoo3.cfg
Starting zookeeper ... STARTED
```

#### 5、查看集群状态

```bash
# zk1
bin/zkServer.sh status ./conf/zoo1.cfg
ZooKeeper JMX enabled by default
Using config: ./conf/zoo1.cfg
Client port found: 2181. Client address: localhost. Client SSL: false.
Mode: follower
# zk2
bin/zkServer.sh status ./conf/zoo2.cfg
ZooKeeper JMX enabled by default
Using config: ./conf/zoo2.cfg
Client port found: 2182. Client address: localhost. Client SSL: false.
Mode: leader
# zk3
bin/zkServer.sh status ./conf/zoo3.cfg
ZooKeeper JMX enabled by default
Using config: ./conf/zoo3.cfg
Client port found: 2183. Client address: localhost. Client SSL: false.
Mode: follower
```



## <u>附录：zoo.cfg配置文件参数</u>

```bash
clientPort 
##客户端连接server的端口，即对外服务端口，一般设置为 2181。
 
dataDir    
##存储快照文件 snapshot 的目录。默认情况下，事务日志也会存储在这里。建议同时配置参数dataLogDir, 事务日志的写性能直接影响zk性能。
 
tickTime   
#ZK中的一个时间单元。ZK中所有时间都是以这个时间单元为基础，进行整数倍配置的。例如，session的最小超时时间是2*tickTime。
 
dataLogDir 
#事务日志输出目录。尽量给事务日志的输出配置单独的磁盘或是挂载点，这将极大的提升ZK性能。（No Java system property）
 
globalOutstandingLimit 
#最大请求堆积数。默认是1000。ZK运行的时候， 尽管server已经没有空闲来处理更多的客户端请求了，但是还是允许客户端将请求提交到服务器上来，以提高吞吐性能。
#当然，为了防止Server内存溢出，这个请求堆积数还是需要限制下的。(Java system property: zookeeper.globalOutstandingLimit.)
 
preAllocSize   
#预先开辟磁盘空间，用于后续写入事务日志。默认是64M，每个事务日志大小就是64M。如果ZK的快照频率较大的话，建议适当减小这个参数。(Java system property: zookeeper.preAllocSize)
 
snapCount  
#每进行 snapCount 次事务日志输出后，触发一次快照(snapshot), 此时，ZK会生成一个snapshot.文件，同时创建一个新的事务日志文件log.。
#默认是100000.（真正的代码实现中，会进行一定的随机数处理，以避免所有服务器在同一时间进行快照而影响性能）(Java system property: zookeeper.snapCount)
 
traceFile  
#用于记录所有请求的log，一般调试过程中可以使用，但是生产环境不建议使用，会严重影响性能。(Java system property:? requestTraceFile)
 
maxClientCnxns 
#单个客户端与单台服务器之间的连接数的限制，是ip级别的，默认是60，如果设置为0，那么表明不作任何限制。请注意这个限制的使用范围，仅仅是单台客户端机器与单台ZK服务器之间的连接数限制，不是针对指定客户端IP，也不是ZK集群的连接数限制，也不是单台ZK对所有客户端的连接数限制。
 
clientPortAddress  
#对于多网卡的机器，可以为每个IP指定不同的监听端口。默认情况是所有IP都监听 clientPort指定的端口。 New in 3.3.0
 
minSessionTimeoutmaxSessionTimeout 
#Session超时时间限制，如果客户端设置的超时时间不在这个范围，那么会被强制设置为最大或最小时间。默认的Session超时时间是在2 * tickTime ~ 20 * tickTime 这个范围 New in 3.3.0
 
fsync.warningthresholdms   
#事务日志输出时，如果调用fsync方法超过指定的超时时间，那么会在日志中输出警告信息。默认是1000ms。(Java system property: fsync.warningthresholdms**) **New in 3.3.4
 
autopurge.purgeInterval    
#3.4.0及之后版本，ZK提供了自动清理事务日志和快照文件的功能，这个参数指定了清理频率，单位是小时，需要配置一个1或更大的整数，默认是0，表示不开启自动清理功能。(No Java system property) New in 3.4.0
 
autopurge.snapRetainCount  
#这个参数和上面的参数搭配使用，这个参数指定了需要保留的文件数目。默认是保留3个。(No Java system property) New in 3.4.0
 
initLimit  
#Follower在启动过程中，会从Leader同步所有最新数据，然后确定自己能够对外服务的起始状态。Leader允许F在 initLimit时间内完成这个工作。通常情况下，我们不用太在意这个参数的设置。如果ZK集群的数据量确实很大了，F在启动的时候，从Leader上同步数据的时间也会相应变长，因此在这种情况下，有必要适当调大这个参数了。(No Java system property)
 
syncLimit  
#在运行过程中，Leader负责与ZK集群中所有机器进行通信，例如通过一些心跳检测机制，来检测机器的存活状态。如果L发出心跳包在syncLimit之后，还没有从F那里收到响应，那么就认为这个F已经不在线了。注意：不要把这个参数设置得过大，否则可能会掩盖一些问题。
 
leaderServes   
#默认情况下，Leader是会接受客户端连接，并提供正常的读写服务。但是，如果你想让Leader专注于集群中机器的协调，那么可以将这个参数设置为no，这样一来，会大大提高写操作的性能。(Java system property: zookeeper. leaderServes)。
 
server.x=[hostname]:nnnnn[:nnnnn]  
#这里的x是一个数字，与myid文件中的id是一致的。右边可以配置两个端口，第一个端口用于F和L之间的数据同步和其它通信，第二个端口用于Leader选举过程中投票通信。
 
group.x=nnnnn[:nnnnn]weight.x=nnnnn    
#对机器分组和权重设置，可以  参见这里(No Java system property)
 
cnxTimeout 
#Leader选举过程中，打开一次连接的超时时间，默认是5s。(Java system property: zookeeper. cnxTimeout)
 
zookeeper.DigestAuthenticationProvider.superDigest 
#ZK权限设置相关，具体参见  《  使用super  身份对有权限的节点进行操作 》  和  《 ZooKeeper   权限控制 》
 
skipACL    
#对所有客户端请求都不作ACL检查。如果之前节点上设置有权限限制，一旦服务器上打开这个开头，那么也将失效。(Java system property: zookeeper.skipACL)
 
forceSync  
#这个参数确定了是否需要在事务日志提交的时候调用 FileChannel .force来保证数据完全同步到磁盘。(Java system property: zookeeper.forceSync )
 
jute.maxbuffer
#每个节点最大数据量，是默认是1M。这个限制必须在server和client端都进行设置才会生效。(Java system property: jute.maxbuffer )
```

