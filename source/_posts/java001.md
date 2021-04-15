---
layout: post
title: 'Kafka初识'
categories: Java
date: 2021-01-01 00:00:01
tags:
    - Java
    - 框架学习
---
@[TOC](目录)
## 一、Kafka概念
### 1. 概念
* **1 定义**
Kafka是一个分布式的基于发布/订阅模式的消息队列（Message Queue），主要应用于大数据实时处理领域。
* 总结：
是一个分布式消息队列，流式计算中，一般用来缓存数据，具有统一、高吞吐、低等待的特性。
* 具体：
在流式计算中，Kafka一般用来缓存数据，Storm通过消费Kafka的数据进行计算。
1）Apache  Kafka是一个开源消息系统，由Scala写成。是由Apache软件基金会开发的一个开源消息系统项目。
2）Kafka最初是由LinkedIn公司开发，并于2011年初开源。2012年10月从Apache Incubator毕业。该项目的目标是为处理实时数据提供一个统一、高通量、低等待的平台。
3）Kafka是一个分布式消息队列。Kafka对消息保存时根据Topic进行归类，发送消息者称为Producer，消息接受者称为Consumer，此外kafka集群有多个kafka实例组成，每个实例(server)称为broker。
4）无论是kafka集群，还是consumer都依赖于zookeeper集群保存一些meta信息，来保证系统可用性。

### 2. Kafka架构
#### 1-基础架构
1）Producer ：消息生产者，就是向kafka broker发消息的客户端；
2）Consumer ：消息消费者，向kafka broker取消息的客户端；
3）Consumer Group （CG）：消费者组，由多个consumer组成。**消费者组内每个消费者负责消费不同分区的数据，一个分区只能由一个组内消费者消费；消费者组之间互不影响。**所有的消费者都属于某个消费者组，即**消费者组是逻辑上的一个订阅者**。
4）Broker ：一台kafka服务器就是一个broker。一个集群由多个broker组成。一个broker可以容纳多个topic。
5）Topic ：可以理解为一个队列，**生产者和消费者面向的都是一个topic**；
6）Partition：为了实现扩展性，一个非常大的topic可以分布到多个broker（即服务器）上，**一个topic可以分为多个partition**，每个partition是一个有序的队列；
7）Replica：副本，为保证集群中的某个节点发生故障时，**该节点上的partition数据不丢失，且kafka仍然能够继续工作**，kafka提供了副本机制，一个topic的每个分区都有若干个副本，一个leader和若干个follower。
8）leader：每个分区多个副本的“主”，生产者发送数据的对象，以及消费者消费数据的对象都是leader。
9）follower：每个分区多个副本中的“从”，实时从leader中同步数据，保持和leader数据的同步。leader发生故障时，某个follower会成为新的follower。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210327111441704.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NzM0NDAz,size_16,color_FFFFFF,t_70)

#### 2-工作流程及文件存储机制
###### 1 工作流程
-- Kafka中消息是以topic进行分类的，生产者生产消息，消费者消费消息，都是面向topic的。

