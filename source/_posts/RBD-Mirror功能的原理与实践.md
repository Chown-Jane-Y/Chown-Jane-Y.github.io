---
title: RBD-Mirror功能的原理与实践
date: 2017-07-21 16:08:15
tags: RBD-Mirror
categories: Ceph
description:
---






## Ceph的组成

![ceph-component](http://i1.buimg.com/588729/fab1fdcf433af561.png)








<!-- more -->

## 为什么要rbd-mirror


Ceph采用的是**强一致性同步模型**，所有副本都必须完成写操作才算一次写入成功，一个请求需要异地返回再确认完成，如果副本在异地，网络延迟就会很大，拖垮整个集群的写性能。因此，Ceph集群很少有跨域部署的，也就缺乏异地容灾。

于是从2015年增加了一个新功能——RBD Mirror，用以解决两个集群间异步备份的问题。



## rbd-mirror设计原则

1. Replication needs to be asynchronous
- IO shouldn't be blocked due to slow WAN
- Support transient connectivity issues
2. Replication needs to be crash consistent
- Respect write barriers
3. Easy management
4. Expect failure can happen any time

>注：write barriers 是一种内核机制，用来确保文件系统metadata被正确并有序的写入持久化存储介质，不管电源是否突然关闭，都可以被确保。

>注：任何文件系统中的数据分为数据和元数据。数据是指普通文件中的实际数据，而元数据指用来描述一个文件的特征的系统数据，诸如访问权限、文件拥有者以及文件数据块的分布信息(inode...)等等。在集群文件系统中，分布信息包括文件在磁盘上的位置以及磁盘在集群中的位置。用户需要操作一个文件必须首先得到它的元数据，才能定位到文件的位置并且得到文件的内容或相关属性。

## rbd-mirror的原理

RBD Mirror原理其实**和MySQL的主从同步原理非常类似，前者基于journaling，后者基于binlog，简单地说就是利用日志进行回放(replay)**：通过在存储系统中增加Mirror组件，采用异步复制的方式，实现异地备份。(此处的journal是指Ceph RBD的journal，而不是OSD的journal)

该能力利用了 RBD image 的日志特性，以确保集群间的副本崩溃一致性。镜像功能需要在同伴集群（ peer clusters ）中的每一个对应的 pool 上进行配置，可设定自动备份某个存储池内的所有 images 或仅备份 images 的一个特定子集。 rbd-mirror 守护进程负责从远端集群拉取 image 的更新，并写入本地集群的对应 image 中。

当RBD Journal功能打开后，所有的数据更新请求会**先写入RBD Journal**，然后后台线程再把数据从Journal区域刷新到对应的image区域。RBD journal提供了比较完整的日志记录、读取、变更通知以及日志回收和空间释放等功能，可以认为是一个分布式的日志系统。




![principle](http://i1.buimg.com/588729/e703bff60b686f0a.png)















上图是**rbd-mirror的工作流程**：
```
1、当接收到一个写入请求后，I/O会先写入主集群的Image Journal

2、Journal写入成功后，通知客户端

3、客户端得到响应后，开始写入image

3、备份集群的mirror进程发现主集群的Journal有更新后，从主集群的Journal读取数据，写入备份集群（和上面序号一样，是因为这两个过程同时发生）

4、备份集群写入成功后，会更新主集群Journal中的元数据，表示该I/O的Journal已经同步完成

5、主集群会定期检查，删除已经写入备份集群的Journal数据。
```

以上就是一个rbd-mirror工作周期内的流程，在现有的Jewel版本中30s为一次工作周期，暂时不能改变这个周期时间。

## 优点

 1、当副本在异地的情况下，减少了单个集群不同节点间的数据写入延时；

2、减少本地集群或异地集群由于意外断电导致的数据丢失。

## 单向备份与双向备份

双向备份：两个集群之间互相同步，两个集群都要运行rbd-mirror进程。
单向备份：分为主集群和从集群，只在从集群运行rbd-mirror进程，主集群的修改会自动同步到从集群。

**在CSM中采用的是单向备份。**


## rbd-mirror的配置

### 安装须知

- RBD 镜像功能需要 Ceph Jewel 或更新的发行版本。
- 目前Jewel版本只支持一对一，不支持一对多。
- 两个集群 (local和remote) 需要能够互通。
- RBD需要开启journal特性， 启动后会记录image的事件。

### 1、安装rbd-mirror

`rbd-mirror`一个新的守护程序：它将会负责将一个镜像从一个集群同步到另一个，如果是双向备份，`rbd-mirror`需要在两个集群上都安装，单向备份的话只需要在从集群中安装即可，它会连接本地和远程的集群。（在jewel版本中还是一对一的方式，在以后的版本中会实现一对多的，所以在以后的版本可以配置一对多的备份）

rbd-mirror的每个实例必须能够同时连接到两个Ceph集群，因为需要同两个集群都进行数据通信。

**每个Ceph集群只运行一个rbd-mirror进程。**

配置yum源：
```xml
[rbd]
name=rbd
baseurl=http://mirrors.aliyun.com/ceph/rpm-jewel/el7/x86_64
enabled=1
gpgcheck=0
```

安装`rbd-mirror`：
```powershell
yum install -y rbd-mirror
```

### 2、拷贝配置文件到其他集群

`rbd-mirror`需要连接主集群和从集群，是通过集群的配置文件和密钥连接的。在ceph中，配置文件的名字和集群名是对应的，比方说名为`local`的集群，配置文件为`local.conf`。默认集群名为`ceph`，配置文件则为`ceph.conf`。密钥是用来验证是否有权力访问该集群，所以也是必须的。

连接`remote`和`local`两个集群，就是把两个集群的配置文件和密钥复制到对方的ceph配置文件目录中：

```powershell
scp local.conf local.client.admin.keyring remote1:/etc/ceph/
scp local.conf local.client.admin.keyring remote2:/etc/ceph/
scp local.conf local.client.admin.keyring remote3:/etc/ceph/
scp remote.conf remote.client.admin.keyring local1:/etc/ceph/
scp remote.conf remote.client.admin.keyring local1:/etc/ceph/
scp remote.conf remote.client.admin.keyring local1:/etc/ceph/
```
在所有节点上执行以下命令，更改权限：
```
chown ceph:ceph -R /etc/ceph
```
在两个集群上执行下面的命令：
```powershell
ceph --cluster=local mon stat
e1: 1 mons at {local1=192.10.10.170:6789/0}, election epoch 5, quorum 0 local1

ceph --cluster=remote mon stat
e1: 1 mons at {remote1=192.10.10.174:6789/0}, election epoch 3, quorum 0 remote1
```
如果能够互相查看集群的状态，则说明两个集群可以互相通信。




### 3、创建存储池

创建一个用于测试的存储池：
```powershell
ceph osd pool create rbdmirror 100 100 replicated --cluster=local
pool 'rbdmirror' created

ceph osd pool create rbdmirror 100 100 replicated --cluster=remote
pool 'rbdmirror' created
```


### 4、启用镜像功能

镜像功能分为两种模式：

- 存储池模式
一个存储池内的所有镜像都会进行备份

- 镜像模式
只有指定的镜像才会进行备份

这里使用的是存储池模式，开启存储池`rbdmirror`的镜像功能：
```powershell
rbd --cluster=local mirror pool enable rbdmirror pool
rbd --cluster=remote mirror pool enable rbdmirror pool
```


### 5、增加同伴集群

把`local`和`remote`设为同伴，这个是为了让rbd-mirror进程找到它peer的集群的存储池：

```powershell
rbd mirror pool peer add rbdmirror client.admin@remote --cluster=local
rbd mirror pool peer add rbdmirror client.admin@local --cluster=remote
```
查看peer的情况：
```cpp
rbd mirror pool info --pool=rbdmirror --cluster=local

[root@remote1 ~]# rbd mirror pool info --pool=mirror --cluster=local
Mode: pool
Peers: 
  UUID                                 NAME   CLIENT       
  f0929e85-259d-450b-917e-9eb231b7e43b remote client.admin 
 
[root@remote1 ~]# rbd mirror pool info --pool=mirror --cluster=remote
Mode: pool
Peers: 
  UUID                                 NAME  CLIENT       
  5851ba6a-e383-4ef0-9b9d-5ae34c9518a6 local client.admin 
```

### 6、主集群创建RBD

创建一个测试用的RBD：
```powershell
rbd create rbdmirror/testrbd --size=1024 --cluster=local
```

### 7、开启jounaling特性

 启动后会才会记录image的事件，才可以被rbd-mirror检测到并同步到从集群：
```powershell
rbd feature enable rbdmirror/testrbd journaling --cluster=local
```

### 8、查看同步状态

开启后，会开始自动同步到`remote`集群，可以用命令查看同步的状态。

pool正在同步的状态`syncing`：
```cpp
[root@remote1 ~]# rbd mirror pool status --pool=rbdmirror --cluster=remote
health: WARNING
images: 2 total
    1 syncing
    1 replaying
```
同步完成显示`replaying`：
```
[root@remote1 ~]# rbd mirror pool status --pool=rbdmirror --cluster=remote
health: OK
images: 2 total
    2 replaying
```
Image正在同步的状态``up+syncing`：
```
[root@remote1 ~]# rbd mirror image status rbdmirror/testrbd --cluster=remote
rbd: error opening image test3: (2) No such file or directory
[root@remote1 ~]# rbd mirror image status rbdmirror/testrbd --cluster=remote
test3:
  global_id:   fc35d898-24c3-40eb-9539-b5a559b8f8c5
  state:       up+syncing
  description: bootstrapping, OPEN_REMOTE_IMAGE
  last_update: 2017-07-19 16:22:55

```
同步完成`up+replaying`：
```cpp
[root@remote1 ~]# rbd mirror image status rbdmirror/testrbd --cluster=remote
test2:
  global_id:   9d376ad7-9ae9-4a50-b132-924a6063c67d
  state:       up+replaying
  description: replaying, master_position=[object_number=3, tag_tid=1, entry_tid=3], mirror_position=[], entries_behind_master=3
  last_update: 2017-07-19 16:21:31
```

9、查看image的信息

```cpp
[root@remote1 ~]# rbd info mirror/test1 --cluster=remote
rbd image 'testrbd':
	size 128 MB in 32 objects
	order 22 (4096 kB objects)
	block_name_prefix: rbd_data.575d12200854
	format: 2
	features: layering, exclusive-lock, object-map, fast-diff, deep-flatten, journaling
	flags: 
	journal: 575d12200854
	mirroring state: enabled
	mirroring global id: 9ae6a662-6d34-474b-bf71-624cd918cd1b
	mirroring primary: false
	
[root@remote1 ~]# rbd info mirror/test1 --cluster=local
rbd image 'testrbd':
	size 128 MB in 32 objects
	order 22 (4096 kB objects)
	block_name_prefix: rbd_data.94522ae8944a
	format: 2
	features: layering, exclusive-lock, object-map, fast-diff, deep-flatten, journaling
	flags: 
	journal: 94522ae8944a
	mirroring state: enabled
	mirroring global id: 9ae6a662-6d34-474b-bf71-624cd918cd1b
	mirroring primary: true
```
开启`mirror`功能的rbd信息会多几个属性，`mirroring state`为`enabled`说明开启了mirror功能，`mirroring primary`为`true`说明是主集群，`false`为从集群。

### 9、查看从集群是否同步

```powershell
rbd ls --pool rbdmirror --cluster=remote
testrbd
```
主集群的testrbd已经成功同步到从集群`remote`。


## 启动rbd-mirror时报错

### 1、failed to bind the UNIX domain socket
```
failed to bind the UNIX domain socket to '/var/run/ceph/local-client.admin.asok': (17) File exists
```

不明，但可以正常使用
### 2、error requesting lock
```
2017-06-27 09:43:59.135464 7fedb7eee700 -1 librbd::ImageWatcher: 0x7fedd800ca50 error requesting lock: (30) Read-only file system
```
原因：已经存在一个rbd-mirror进程了。


## 常用命令

### 删除pool
```powershell
ceph osd pool delete {pool-name} [{pool-name} --yes-i-really-really-mean-it]
```

<!-- more -->