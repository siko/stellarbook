独立部署安装的必要性？
====================

1.  从Stellar网络中及时获取最新的可靠数据；
2.  获取原始元数据；
3.  提交Transaction到网络中无需依赖第三方；
4.  灵活控制；
5.  参与Stellar网络共识，增加整个网络的健壮性；

节点类型
--------
```
| watcher                                            | archiver      | basic validator**                   | full validator                                                                                                  |                                      |
|----------------------------------------------------|---------------|-------------------------------------|-----------------------------------------------------------------------------------------------------------------------------|--------------------------------------|
| description                                        | non-validator | all of watcher + publish to archive | all of watcher + active participation in consensus (submit proposals for the transaction set to include in the next ledger) | basic validator + publish to archive |
| submits transactions                               | yes           | yes                                 | yes                                                                                                                         | yes                                  |
| supports horizon                                   | yes           | yes                                 | yes                                                                                                                         | yes                                  |
| participates in consensus                          | no            | no                                  | yes                                                                                                                         | yes                                  |
| helps other nodes to catch up and join the network | no            | yes                                 | no                                                                                                                          | yes                                  |
```

官方公布的各类节点
------------

*https://github.com/stellar/docs/blob/master/validators.md*

编译安装
============

### 1. 环境描述

+ Ubuntu14.04 64位
+ PostgreSQL9.4

### 2. 安装基础支撑环境

#### 安装add-apt-repository

`apt-get install python-software-properties software-properties-common`

#### 安装配套软件

`add-apt-repository ppa:ubuntu-toolchain-r/test`

_说明：需要连接国外源，可能会会比较慢_

```
apt-get update

apt-get install git

apt-get install build-essential

apt-get install pkg-config

apt-get install autoconf

apt-get install automake

apt-get install libtool

apt-get install bison

apt-get install flex

apt-get install libpq-dev

apt-get install clang++-3.5

apt-get install gcc-4.9

apt-get install g++-4.9

apt-get install cpp-4.9
```

_说明：查看安装版本：dpkg -l \| grep XXX_

#### 安装Stellar-Core

```
git clone https://github.com/stellar/stellar-core.git

cd stellar-core

git submodule init

git submodule update

切换分支：git checkout v0.5.1

./autogen.sh
```


1. 安装到默认路径
```
配置：CXX=g++-4.9 ./configure

构建：make（ or make -j (并行构建)）

单元测试执行：make check

安装：make install
```
2. 安装到指定路径
```
CXX=g++-4.9 ./configure --prefix=/目录/stellar-core-051

make

make check

make install
```

配置
========


添加到已有网络
--------------

【生成节点Seed】

\$stellar-core --genseed

【初始化数据库】

\$ stellar-core –newdb

【启动服务】默认配置文件./stellar-core.cfg，如果使用不同的配置文件使用如下命令：\$
stellar-core --conf betterfile.cfg

\$ stellar-core

开始一个新的网络
----------------

【生成节点Seed】

\$stellar-core --genseed

【初始化历史归档】避免多个节点put到一个历史归档。

\$ stellar-core --newhist local 例：sdf1

【初始化数据库】

\$ stellar-core –newdb

【立刻SCP】

\$ stellar-core --forcescp  启动一个网络，首次发起Transaction共识

【启动服务】默认配置文件./stellar-core.cfg，如果使用不同的配置文件使用如下命令：\$
stellar-core --conf betterfile.cfg

\$ stellar-core

命令控制
========

说明
----

默认情况下在本机（localhost）访问端口11626，可以通过浏览器、curl、命令行（需要添加--c）。

参考地址：

*https://www.stellar.org/developers/stellar-core/software/commands.html*

获取历史归档catchup
-------

触发当前实例获取账本信息，格式如下：

/catchup?ledger=NNN[&mode=MODE]

MODE为minimal（默认）或者complete

NNN为截止账本编号

日志等级控制
------------

使用Admin命令可以配置日志等级：\$ stellar-core -c "ll?level=debug"

也可以按照分区进行设置日志等级：

\$ stellar-core -c "ll?level=debug&partition=history"

在BUCKET\_DIR\_PATH里有状态锁文件，在重启前需要将此文件删除。