tips：Kafka只能保证区内有序，而不能保证全局有序
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210327131042798.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NzM0NDAz,size_16,color_FFFFFF,t_70)
官网图：
![](https://img-blog.csdnimg.cn/20210327130558619.png)
###### 2 文件存储机制 -日志结构

-- topic是逻辑上的概念，而partition是物理上的概念，每个partition对应于一个log文件，该log文件中存储的就是producer生产的数据。Producer生产的数据会被不断追加到该log文件末端，且每条数据都有自己的offset。消费者组中的每个消费者，都会实时记录自己消费到了哪个offset，以便出错恢复时，从上次的位置继续消费。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210327132556852.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NzM0NDAz,size_16,color_FFFFFF,t_70)
-- 由于生产者生产的消息会不断追加到log文件末尾，为防止log文件过大导致数据定位效率低下，Kafka采取了**分片和索引机制**，将每个partition分为多个segment。每个segment对应两个文件——“.index”文件和“.log”文件。这些文件位于一个文件夹下，该文件夹的命名规则为：topic名称+分区序号。例如，first这个topic有三个分区，则其对应的文件夹为first-0,first-1,first-2。
```
00000000000000000000.index
00000000000000000000.log
00000000000000170410.index
00000000000000170410.log
00000000000000239430.index
00000000000000239430.log
```
index和log文件以当前segment的第一条消息的offset命名。下图为index文件和log文件的结构示意图:
**“.index”文件存储大量的索引信息，“.log”文件存储大量的数据**，索引文件中的元数据指向对应数据文件中message的物理偏移地址。
tips：索引信息还包含对应数据的大小、seed方法
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210327132948649.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NzM0NDAz,size_16,color_FFFFFF,t_70)
**补充：**
日志中包含多个日志段，而每个日志段又包含：消息日志文件、位移索引文件、时间戳索引文件、已终止的事务索引文件。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210408112934730.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NzM0NDAz,size_16,color_FFFFFF,t_70)


#### 3-生产者
##### 1> 分区策略
* **1）分区的原因**
（1）方便在集群中扩展，每个Partition可以通过调整以适应它所在的机器，而一个topic又可以有多个Partition组成，因此整个集群就可以适应任意大小的数据了；---注：即负载均衡
（2）可以提高并发，因为可以以Partition为单位读写了。
* **2）分区的原则**
我们需要将producer发送的数据封装成一个**ProducerRecord对象**。
（1）指明partition 的情况下，直接将指明的值直接作为partiton 值；
（2）没有指明partition 值但有key 的情况下，将key 的hash 值与topic 的partition 数进行取余得到partition 值；
（3）既没有partition 值又没有key 值的情况下，第一次调用时随机生成一个整数（后面每次调用在这个整数上自增），将这个值与topic 可用的partition 总数取余得到partition 值，也就是常说的round-robin 算法
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210327134938950.png)

