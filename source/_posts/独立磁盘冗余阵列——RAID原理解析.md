---
title: 独立磁盘冗余阵列——RAID原理解析
date: 2017-08-03 15:21:52
tags:
categories: 计算机基础
description:
---




## 基本概念

### 何为RAID
Raid 是一系列放在一起，成为一个逻辑卷的磁盘集合。

>磁盘阵列是由很多价格较便宜的磁盘，组合成一个容量巨大的磁盘组，利用个别磁盘提供数据所产生加成效果提升整个磁盘系统效能。利用这项技术，将数据切割成许多区段，分别存放在各个硬盘上。

*来自百度百科*

基本思想就是把多个相对便宜的硬盘组合起来，成为一个硬盘阵列组，使性能达到甚至超过一个价格昂贵、 容量巨大的硬盘。
RAID通常被用在服务器电脑上，使用完全相同的硬盘组成一个逻辑扇区，因此操作系统只会把它当做一个硬盘。

其中RAID又分为几个级别（RAID0、RAID1、RAID5等），各个不同的等级均在数据可靠性及读写性能上做了不同的权衡，需要根据实际情况去选择。

<!--more-->

下图虽然有些恶搞，但是形象的说明了RAID的原理和之间的区别，现在看不懂没关系，看完文章再回头看，应该很容易明白。