实例健康检查
------------
```
\$ stellar-core --c 'info'

返回如下所示。

"info" : {

"build" : "v0.2.3-9-g73147b7",

"ledger" : {

"age" : 6, 应该\<=10S，什么时候最后一个账页关闭。

"closeTime" : 1446178539,

"hash" : "f3c3424b85c004ebea1ae25991cf2ff902b46a5fea3bce1850c032118cd4567c",

"num" : 474367

},

"network" : "Public Global Stellar Network ; September 2015", 属于哪个网络

"numPeers" : 12, 连接的Peer数量。

"protocol\_version" : 1,

"quorum" : { 当前节点权重汇总信息

"474366" : {

"agree" : 5,

"disagree" : 0,

"fail\_at" : 2,

"hash" : "ac8c66",

"missing" : 0,

"phase" : "EXTERNALIZE"

}

},

"state" : "Synced!"

一个实例：

2016-11-18T16:14:30.853 GB3H4 [default INFO ] {

"info" : {

"build" : "v0.5.0-83-g4a8323d",

"ledger" : {

"age" : 1479456870,

"closeTime" : 0,

"hash" : "39c2a3cd4141b2853e70d84601faa44744660334b48f3228e0309342e3f4eb48",

"num" : 2

},

"network" : "Public Global Stellar Network ; September 2015",

"numPeers" : 5,

"protocol\_version" : 2,

"state" : "Joining SCP"

}

}
```

权重健康检查
------------
```
\$ stellar-core --c 'quorum'

返回如下所示。

"474313" : {

"agree" : 6, 在quorum里面设置的同意节点数量

"disagree" : null, 在quorum里面设置的不同意节点数量

"fail\_at" : 2, 能导致共识失败的失败节点数量；

"fail\_with" : [ "lab1", "lab2" ],

"hash" : "d1dacb", 当前节点Hash

"missing" : [ "donovan" ], 在共识节点列表中，挂掉节点列表

"phase" : "EXTERNALIZE",

"value" : { 当前节点设置的quorum所有节点

"t" : 5,

"v" : [ "lab1", "lab2", "lab3", "donovan", "GDVFV", "nelisky1", "nelisky2" ]

}
```

节点维护
========

维护步骤
--------
当维护几点时的几点建议：

1.  告知信任该节点的其他人；

2.  检查节点状态；

3.  如果没有人信任该节点，关机；

4.  重启；

5.  检查重启后的状态，Synced；

当前启动后，需要5到6分钟的同步时间。

判断一个验证节点是否正常
------------------------

\$ stellar-core --c 'quorum?node=\$sdf1

\$ stellar-core --c 'quorum?node=\@GABCDE

配置细节描述
============

数据存储
--------

### Ledger状态存储

Stellar-Core存储Ledger状态数据在SQL数据库中，可以是SQLite也可是PostgreSQL数据库。

运行一个新的Stellar-Core，必须执行创建数据库：

\$ stellar-core –newdb

### XDR存储

XDR文件存储在buckets中，需要快速硬盘并且要足够大的容量

### 历史归档History Archives

Stellar-Core通常需要一个或者多个历史归档，主要用于存储和读取历史数据，包括了两部分数据：Bucket
Files和历史Log。可以存储在Amazon S3、Google Cloud Storage、Azure Blob
Storage或者SCP/SFTP/HTTP服务器。

配置方式如下所示。
```
[HISTORY.local]

get="cp /tmp/stellar-core/history/vs/{0} {1}"

put="cp {0} /tmp/stellar-core/history/vs/{1}"

mkdir="mkdir -p /tmp/stellar-core/history/vs/{0}"
```
可以配置一个只读的源，至少需要配置get。

**在put中，启动前必须执行\$ stellar-core --newhist \<historyarchive\>，例如：**

**\$ stellar-core --newhist local**

其他实例：

\# other examples:

\# [HISTORY.stellar]

\# get="curl http://history.stellar.org/{0} -o {1}"

\# put="aws s3 cp {0} s3://history.stellar.org/{1}"

\# [HISTORY.backup]

\# get="curl http://backupstore.blob.core.windows.net/backupstore/{0} -o {1}"

\# put="azure storage blob upload {0} backupstore {1}"

\#The history store of the Stellar testnet

\#[HISTORY.h1]