##### 2> 数据可靠性保证---重复、一致
* **为保证producer发送的数据，能可靠的发送到指定的topic，topic的每个partition收到producer发送的数据后，都需要向producer发送ack（acknowledgement确认收到），如果producer收到ack，就会进行下一轮的发送，否则重新发送数据。**
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021032714065464.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NzM0NDAz,size_16,color_FFFFFF,t_70)
* **1）副本数据同步策略**
Kafka选择了第二种方案，原因如下：
1.同样为了容忍n台节点的故障，第一种方案需要2n+1个副本，而第二种方案只需要n+1个副本，而Kafka的每个分区都有大量的数据，第一种方案会造成大量数据的冗余。
2.虽然第二种方案的网络延迟会比较高，但网络延迟对Kafka的影响较小。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210327140820226.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NzM0NDAz,size_16,color_FFFFFF,t_70)
* **2）ISR**
-- 问题：
采用第二种方案之后，设想以下情景：leader收到数据，所有follower都开始同步数据，但有一个follower，因为某种故障，迟迟不能与leader进行同步，那leader就要一直等下去，直到它完成同步，才能发送ack。这个问题怎么解决呢？
-- 解决：
**Leader维护了一个动态的in-sync replica set (ISR)，意为和leader保持同步的follower集合。当ISR中的follower完成数据的同步之后，leader就会给follower发送ack。如果follower长时间未向leader同步数据，则该follower将被踢出ISR，该时间阈值由replica.lag.time.max.ms参数设定。Leader发生故障之后，就会从ISR中选举新的leader。**
tips：满足replica.lag.time.max.ms参数设置内时间，follower被加入ISR，ISR全部同步完，即完成，
0.9之前还有个同步条数参数，后被移除
ISR包含leader
10s
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210327140029551.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210327140112684.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NzM0NDAz,size_16,color_FFFFFF,t_70)
* **3）ack应答机制**
-- 不重要的数据：
对于某些不太重要的数据，对数据的可靠性要求不是很高，能够容忍数据的少量丢失，所以没必要等ISR中的follower（ISR）全部接收成功。
-- 三种可靠级别：
所以Kafka为用户提供了三种可靠性级别，用户根据对可靠性和延迟的要求进行权衡，选择以下的配置。
**acks参数配置**：
**acks：**
0：producer不等待broker的ack，这一操作提供了一个最低的延迟，broker一接收到还没有写入磁盘就已经返回，当broker故障时有可能**丢失数据**；
1：producer等待broker的ack，partition的leader落盘成功后返回ack，如果在follower同步成功之前leader故障，那么将会**丢失数据**；
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210327142750343.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NzM0NDAz,size_16,color_FFFFFF,t_70)
-1（all）：producer等待broker的ack，partition的leader和follower全部落盘成功后才返回ack。但是如果在follower同步完成后，broker发送ack之前，leader发生故障，那么会造成==数据重复==。
tips:leader保存数据后未发生ack挂掉，生产者没收到ack，向新leader重新发送，新leader重新保存数据。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210327142856385.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NzM0NDAz,size_16,color_FFFFFF,t_70)
* **4）故障处理细节**
**LEO：指的是每个副本最大的offset；**
**HW：指的是消费者能见到的最大的offset，ISR队列中最小的LEO。**
（1）follower故障
follower发生故障后会被临时踢出ISR，待该follower恢复后，follower会读取本地磁盘记录的上次的HW，并将log文件高于HW的部分截取掉，从HW开始向leader进行同步。等该follower的LEO大于等于该Partition的HW，即follower追上leader之后，就可以重新加入ISR了。
（2）leader故障
leader发生故障之后，会从ISR中选出一个新的leader，之后，为保证多个副本之间的数据一致性，其余的follower会先将各自的log文件高于HW的部分截掉，然后从新的leader同步数据。（注：多了会截取，少了会同步补上）
==注意：这只能保证副本之间的数据一致性，并不能保证数据不丢失或者不重复。==
tips：保证了消费一致性 存储一致性，ack 处理数据丢失和重复，此处的leader和follower都是ISR中的。
Log文件中的HW和LEO，如图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210327150604199.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NzM0NDAz,size_16,color_FFFFFF,t_70)

##### 3> Exactly Once语义---精准一次性
* **1 AtLeast Once语义：** 至少一次
将服务器的ACK级别设置为-1，可以保证Producer到Server之间不会丢失数据，即AtLeast Once语义。
* **2 AtMostOnce语义：**至多一次
相对的，将服务器ACK级别设置为0，可以保证生产者每条消息只会被发送一次，即AtMostOnce语义。
* **3 重复、丢失**
AtLeastOnce可以保证数据不丢失，但是不能保证数据不重复；相对的，AtLeastOnce可以保证数据不重复，但是不能保证数据不丢失。但是，**对于一些非常重要的信息，比如说交易数据，下游数据消费者要求数据既不重复也不丢失，即ExactlyOnce语义。**在0.11版本以前的Kafka，对此是无能为力的，只能保证数据不丢失，再在下游消费者对数据做全局去重。对于多个下游应用的情况，每个都需要单独做全局去重，这就对性能造成了很大影响。
* **4 幂等性**
0.11版本的Kafka，引入了一项重大特性：幂等性。
所谓的**幂等性就是指Producer不论向Server发送多少次重复数据，Server端都只会持久化一条**。幂等性结合AtLeastOnce语义，就构成了Kafka的ExactlyOnce语义。即：
AtLeastOnce+幂等性=ExactlyOnce
* **5 启用幂等性**
要启用幂等性，只需要将Producer的参数中enable.idompotence设置为true即可（注，即ack=-1）。
* **6 幂等实现**
Kafka的幂等性实现其实就是将原来下游需要做的去重放在了数据上游。开启幂等性的Producer在初始化的时候会被分配一个PID，发往同一Partition的消息会附带SequenceNumber。而**Broker端**会对<PID, Partition,SeqNumber>做缓存，当具有相同主键的消息提交时，Broker只会持久化一条。
但是PID重启就会变化，同时不同的Partition也具有不同主键，所以幂等性无法保证跨分区跨会话的ExactlyOnce。（注，重新建立会话，pid变化，重新发送幂等会失效）