![Markdown](http://i2.tiimg.com/588729/d60b091d7f59ba1f.png)



















### RAID中一些重要概念

重要的 RAID 概念

**校验方式**——用在 RAID 重建中从校验所保存的信息中重新生成丢失的内容。 RAID 5，RAID 6 基于校验。
**条带化**——是将切片数据随机存储到多个磁盘。它不会在单个磁盘中保存完整的数据。如果我们使用2个磁盘，则每个磁盘存储我们的一半数据。
**镜像**——被用于 RAID 1 和 RAID 10。镜像会自动备份数据。在 RAID 1 中，它会保存相同的内容到其他盘上。
**热备份**——只是我们的服务器上的一个备用驱动器，它可以自动更换发生故障的驱动器。在我们的阵列中，如果任何一个驱动器损坏，热备份驱动器会自动用于重建 RAID。
**块**——是 RAID 控制器每次读写数据时的最小单位，最小 4KB。通过定义块大小，我们可以增加 I/O 性能。

### RAID级别
RAID有不同的级别。在这里，我们仅列出在真实环境下的使用最多的 RAID 级别。

- **RAID0** 
条带化
- **RAID1** 
镜像
- **RAID5** 
单磁盘分布式奇偶校验
- **RAID6** 
双磁盘分布式奇偶校验
- **RAID10** 
镜像 + 条带。（嵌套RAID）


## 起源

**该技术诞生的起因就是为了提高机械磁盘的读写速度（跟上CPU），与控制成本（只需要廉价磁盘）。**

由加利福尼亚大学伯克利分校（University of California-Berkeley）在1988年，发表的文章：“A Case for Redundant Arrays of Inexpensive Disks”。文章中，谈到了RAID这个词汇，而且定义了RAID的5层级。伯克利大学研究目的是反应当时CPU快速的性能。CPU效能每年大约成长30～50%，而硬磁机只能成长约7%。研究小组希望能找出一种新的技术，在短期内，立即提升效能来平衡计算机的运算能力。在当时，柏克莱研究小组的主要研究目的是效能与成本。

另外，研究小组也设计出容错（fault-tolerance），逻辑数据备份（logical data redundancy），而产生了RAID理论。研究初期，便宜（Inexpensive）的磁盘也是主要的重点，但后来发现，大量便宜磁盘组合并不能适用于现实的生产环境，后来Inexpensive被改为independent，许多独立的磁盘组。


## RAID0

RAID0称为条带化(Striping)存储，将数据分段存储于各个磁盘中，读写均可以并行处理。

Raid0是**所有RAID中存储性能最强的阵列形式**。其工作原理就是在多个磁盘上分散存取连续的数据，这样，当需要存取数据是多个磁盘可以并排执行，每个磁盘执行属于它自己的那部分数据请求，显著提高磁盘整体存取性能。

但是**没有数据冗余，不具备容错能力**，适用于低成本、低可靠性的台式系统，磁盘的损坏会导致数据的不可修复。

### 工作原理

RAID0工作方式：
![RAID0](http://i2.tiimg.com/588729/3f91f1333eaf9dad.png)







如图所示，系统向 三个磁盘组成的逻辑硬盘(RADI 0 磁盘组)发出的I/O数据请求被转化为3项操作，其中的每一项操作都对应于一块物理硬盘。我们从图中可以清楚的看到通过建立RAID 0，原先顺序的数据请求被分散到所有的三块硬盘中同时执行。

### 实现方式

（1）RAID 0最简单方式
就是把x块同样的硬盘用硬件的形式**通过智能磁盘控制器或用操作系统中的磁盘驱动程序以软件的方式串联在一起**，形成一个独立的逻辑驱动器，容量是单独硬盘的 x倍,在电脑数据写时被依次写入到各磁盘 中，当一块磁盘的空间用尽时，数据就会被自动写入到下一块磁盘中，它的好处是可以增加磁盘的容量。
**速度与其中任何一块磁盘的速度相同**，如果其中的任何一块磁盘出现故障，整个系统将会受到破坏，可靠
性是单独使用一块硬盘的1/n。

（2）RAID 0的另一方式（常指的RAID 0就是指的这个）
是用n块硬盘选择合理的带区大小创建带区集，最好是**为每一块硬盘都配备一个专门的磁盘控制器**，在电脑数据读写时同时向n块磁盘读写数据，**速度提升n倍**。提高系统的性能。

### 适用场景

RAID 0具有的特点，使其特别适用于对性能要求较高，而对数据安全不太在乎的领域，如图形工作站等。对于个人用户，RAID 0也是提高硬盘存储性能的绝佳选择。


## RAID1

又称镜像盘，数据被同等地写入两个或多个磁盘中：把一个磁盘的数据镜像到另一个磁盘上，采用镜像容错来提高可靠性，**具有raid中最高的数据冗余能力**。存数据时会将数据同时写入镜像盘内，读取数据则只从工作盘读出。发生故障时，系统将从镜像盘读取数据，然后再恢复工作盘正确数据。**这种阵列方式可靠性极高，但是其容量会减去一半。**


### 特点
RAID1有以下特点：
- RAID 1的每一个磁盘都具有一个对应的镜像盘，任何时候数据都同步镜像，系统可以从一组镜像盘中的任何一个磁盘读取数据。
- 磁盘所能使用的空间只有磁盘容量总和的一半，系统成本高。
- 只要系统中任何一对镜像盘中至少有一块磁盘可以使用，甚至可以在一半数量的硬盘出现问题时系统都可以正常运行。
- 出现硬盘故障的RAID系统不再可靠，应当及时的更换损坏的硬盘，否则剩余的镜像盘也出现问题，那么整个系统就会崩溃。
- 更换新盘后原有数据会需要很长时间同步镜像，外界对数据的访问不会受到影响，只是这时整个系统的性能有所下降。
- RAID1 磁盘控制器的负载相当大，用多个磁盘控制器可以提高数据的安全性和可用性。




### 工作原理 

![RAID1](http://i2.tiimg.com/588729/15757713589c2325.png)






如图所 示：当读取数据时，系统先从RAID1的源盘读取数据，如果读取数据成功，则系统不去管备份盘上的数据；如果读取源盘数据失败，则系统自动转而读取备份盘 上的数据，不会造成用户工作任务的中断。当然，我们应当及时地更换损坏的硬盘并利用备份数据重新建立镜像，避免备份盘在发生损坏时，造成不可挽回 的数据损失。


### 适用场景

由于对存储的数据进行百分之百的备份，在所有RAID级别中，RAID 1提供最高的数据安全保障。同样，由于数据的百分之百备份，备份数据占了总存储空间的一半，因而磁盘空间利用率低，存储成本高。 Mirror虽不能提高存储性能，但由于其具有的高数据安全性，使其尤其适用于存放重要数据，如服务器、数据库存储、商业金融、档案管理等领域。


## RAID5

可以理解为是RAID 0和RAID 1的折衷方案，但没有完全使用RAID1镜像理念，而是**使用了“奇偶校验信息”来作为数据恢复的方式**。

RAID 5可以理解为是RAID 0和RAID 1的折衷方案。RAID 5可以为系统提供数据安全保障，但 保障程度要比镜像低而磁盘空间利用率要比镜像高。RAID 5具有和RAID 0相近似的数据读取 速度，只是因为多了一个奇偶校验信息，写入数据的速度相对单独写入一块硬盘的速度略慢。

### 工作原理

![RAID5](http://i2.tiimg.com/588729/0866e01175f5928f.png)

RAID5把数据和相对应的奇偶校验信息存储到组成RAID5的各个磁盘上，并且**奇偶校验信息和相对应的数据分别存储于不同的磁盘上，其中任意N-1块磁盘上都存储完整的数据**，也就是说有相当于一块磁盘容量的空间用于存储奇偶校验信息。因此当RAID5的一个磁盘发生损坏 后，不会影响数据的完整性，从而保证了数据安全。当损坏的磁盘被替换后，RAID还会自动 利用剩下奇偶校验信息去重建此磁盘上的数据，来保持RAID5的高可靠性。


## RAID01

就是RAID0和RAID1的结合。先做条带(0)，再做镜像(1)。

![RAID01](http://i2.tiimg.com/588729/609abbc80ff87c8f.png)


## RAID10

先做镜像(1)，再做条带(0)

![RAID10](http://i2.tiimg.com/588729/9d4178efb835f5ee.png)

RAID01和RAID10非常相似，二者在读写性能上没有什么差别。但是在安全性上RAID10要好于 RAID01。如图中所示，假设DISK0损坏，在RAID10中，在剩下的3块盘中，只有当DISK1故障， 整个RAID才会失效。但在RAID01中，DISK0损坏后，左边的条带将无法读取，在剩下的3快盘 中，只要DISK2或DISK3两个盘中任何一个损坏，都会导致RAID失效。

## 参考

RAID技术介绍和总结
http://blog.jobbole.com/83808/

RAID详解[RAID0/RAID1/RAID10/RAID5]
http://blog.chinaunix.net/uid-639516-id-2692517.html


<!--more-->