\#get="curl -sf
http://s3-eu-west-1.amazonaws.com/history.stellar.org/prd/core-testnet/core\_testnet\_001/{0}
-o {1}"

\#[HISTORY.h2]

\#get="curl -sf
http://s3-eu-west-1.amazonaws.com/history.stellar.org/prd/core-testnet/core\_testnet\_002/{0}
-o {1}"

\#[HISTORY.h3]

\#get="curl -sf
http://s3-eu-west-1.amazonaws.com/history.stellar.org/prd/core-testnet/core\_testnet\_003/{0}
-o {1}"

注意事项：

-   put和mkdir必须配置，put不会自动创建子文件夹；

-   不支持不同节点写入同一个档案；

-   当已经存在一个存档，不要运行newhist，否则数据将被擦除；

权重配置quorum
--------------

### 阈值计算方法

### 设计原则

平衡安全和效率性，阈值太小容易导致少数说算。

拜占庭容错，错误节点数量f必须满足n\>=3f+1

例如：当有四个节点，至多能挂掉1个；

问题解决
========

配置文件解析错误
----------------

问题：

\<startup\> [default FATAL] Got an exception: Failed to parse 'stellar-core.cfg'
:Value must follow after a '=' at line 6 [main.cpp:564]

root\@iZ2zeficl3p7xdce7g3lnrZ:/letv/stellar-core-050/bin\# stellar-core

\---did you forget a '\#'? at line 1 [main.cpp:564] Got an excepti

解决方案：此类问题都是文件中存在不可见字符。文件中多了换行符（空行），导致stellar加载配置文件失败。


Peer节点链接不上
----------------

启动Stellar-Core后，链接Peer链接失败。

当连接上后，stellar数据库表中的peers显示所有peers，不要轻易删除此表。

浏览器执行命令
--------------

http://IP地址:11626/

退出shell后程序自动关闭
-----------------------

使用tmux多窗口命令，新建一个窗口启动stellar-core，可以避免此种问题。

使用nohup命令，启动后台服务。

需要进行SCP共识后才能生效的参数
---------------------------------

[QUORUM\_SET]

DESIRED\_MAX\_TX\_PER\_LEDGER

DESIRED\_BASE\_FEE

三个值需要通过全网SCP共识才能生效。

Docker方式运行
--------------

Supervisord进行进程管理进程软件，生产环境应该部署。用于监控服务。

针对分区（partition）进行日志区分
---------------------------------

分析例如历史分区partition，只记录历史信息，SCP等其他的日志不记录。

\$ stellar-core -c "ll?level=debug&partition=history"

日志等级可用配置项控制，使用 -ll
命令行标志或者通过管理（HTTP）命令动态调整。对正执行对系统执行：

\$ stellar-core -c "ll?level=debug"

日志级别也可以通过管理接口在逐个分区（partition）的基础上进行调整。
例如，可以通过运行以下命令将正在运行的系统，历史记录系统设置为DEBUG级：

\$ stellar-core -c "ll?level=debug&partition=history"

默认等级为 INFO，它适度详细，并且应该在正常操作下每隔几秒钟发出进度消息。

Stellar-core退出
----------------

Stellar-core可以随时通过发送 SIGINT 信号或按CTRL-C退出。 它可以安全地，使SIGTERM
SIGKILL强制终止也是安全的。但后者可能会在BUCKET\_DIR\_PATH中留下过时的锁定文件，您可能需要在重新启动之前删除该文件。
否则，所有组件设计为从突然终止恢复。

Stellar-core也可以被打包到一个容器系统如Docker中，只要BUCKET\_DIR\_PATH，TMP\_DIR\_PATH和数据库都存储在持久卷上。
有关示例，请参阅 docker-stellar-core.

备注：BUCKET\_DIR\_PATH 和 TMP\_DIR\_PATH 必须 在同一卷中。因为 stellar-core
需要在两者间重命名文件。

历史归档公开
------------

使用本地磁盘进行历史归档存储，本地stellar-core进行写入，通过nginx方式对外提供读权限（访问服务）。

第一次启动SCP共识
-----------------

\$ stellar-core --forcescp

此设置将强制每个节点立即开始SCP，而不是等待从网络获取。

第一个SCP共识。