#### 4-消费者
##### 1> 消费方式
* ==consumer采用pull（拉）模式从broker中读取数据。==
* **push（推）模式很难适应消费速率不同的消费者，因为消息发送速率是由broker决定的。**
它的目标是尽可能以最快速度传递消息，但是这样很容易造成consumer来不及处理消息，典型的表现就是拒绝服务以及网络拥塞。而pull模式则可以根据consumer的消费能力以适当的速率消费消息。
* **pull模式不足之处是，如果kafka没有数据，消费者可能会陷入循环中，一直返回空数据。**针对这一点，Kafka的消费者在消费数据时会传入一个时长参数timeout，如果当前没有数据可供消费，consumer会等待一段时间之后再返回，这段时长即为timeout。

##### 2> 分区分配策略
* **1 分配问题**
一个consumergroup中有多个consumer，一个topic有多个partition，所以必然会涉及到partition的分配问题，即确定那个partition由哪个consumer来消费。
* **2 Kafka有两种分配策略**
一是RoundRobin，一是Range（默认）。
（注，消费者增减需要重分配。RoundRobin直接看那个组订阅了它，组订阅了就把T1T2轮询给组，Range优先看消费者，然后再看消费者分组，/2分配给消费者组）
一个topic的消费，如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210327160307743.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NzM0NDAz,size_16,color_FFFFFF,t_70)
1）RoundRobin
好处：最多差一个
弊端：订阅主体一样才能使用
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210327160446451.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NzM0NDAz,size_10,color_FFFFFF,t_70)
多topic：
![在这里插入图片描述](https://img-blog.csdnimg.cn/202103271608335.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NzM0NDAz,size_16,color_FFFFFF,t_70)
问题：
![在这里插入图片描述](https://img-blog.csdnimg.cn/202103271612173.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NzM0NDAz,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210327161524488.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NzM0NDAz,size_16,color_FFFFFF,t_70)
2）Range
缺点：数据不均衡
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210327161730584.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NzM0NDAz,size_16,color_FFFFFF,t_70)
组        topic
轮询      面向主体  不均衡

##### 3> offset的维护
-- 由于consumer在消费过程中可能会出现断电宕机等故障，consumer恢复后，需要从故障前的位置的继续消费，所以consumer需要实时记录自己消费到了哪个offset，以便故障恢复后继续消费。
-- Kafka0.9版本之前，consumer默认将offset保存在Zookeeper中，从0.9版本开始，consumer默认将offset保存在Kafka一个内置的topic中，该topic为__consumer_offsets。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210327191241434.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NzM0NDAz,size_16,color_FFFFFF,t_70)
##### 4> 消费者组案例
1）需求：测试同一个消费者组中的消费者，同一时刻只能有一个消费者消费。

#### 5-Kafka 高效读写数据
1）顺序写磁盘
Kafka的producer生产数据，要写入到log文件中，写的过程是一直追加到文件末端，为顺序写。官网有数据表明，同样的磁盘，顺序写能到600M/s，而随机写只有100K/s。这与磁盘的机械机构有关，顺序写之所以快，是因为其省去了大量磁头寻址的时间。
2）零复制技术
正常io不包含中间那条线
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210327191625484.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NzM0NDAz,size_16,color_FFFFFF,t_70)
零拷贝：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210327192145969.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NzM0NDAz,size_16,color_FFFFFF,t_70)
#### 6-Zookeeper在Kafka中的作用
Kafka集群中有一个broker会被选举为Controller，负责**管理集群broker的上下线**，所有**topic的分区副本分配和leader选举等工作**。Controller的管理工作都是依赖于Zookeeper的。
tiips：controller选举：隔断时间看一下controller是否还在，先到先得 （Controller是kafka实例，leader是数据副本）
以下为partition的leader选举过程：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210327200206246.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NzM0NDAz,size_16,color_FFFFFF,t_70)

#### 7-Kafka事务
Kafka从0.11版本开始引入了事务支持。事务可以保证Kafka在ExactlyOnce语义的基础上，生产和消费可以跨分区和会话，要么全部成功，要么全部失败。
* **1 Producer事务**
-- 为了实现跨分区跨会话的事务，需要引入一个全局唯一的Transaction ID，并将Producer获得的PID和Transaction ID绑定。这样当Producer重启后就可以通过正在进行的Transaction ID获得原来的PID。
-- 为了管理Transaction，Kafka引入了一个新的组件Transaction Coordinator。Producer就是通过和TransactionCoordinator交互获得TransactionID对应的任务状态。Transaction Coordinator还负责将事务所有写入Kafka的一个内部Topic，这样即使整个服务重启，由于事务状态得到保存，进行中的事务状态可以得到恢复，从而继续进行。
tips：如，30个数据，3个broker每个10个数据，第3个broker故障，重复发送，1，2重复，3不重复。上述方法是，PID和客户端事务ID关联，获取到故障前的PID，幂等。

* **2 Consumer事务**
上述事务机制主要是从Producer方面考虑，对于Consumer而言，事务的保证就会相对较弱，尤其时无法保证Commit的信息被精确消费。这是由于Consumer可以通过offset访问任意信息，而且不同的SegmentFile生命周期不同，同一事务的消息可能会出现重启后被删除的情况。

## 二、Kafka API
### 1.  Producer API
#### 1-消息发送流程
-- Kafka的Producer发送消息采用的是**异步发送**的方式。在消息发送的过程中，涉及到了**两个线程——main线程和Sender线程，以及一个线程共享变量——RecordAccumulator**。main线程将消息发送给RecordAccumulator，Sender线程不断从RecordAccumulator中拉取消息发送到Kafkabroker。
--相关参数：
batch.size：只有数据积累到batch.size之后，sender才会发送数据。
linger.ms：如果数据迟迟未达到batch.size，sender等待linger.time之后就会发送数据
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210327212719353.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NzM0NDAz,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210328135809288.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NzM0NDAz,size_16,color_FFFFFF,t_70)
#### 2-异步发送API--producer接口
* **需要用到的类：**
-- KafkaProducer：需要创建一个生产者对象，用来发送数据
-- ProducerConfig：获取所需的一系列配置参数
-- ProducerRecord：每条数据都要封装成一个ProducerRecord对象

* **2种实现**
* **1）不带回调函数的API**

* **2）带回调函数的API**
回调函数会在producer收到ack时调用，为异步调用，该方法有两个参数，分别是RecordMetadata和Exception，如果Exception为null，说明消息发送成功，如果Exception不为null，说明消息发送失败。
**注意：消息发送失败会自动重试，不需要我们在回调函数中手动重试。**

#### 3 同步发送API--producer接口
同步发送的意思就是，一条消息发送之后，会阻塞当前线程，直至返回ack。由于send方法返回的是一个Future对象，根据Futrue对象的特点，我们也可以实现同步发送的效果，只需在调用Future对象的get方发即可。


### 2. Consumer API
* **可靠性有保证**
Consumer消费数据时的可靠性是很容易保证的，因为数据在Kafka中是持久化的，故不用担心数据丢失问题。
* **offset必须考虑**
由于consumer在消费过程中可能会出现断电宕机等故障，consumer恢复后，需要从故障前的位置的继续消费，所以consumer需要实时记录自己消费到了哪个offset，以便故障恢复后继续消费。
**所以offset的维护是Consumer消费数据是必须考虑的问题。**

 
 #### 1-自动提交offset--consumer接口
 
* 编写代码需要用到的类：
-- KafkaConsumer：需要创建一个消费者对象，用来消费数据
-- ConsumerConfig：获取所需的一系列配置参数
-- ConsuemrRecord：每条数据都要封装成一个ConsumerRecord对象
* 为了使我们能够专注于自己的业务逻辑，Kafka提供了自动提交offset的功能。
自动提交offset的相关参数：
**enable.auto.commit**：是否开启自动提交offset功能
**auto.commit.interval.ms**：自动提交offset的时间间隔以下为自动提交offset的代码：
* 代码如下：

#### 2-手动提交offset--consumer接口
* **1 手动 Why?**
虽然自动提交offset十分简介便利，但由于其是基于时间提交的，**开发人员难以把握offset提交的时机**。因此Kafka还提供了手动提交offset的API。
* **2 两种方法**
-- 手动提交offset的方法有两种：分别是**commitSync（同步提交）和commitAsync（异步提交）**。
-- 相同点：都会将本次poll的一批数据最高的偏移量提交；
-- 不同点：commitSync阻塞当前线程，一直到提交成功，并且会自动失败重试（由不可控因素导致，也会出现提交失败）；而commitAsync则没有失败重试机制，故有可能提交失败。
**1）同步提交**
offset由于同步提交offset有失败重试机制，故更加可靠，以下为同步提交offset的示例。

**2）异步提交offset**
虽然同步提交offset更可靠一些，但是由于其会阻塞当前线程，直到提交成功。因此吞吐量会收到很大的影响。因此更多的情况下，会选用异步提交offset的方式。
以下为异步提交offset的示例：

**3）漏和重复**
数据漏消费和重复消费分析无论是同步提交还是异步提交offset，都有可能会造成数据的漏消费或者重复消费。
**先提交offset后消费，有可能造成数据的漏消费；而先消费后提交offset，有可能会造成数据的重复消费。**


#### 3-自定义存储offset--consumer接口
* **1 自定义存储offset**
Kafka0.9版本之前，offset存储在zookeeper，0.9版本及之后，默认将offset存储在Kafka的一个内置的topic中。除此之外，Kafka还可以选择自定义存储offset。
* **2 消费者Rebalace**
-- offset的维护是相当繁琐的，因为需要考虑到消费者的Rebalace。
-- **当有新的消费者加入消费者组、已有的消费者推出消费者组或者所订阅的主题的分区发生变化，就会触发到分区的重新分配，重新分配的过程叫做Rebalance。**
--消费者发生Rebalance之后，每个消费者消费的分区就会发生变化。**因此消费者要首先获取到自己被重新分配到的分区，并且定位到每个分区最近提交的offset位置继续消费。**
* **3 实现Rebalace**
要实现自定义存储offset，需要借助**ConsumerRebalanceListener**，以下为示例代码，其中提交和获取offset的方法，需要根据所选的offset存储系统自行实现。

### 3. 自定义拦截器（Interceptor）
#### 1-拦截器原理
* **1 概念**
Producer拦截器(interceptor)是在Kafka 0.10版本被引入的，主要**用于实现clients端的定制化控制逻辑**。
* **2 原理**
对于producer而言，interceptor使得用户在消息发送前以及producer回调逻辑前有机会对消息做一些定制化需求，比如修改消息等。同时，producer允许用户指定多个interceptor按序作用于同一条消息从而形成一个拦截链(interceptor chain)。
Intercetpor的实现接口是org.apache.kafka.clients.producer.ProducerInterceptor，其定义的方法包括：
**（1）configure(configs)**
获取配置信息和初始化数据时调用。
**（2）onSend(ProducerRecord)**
该方法封装进KafkaProducer.send方法中，即它运行在用户主线程中。Producer确保在消息被序列化以及计算分区前调用该方法。**用户可以在该方法中对消息做任何操作，但最好保证不要修改消息所属的topic和分区**，否则会影响目标分区的计算。
**（3）onAcknowledgement(RecordMetadata, Exception)**
**该方法会在消息从RecordAccumulator成功发送到KafkaBroker之后，或者在发送过程中失败时调用。**并且通常都是在producer回调逻辑触发之前。onAcknowledgement运行在producer的IO线程中，因此不要在该方法中放入很重的逻辑，否则会拖慢producer的消息发送效率。
**（4）close**
**关闭interceptor，主要用于执行一些资源清理工作**
如前所述，interceptor可能被运行在多个线程中，因此在具体实现时用户需要自行确保线程安全。另外**倘若指定了多个interceptor，则producer将按照指定顺序调用它们**，并仅仅是捕获每个interceptor可能抛出的异常记录到错误日志中而非在向上传递。这在使用过程中要特别留意。

#### 2-拦截器案例
1）需求：实现一个简单的双interceptor组成的拦截链。第一个interceptor会在消息发送前将时间戳信息加到消息value的最前部；第二个interceptor会在消息发送后更新成功发送消息数或失败发送消息数。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210328161857456.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NzM0NDAz,size_16,color_FFFFFF,t_70)
2）案例实操
（1）增加时间戳拦截器
（2）统计发送消息成功和发送失败消息数，并在producer关闭时打印这两个计数器
（3） producer主程序
3）测试
（1）在kafka上启动消费者，然后运行客户端java程序。
[atguigu@hadoop102 kafka]$ bin/kafka-console-consumer.sh \--bootstrap-serverhadoop102:9092--from-beginning --topic first
1501904047034,message0
1501904047225,message1
1501904047230,message2
1501904047234,message3
1501904047236,message4
1501904047240,message5
1501904047243,message6
1501904047246,message7
1501904047249,message8
1501904047252,message9

### 4. Kafka监控
#### 1-KafkaEagle
**1.修改kafka启动命令**
修改kafka-server-start.sh命令中
```
if [ "x$KAFKA_HEAP_OPTS" = "x" ]; then
	export KAFKA_HEAP_OPTS="-Xmx1G -Xms1G"
fi
```
为
```
if [ "x$KAFKA_HEAP_OPTS" = "x" ]; then
	export KAFKA_HEAP_OPTS="-server -Xms2G -Xmx2G -XX:PermSize=128m -XX:+UseG1GC -XX:MaxGCPauseMillis=200 -XX:ParallelGCThreads=8 -XX:ConcGCThreads=5 -XX:InitiatingHeapOccupancyPercent=70"
	export JMX_PORT="9999"
	#export KAFKA_HEAP_OPTS="-Xmx1G -Xms1G"
fi
```
**注意：修改之后在启动Kafka之前要分发之其他节点**
2.上传压缩包kafka-eagle-bin-1.3.7.tar.gz到集群/opt/software目录
3.解压到本地
[atguigu@hadoop102  software]$   tar -zxvf   kafka-eagle-bin-1.3.7.tar.gz

4.进入刚才解压的目录
[atguigu@hadoop102 kafka-eagle-bin-1.3.7]$ ll
总用量82932
-rw-rw-r--. 1 atguigu atguigu 84920710 8月13 23:00 kafka-eagle-web-1.3.7-bin.tar.gz

5.将kafka-eagle-web-1.3.7-bin.tar.gz解压至/opt/module
[atguigu@hadoop102 kafka-eagle-bin-1.3.7]$ tar -zxvf kafka-eagle-web-1.3.7-bin.tar.gz -C /opt/module/

6.修改名称
[atguigu@hadoop102 module]$ mv kafka-eagle-web-1.3.7/ eagle

7.给启动文件执行权限[atguigu@hadoop102 eagle]$ cd bin/
[atguigu@hadoop102 bin]$ ll
总用量12
-rw-r--r--. 1 atguigu atguigu 1848 8月22 2017 ke.bat
-rw-r--r--. 1 atguigu atguigu 7190 7月30 20:12 ke.sh
[atguigu@hadoop102 bin]$chmod 777 ke.sh

8.修改配置文件
######################################
#multi zookeeper&kafka cluster list
######################################
kafka.eagle.zk.cluster.alias=cluster1cluster1.zk.list=hadoop102:2181,hadoop103:2181,hadoop104:2181
######################################
#kafka offset storage
######################################
cluster1.kafka.eagle.offset.storage=kafka
######################################
#enable kafka metrics
######################################
kafka.eagle.metrics.charts=truekafka.eagle.sql.fix.error=false
######################################
#kafka jdbc driver address
######################################
kafka.eagle.driver=com.mysql.jdbc.Driverkafka.eagle.url=jdbc:mysql://hadoop102:3306/ke?useUnicode=true&characterEncoding=UTF-8&zeroDateTimeBehavior=convertToNullkafka.eagle.username=root
kafka.eagle.password=000000
9.添加环境变量
export KE_HOME=/opt/module/eagle
export PATH=$PATH:$KE_HOME/bin
注意：source /etc/profile
10.启动
[atguigu@hadoop102 eagle]$ bin/ke.sh start
... ...
... ...
*****************************************************************
**
* Kafka Eagle Service has started success.
* Welcome, Now you can visit 'http://192.168.9.102:8048/ke'
* Account:admin ,Password:123456
*****************************************************************
**
* <Usage> ke.sh [start|status|stop|restart|stats] </Usage>
* <Usage> https://www.kafka-eagle.org/ </Usage>
*******************************************************************
[atguigu@hadoop102 eagle]$
注意：启动之前需要先启动ZK以及KAFKA
11.登录页面查看监控数据
http://192.168.9.102:8048/ke第6章Flume对接Kafka1）配置flume(flume-kafka.conf)# definea1.sources = r1a1.sinks = k1
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210328164438403.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NzM0NDAz,size_16,color_FFFFFF,t_70)

### 5.其他
1.Kafka中的ISR(InSyncRepli)、OSR(OutSyncRepli)、AR(AllRepli)代表什么？2.Kafka中的HW、LEO等分别代表什么？3.Kafka中是怎么体现消息顺序性的？4.Kafka中的分区器、序列化器、拦截器是否了解？它们之间的处理顺序是什么？5.Kafka生产者客户端的整体结构是什么样子的？使用了几个线程来处理？分别是什么？6.“消费组中的消费者个数如果超过topic的分区，那么就会有消费者消费不到数据”这句话是否正确？7.消费者提交消费位移时提交的是当前消费到的最新消息的offset还是offset+1？8.有哪些情形会造成重复消费？9.那些情景会造成消息漏消费？
10.当你使用kafka-topics.sh创建（删除）了一个topic之后，Kafka背后会执行什么逻辑？1）会在zookeeper中的/brokers/topics节点下创建一个新的topic节点，如：/brokers/topics/first2）触发Controller的监听程序3）kafka Controller 负责topic的创建工作，并更新metadata cache11.topic的分区数可不可以增加？如果可以怎么增加？如果不可以，那又是为什么？12.topic的分区数可不可以减少？如果可以怎么减少？如果不可以，那又是为什么？13.Kafka有内部的topic吗？如果有是什么？有什么所用？14.Kafka分区分配的概念？15.简述Kafka的日志目录结构？16.如果我指定了一个offset，Kafka Controller怎么查找到对应的消息？17.聊一聊Kafka Controller的作用？18.Kafka中有那些地方需要选举？这些地方的选举策略又有哪些？19.失效副本是指什么？有那些应对措施？20.Kafka的哪些设计让它有如此高的性能？




区内有序：一个分区内有序
全局有序：一个分区+同步：get方法阻塞send

Kafka选举：Controller 抢资源  Leader选举 ISR  0.9前 响应时间+条数 0.9及后 响应时间






