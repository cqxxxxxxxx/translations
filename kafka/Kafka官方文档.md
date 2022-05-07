

[TOC]

http://kafka.apache.org/

> Kafka® is used for building real-time data pipelines and streaming apps. It is horizontally scalable, fault-tolerant, wicked fast, and runs in production in thousands of companies.



## Introduction

### Apache Kafka® is a distributed streaming platform. What exactly does that mean?

#### 1. 提供三个核心能力
- Publish and subscribe to streams of records, similar to a message queue or enterprise messaging system.  
可以发布、订阅流式记录，类似其他消息队列
- Store streams of records in a fault-tolerant durable way.  
可以持久化存储流式记录，并提供一定的容错性。
- Process streams of records as they occur.  
可以在流式记录产生时候就进行处理。

#### 2. 构建两大类别的应用
- Building real-time streaming data pipelines that reliably get data between systems or applications  
用于构建实时可靠的数据流通道，用于从系统或者应用获取数据
- Building real-time streaming applications that transform or react to the streams of data  
用于构建实时流处理应用，用于转换或者响应流式记录。

#### 3. 三个基础概念
- Kafka is run as a cluster on one or more servers that can span multiple datacenters.  
Kafka通常以集群方式在一个或者多个服务器上运行，可以横跨多个数据中心。
- The Kafka cluster stores streams of records in categories called topics.  
Kafka集群以topic为类别来存储流式记录。
- Each record consists of a key, a value, and a timestamp.  
每一条记录都包含一个key、value、timestamp

#### 4. 五个核心API
- The Producer API allows an application to publish a stream of records to one or more Kafka topics.  
Produce API允许应用程序发布一连串记录到一个或多个topic。

- The Consumer API allows an application to subscribe to one or more topics and process the stream of records produced to them.  
Consumer API允许应用程序订阅一个或多个topic并处理发送给这些topic的流式记录。

- The Streams API allows an application to act as a stream processor, consuming an input stream from one or more topics and producing an output stream to one or more output topics, effectively transforming the input streams to output streams.  
Stream API允许应用程序作为一个流处理器，消费一或多个topic的输入流并产生输出流到一或多个topic，在输入输出流中提供高效的数据转换。

- The Connector API allows building and running reusable producers or consumers that connect Kafka topics to existing applications or data systems. For example, a connector to a relational database might capture every change to a table.
Connector API允许构建运行一个可重用的生产者或消费者，that可以连接topics到应用程序或者数据系统。比如连接到关系型数据库用于捕获table的每一个变更。
https://github.com/search?q=kafka+connector 

- The Admin API allows managing and inspecting topics, brokers and other Kafka objects.  
Admin API可以用于管理，查看topics、brokers和kafka的其他对象。




### Topics and Logs
For each topic, the Kafka cluster maintains a partitioned log that looks like this:  
对于每一个topic，Kafka集群都维护一个分区过的日志，如下图：
![image.png](http://ww1.sinaimg.cn/large/d9913e6ely1geu9l8nh5ej20bk07ft8x.jpg)

Each partition is an ordered, immutable sequence of records that is continually appended to—a structured commit log. The records in the partitions are each assigned a sequential id number called the offset that uniquely identifies each record within the partition.  
每个分区都是一连串有序，不可变记录，并持续的追加到结构化的log中。分区中每一条记录都会分配一个连续的id即offset，offset是record在分区中的唯一标识。

The Kafka cluster durably persists all published records—whether or not they have been consumed—using a configurable retention period.  
Kafka集群持久化所有发布的记录，无论这些记录是否被消费过，并通过一个可配置的参数来设置持久化的时间。

Kafka's performance is effectively constant with respect to data size so storing data for a long time is not a problem.  
Kafka的性能和数据大小无关，所以长时间存储数据没有什么问题。

![image.png](http://ww1.sinaimg.cn/large/d9913e6ely1geuawdkb4wj21kp0yj429.jpg)
In fact, the only metadata retained on a per-consumer basis is the offset or position of that consumer in the log. This offset is controlled by the consumer: normally a consumer will advance its offset linearly as it reads records, but, in fact, since the position is controlled by the consumer it can consume records in any order it likes.  
实际上，每个消费者中唯一保存的元数据就是offset(消费记录的位置)。offset是由消费者控制的，通常它会线性增加，但是实际上，由于offset是由消费者控制的，因此它可以采用任意的顺序来消费记录。

For example a consumer can reset to an older offset to reprocess data from the past or skip ahead to the most recent record and start consuming from "now".  
例如，消费者可以重置offset到一个之前的位置来重新处理过去的数据，或者跳过最近的部分记录直接从当前开始消费。

The partitions in the log serve several purposes. First, they allow the log to scale beyond a size that will fit on a single server. Each individual partition must fit on the servers that host it, but a topic may have many partitions so it can handle an arbitrary amount of data. Second they act as the unit of parallelism—more on that in a bit.
log分区的主要用途有如下几个。一是当日志大小超过了单台服务器限制时可以进行扩展。虽然每个分区都受限于所在的主机，但是topic可以有多个分区来扩展处理任意数量的数据；二是可以作为并行的单元工作，提高性能。

### Distribution
The partitions of the log are distributed over the servers in the Kafka cluster with each server handling data and requests for a share of the partitions. Each partition is replicated across a configurable number of servers for fault tolerance.
log的分区分布于kafka集群的服务器上，每个服务处理数据和请求时都共享这些分区。分区可以通过配置进行进行特定数量的备份，用于提供容错性。

Each partition has one server which acts as the "leader" and zero or more servers which act as "followers". The leader handles all read and write requests for the partition while the followers passively replicate the leader. 
每一个分区都会有一台server作为leader，并且有0或N台server作为followers。leader负责处理所有的针对该分区的读写请求，与此同时followers负责被动的同步leader上的数据。

If the leader fails, one of the followers will automatically become the new leader. Each server acts as a leader for some of its partitions and a follower for others so load is well balanced within the cluster.
如果leader宕机了，其中的一个follower会自动成为新的leader。每台server都会成为某个分区的leader（不会所有的leader都堆积在某一两台server上），以及成为某个分区的follower，因此整个集群是负载均衡的。

### Geo-Replication
Kafka MirrorMaker provides geo-replication support for your clusters. With MirrorMaker, messages are replicated across multiple datacenters or cloud regions. You can use this in active/passive scenarios for backup and recovery; or in active/active scenarios to place data closer to your users, or support data locality requirements.
Kafka的MirrorMaker提供一个集群级别的跨区域的备份功能。通过MirrorMaker消息可以跨多个数据中心或者云服务提供商进行数据备份。通过这个功能可以进行数据的备份和恢复，或者把数据传输到离用户地理位置更近的区域。。。

### Producers
Producers publish data to the topics of their choice. The producer is responsible for choosing which record to assign to which partition within the topic. This can be done in a round-robin fashion simply to balance load or it can be done according to some semantic partition function (say based on some key in the record).   
生产者可以决定推送数据到哪些topics。Producer可以决定分配哪些记录到topic中的哪些分区中。可以使用round-robin方式来简单的负载均衡，或者基于某些分区函数来指定分区(例如基于record中的key来决定推送到哪个分区)。

### Consumers
Consumers label themselves with a consumer group name, and each record published to a topic is delivered to one consumer instance within each subscribing consumer group. Consumer instances can be in separate processes or on separate machines.
消费者使用一个消费组名称来标识，发布到topic中的每条记录会被分配给订阅的消费组中的一个消费者实例。消费者实例可以分布到不同进程或者不同机器上。

If all the consumer instances have the same consumer group, then the records will effectively be load balanced over the consumer instances.
如果所有的消费者实例在相同的消费组中，那么记录会有效的负载均衡到每个消费者实例上。

If all the consumer instances have different consumer groups, then each record will be broadcast to all the consumer processes.
如果所有的消费者实例都属于不同的消费组，那么每条记录都会广播到所有的消费者进程。

![image.png](http://ww1.sinaimg.cn/large/d9913e6ely1geud3h3a75j20d6070q3e.jpg)

More commonly, however, we have found that topics have a small number of consumer groups, one for each "logical subscriber". Each group is composed of many consumer instances for scalability and fault tolerance. 
比较常见的是，每一个topic都有一些消费组，每个消费组都是一个逻辑订阅者。每个消费者都由多个消费者实例组成便于扩展和容错。

The way consumption is implemented in Kafka is by dividing up the partitions in the log over the consumer instances so that each instance is the exclusive consumer of a "fair share" of partitions at any point in time. This process of maintaining membership in the group is handled by the Kafka protocol dynamically. If new instances join the group they will take over some partitions from other members of the group; if an instance dies, its partitions will be distributed to the remaining instances.  
在Kafka中实现消费的方式是将日志中的分区划分到每一个消费者实例上，因此在任何时间，每个实例都是分区唯一的消费者。维护消费组中的消费关系由Kafka协议动态处理。如果新的实例加入组，他们将从组中其他成员处接管一些 partition 分区;如果一个实例消失，拥有的分区将被分发到剩余的实例。

Kafka only provides a total order over records within a partition, not between different partitions in a topic. Per-partition ordering combined with the ability to partition data by key is sufficient for most applications. However, if you require a total order over records this can be achieved with a topic that has only one partition, though this will mean only one consumer process per consumer group.  
Kafka 只保证分区内的记录是有序的，而不保证主题中不同分区的顺序。每个 partition 分区按照key值排序足以满足大多数应用程序的需求。但如果你需要总记录在所有记录的上面，可使用仅有一个分区的主题来实现，这意味着每个消费者组只有一个消费者进程。

### Multi-tenancy
You can deploy Kafka as a multi-tenant solution. Multi-tenancy is enabled by configuring which topics can produce or consume data. There is also operations support for quotas. Administrators can define and enforce quotas on requests to control the broker resources that are used by clients. For more information, see the security documentation.



### Guarantees
At a high-level Kafka gives the following guarantees:
Kafka提供一下保证：

- Messages sent by a producer to a particular topic partition will be appended in the order they are sent.   
生产者发送消息给特定的topic的分区时，消息会以发送的顺序追加存储。

- A consumer instance sees records in the order they are stored in the log.  
消费者实例查看记录的顺序就是记录存储在log中的顺序。 

- For a topic with replication factor N, we will tolerate up to N-1 server failures without losing any records committed to the log.  
如果一个topic有N个副本，我们则可以最多可以忍受N-1台server故障，而不会丢失任何committed到log的记录。



### Kafka as a Messaging System
Messaging traditionally has two models: queuing and publish-subscribe. In a queue, a pool of consumers may read from a server and each record goes to one of them; in publish-subscribe the record is broadcast to all consumers. Each of these two models has a strength and a weakness. The strength of queuing is that it allows you to divide up the processing of data over multiple consumer instances, which lets you scale your processing. Unfortunately, queues aren't multi-subscriber—once one process reads the data it's gone. Publish-subscribe allows you broadcast data to multiple processes, but has no way of scaling processing since every message goes to every subscriber.  
传统的消息系统有两个模块: 队列 和 发布-订阅。 在队列中，消费者池从server读取数据，每条记录被池子中的一个消费者消费; 在发布订阅中，记录被广播到所有的消费者。两者均有优缺点。 队列的优点在于它允许你将处理数据的过程分给多个消费者实例，使你可以扩展处理过程。 不好的是，队列不是多订阅者模式的—一旦一个进程读取了数据，数据就会被丢弃。 而发布-订阅系统允许你广播数据到多个进程，但是无法进行扩展处理，因为每条消息都会发送给所有的订阅者。

The consumer group concept in Kafka generalizes these two concepts. As with a queue the consumer group allows you to divide up processing over a collection of processes (the members of the consumer group). As with publish-subscribe, Kafka allows you to broadcast messages to multiple consumer groups.  
消费组在Kafka有两层概念。在队列中，消费组允许你将处理过程分发给一系列进程(消费组中的成员)。 在发布订阅中，Kafka允许你将消息广播给多个消费组。

The advantage of Kafka's model is that every topic has both these properties—it can scale processing and is also multi-subscriber—there is no need to choose one or the other.  
Kafka的优势在于每个topic都有以下特性—可以扩展处理并且允许多订阅者模式—不需要只选择其中一个.

Kafka has stronger ordering guarantees than a traditional messaging system, too.  
Kafka相比于传统消息队列还具有更严格的顺序保证

传统队列在服务器上保存有序的记录，如果多个消费者消费队列中的数据， 服务器将按照存储顺序输出记录。 虽然服务器按顺序输出记录，但是记录被异步传递给消费者， 因此记录可能会无序的到达不同的消费者。这意味着在并行消耗的情况下， 记录的顺序是丢失的。因此消息系统通常使用“唯一消费者”的概念，即只让一个进程从队列中消费， 但这就意味着不能够并行地处理数据。

Kafka 设计的更好。topic中的partition是一个并行的概念。 Kafka能够为一个消费者池提供顺序保证和负载平衡，是通过将topic中的partition分配给消费者组中的消费者来实现的， 以便每个分区由消费组中的一个消费者消耗。通过这样，我们能够确保消费者是该分区的唯一读者，并按顺序消费数据。 众多分区保证了多个消费者实例间的负载均衡。但请注意，消费者组中的消费者实例个数不能超过分区的数量。

### Kafka as a Storage System
许多消息队列可以发布消息，除了消费消息之外还可以充当中间数据的存储系统。那么Kafka作为一个优秀的存储系统有什么不同呢?

数据写入Kafka后被写到磁盘，并且进行备份以便容错。直到完全备份，Kafka才让生产者认为完成写入，即使写入失败Kafka也会确保继续写入

Kafka使用磁盘结构，具有很好的扩展性—50kb和50TB的数据在server上表现一致。

可以存储大量数据，并且可通过客户端控制它读取数据的位置，您可认为Kafka是一种高性能、低延迟、具备日志存储、备份和传播功能的分布式文件系统。

### Kafka for Stream Processing

Kafka 流处理不仅仅用来读写和存储流式数据，它最终的目的是为了能够进行实时的流处理。

在Kafka中，流处理器不断地从输入的topic获取流数据，处理数据后，再不断生产流数据到输出的topic中去。

例如，零售应用程序可能会接收销售和出货的输入流，经过价格调整计算后，再输出一串流式数据。

简单的数据处理可以直接用生产者和消费者的API。对于复杂的数据变换，Kafka提供了Streams API。 Stream API 允许应用做一些复杂的处理，比如将流数据聚合或者join。

这一功能有助于解决以下这种应用程序所面临的问题：处理无序数据，当消费端代码变更后重新处理输入，执行有状态计算等。

Streams API建立在Kafka的核心之上：它使用Producer和Consumer API作为输入，使用Kafka进行有状态的存储， 并在流处理器实例之间使用相同的消费组机制来实现容错。



## DESIGN
### Motivation
It would have to have high-throughput to support high volume event streams such as real-time log aggregation.  
他需要有高吞吐量来支持高额的事件流，比如实时的日志聚合。

It would need to deal gracefully with large data backlogs to be able to support periodic data loads from offline systems.  
他需要能够很好的处理大量积压数据，以便周期性的处理离线系统的数据。

It also meant the system would have to handle low-latency delivery to handle more traditional messaging use-cases.  
他需要支持低延迟的消息分发，以便支持经典传统的消息传输场景

We wanted to support partitioned, distributed, real-time processing of these feeds to create new, derived feeds. This motivated our partitioning and consumer model.  
我们想要支持分区、分布式、实时处理输入数据并转换成新的输入数据，这激发了我们创造了分区和消费者模型。

Finally in cases where the stream is fed into other data systems for serving, we knew the system would have to be able to guarantee fault-tolerance in the presence of machine failures.
最后，在一些场景下数据需要经过kafka推送服务于其他数据系统，这种情况下我们要求系统在机器故障时保证容错。

### Persistence
Kafka relies heavily on the filesystem for storing and caching messages. There is a general perception that "disks are slow" which makes people skeptical that a persistent structure can offer competitive performance. In fact disks are both much slower and much faster than people expect depending on how they are used; and a properly designed disk structure can often be as fast as the network.  
kafka中消息的存储与缓存主要倚靠filesystem来实现。硬盘速度很慢，这是一个很普遍的看法，因此大家就会对基于磁盘的持久化架构能不能提供一个有竞争力的性能感到怀疑。实际上，基于对硬盘的使用方式，它的速度可能会比人们想象中的要更慢或者更快。实际上一个设计良好的磁盘结构可以跟网络速度一样快。

The key fact about disk performance is that the throughput of hard drives has been diverging from the latency of a disk seek for the last decade.   
关于磁盘的性能真正事实是，其实早在十年前硬盘的吞吐量就与磁盘寻址延迟分离开来了。 

As a result the performance of linear writes on a JBOD configuration with six 7200rpm SATA RAID-5 array is about 600MB/sec but the performance of random writes is only about 100k/sec—a difference of over 6000X.  


These linear reads and writes are the most predictable of all usage patterns, and are heavily optimized by the operating system. A modern operating system provides read-ahead and write-behind techniques that prefetch data in large block multiples and group smaller logical writes into large physical writes. 
在磁盘的所有使用方式中线性的读取和写入是可预测性最高的，也是被操作系统极度优化过的。现代的操作系统提供`read-ahead`和`write-behind`技术，可以从大数据块中预先读取数据，可以将多个小的逻辑写聚合成一次大的写入。  

A further discussion of this issue can be found in this ACM Queue article; they actually find that sequential disk access can in some cases be faster than random memory access!  
他们实际上发现顺序的磁盘访问在某些场景下比随机的内存访问还要快！！！

To compensate for this performance divergence, modern operating systems have become increasingly aggressive in their use of main memory for disk caching. A modern OS will happily divert all free memory to disk caching with little performance penalty when the memory is reclaimed. All disk reads and writes will go through this unified cache. This feature cannot easily be turned off without using direct I/O, so even if a process maintains an in-process cache of the data, this data will likely be duplicated in OS pagecache, effectively storing everything twice.  
（虽然上面说磁盘速度并不慢，但是只有在特定情况下，肯定不能完全依靠这个特性的）为了弥补性能差异，现代操作系统变得越来越积极的使用主内存来进行磁盘缓存。OS会很乐意转移所有闲置的内存用于磁盘缓存，尽管在内存回收时会造成一点小的性能影响。所有的磁盘读写都会经过OS这一个统一的缓存。除非使用直接IO，该缓存特性无法被避免，所以即使你的应用在在进程中自己维护了一个数据缓存，但实际上很可能这个数据在OS的pagecache上也被缓存了，即实际上做了两份缓存。


Furthermore, we are building on top of the JVM, and anyone who has spent any time with Java memory usage knows two things:    
1.The memory overhead of objects is very high, often doubling the size of the data stored (or worse).  
2.Java garbage collection becomes increasingly fiddly and slow as the in-heap data increases.    

Kafka是运行在JVM上面的，我们知道基于JVM的系统有如下两个特性  
1. 对象的内存开销很大，经常会比它实际存储的数据大上一倍。
2. java的垃圾回收会随着heap的增加而变的缓慢和难使用。

As a result of these factors using the filesystem and relying on pagecache is superior to maintaining an in-memory cache or other structure—we at least double the available cache by having automatic access to all free memory, and likely double again by storing a compact byte structure rather than individual objects.Doing so will result in a cache of up to 28-30GB on a 32GB machine without GC penalties.    
受上述因素的影响，使用filesystem并且倚靠pagecache比自己维护基于内存的缓存或者其他数据结构更占优势。（我们至少double了可用的缓存容量，通过自动获取所有的空闲内存。然后又double了一次通过存储紧凑的字节结构而不是jvm对象）通过这种方式，我们可以在32G的机器上缓存28-30G的数据，并且无需担心JVM的gc问题。  

Furthermore, this cache will stay warm even if the service is restarted, whereas the in-process cache will need to be rebuilt in memory (which for a 10GB cache may take 10 minutes) or else it will need to start with a completely cold cache (which likely means terrible initial performance). This also greatly simplifies the code as all logic for maintaining coherency between the cache and filesystem is now in the OS, which tends to do so more efficiently and more correctly than one-off in-process attempts. If your disk usage favors linear reads then read-ahead is effectively pre-populating this cache with useful data on each disk read.  
除此之外，即使服务重启pagecache也依旧可用，然而`in-process cache`需要在服务重启后重新构建(10GB缓存大概需要10分钟才能恢复完毕),或者也可以让服务从cold-cache状态开始慢慢恢复缓存（这会导致前期性能不佳）。基于OS的pagecache的缓存做法也极大的简化了kafka的代码，因为cache与filesystem一致性有关的事宜都由OS维护处理了，这比一次性的基于进程的缓存更加高效和准确。如果你的磁盘更多的是线性读取，那么read-ahead机制可以在你每次磁盘读时提前把有用的数据读取填充到cache中。

### Constant Time Suffices

The persistent data structure used in messaging systems are often a per-consumer queue with an associated BTree or other general-purpose random access data structures to maintain metadata about messages. BTrees are the most versatile data structure available, and make it possible to support a wide variety of transactional and non-transactional semantics in the messaging system. They do come with a fairly high cost, though: Btree operations are O(log N). Normally O(log N) is considered essentially equivalent to constant time, but this is not true for disk operations. Disk seeks come at 10 ms a pop, and each disk can do only one seek at a time so parallelism is limited. Hence even a handful of disk seeks leads to very high overhead. Since storage systems mix very fast cached operations with very slow physical disk operations, the observed performance of tree structures is often superlinear as data increases with fixed cache--i.e. doubling your data makes things much worse than twice as slow.  
在消息系统中使用的持久化数据结构通常是每个消费者队列绑定一个BTree或者使用其他通用的用于随机访问的数据结构来维护消息的元数据。BTree是最通用的数据结构，可以支持消息系统中的事务或者非事务性语义。BTree会带来比较高的成本，尽管BTree的操作复杂度是O(log N)。通常来说O(log N)本质上认为是常数时间，但这在磁盘操作上不成立。磁盘寻址大概10ms每一跳，并且一个磁盘同时只能进行一个寻址操作，所以并发能力就受到了限制。因此即使很少的磁盘寻址也会导致高额的开销。自从存储系统混合了非常快的缓存操作以及非常慢的物理磁盘操作，观察到基于树结构的存储通常随着数据增加它的性能是超线性的，例如数据翻倍会导致你性能下降两倍不止。（树是随机存储结构，磁盘寻址比较多）

Intuitively a persistent queue could be built on simple reads and appends to files as is commonly the case with logging solutions. This structure has the advantage that all operations are O(1) and reads do not block writes or each other. This has obvious performance advantages since the performance is completely decoupled from the data size—one server can now take full advantage of a number of cheap, low-rotational speed 1+TB SATA drives. Though they have poor seek performance, these drives have acceptable performance for large reads and writes and come at 1/3 the price and 3x the capacity.  
直观的说，持久化队列可以基于简单的文件读和追加写操作上，这在日志解决方案中是很常见的。队列结构有个优点，那就是所有操作都是O（1），并且读写操作不会互相阻塞。显然这种结构的性能优势很明显，因为性能和数据大小完全解耦开来，因此可以充分利用大量廉价、低转速的1+TB SATA硬盘，尽管这些硬盘的寻址性能比较差，但他们在大规模读写方面的性能是可以接受的，而且价格是原来的三分之一、容量是原来的三倍。
Having access to virtually unlimited disk space without any performance penalty means that we can provide some features not usually found in a messaging system. For example, in Kafka, instead of attempting to delete messages as soon as they are consumed, we can retain messages for a relatively long period (say a week). This leads to a great deal of flexibility for consumers, as we will describe.
在不损失任何性能的情况下我们还能访问几乎无限的硬盘空间，这使得我们可以提供常规消息系统不具备的特性。比如在kafka中，我们可以把消息数据保存相对较长的一段时间（例如一周），而不是当消息被消费后尝试删除这些数据。正如我们描述的，这将为消费者提供极大的灵活性。


### Efficiency

We have put significant effort into efficiency. One of our primary use cases is handling web activity data, which is very high volume: each page view may generate dozens of writes. Furthermore, we assume each message published is read by at least one consumer (often many), hence we strive to make consumption as cheap as possible.  
我们做了很多努力来提高效率。我们的一个主要用例就是处理web活动数据，这个数据量非常大：每一次页面流量可能会产生几十个写操作。此外，我们假设消息发送后会被至少一个conusemer读取（通常是更多的消费者），因此我们努力降低消费的代价。

We have also found, from experience building and running a number of similar systems, that efficiency is a key to effective multi-tenant operations. If the downstream infrastructure service can easily become a bottleneck due to a small bump in usage by the application, such small changes will often create problems. By being very fast we help ensure that the application will tip-over under load before the infrastructure. This is particularly important when trying to run a centralized service that supports dozens or hundreds of applications on a centralized cluster as changes in usage patterns are a near-daily occurrence.

We discussed disk efficiency in the previous section. Once poor disk access patterns have been eliminated, there are two common causes of inefficiency in this type of system: too many small I/O operations, and excessive byte copying.
我们再之前章节讨论了磁盘的效率。一旦消除了较差的磁盘访问方式，这类系统性能低下的原因主要还有两种：太多小的IO操作，昂贵的字节拷贝。

The small I/O problem happens both between the client and the server and in the server's own persistent operations.  
小的IO问题发生在client和server端之间以及server端自己的持久化操作中。  

To avoid this, our protocol is built around a "message set" abstraction that naturally groups messages together. This allows network requests to group messages together and amortize the overhead of the network roundtrip rather than sending a single message at a time. The server in turn appends chunks of messages to its log in one go, and the consumer fetches large linear chunks at a time.
为了避免太多小IO，我们的协议基于一个抽象层“消息集”构建，自然的将消息进行分组。这样就允许网络请求把消息分组在一起从而均摊网络往返的开销，而不是每个消息发送一次。server端则一次性将消息块append到log后面，消费者可以则一次性拉取大的线性消息块。

This simple optimization produces orders of magnitude speed up. Batching leads to larger network packets, larger sequential disk operations, contiguous memory blocks, and so on, all of which allows Kafka to turn a bursty stream of random message writes into linear writes that flow to the consumers.  
这个简单的优化对性能产生了数量级的提升。批处理允许更大的网络数据包，更大的顺序磁盘操作，连续的内存块等等，所有的这一切允许kakfa把间歇性的随机消息写入流转换成线性写入并流向消费者。

The other inefficiency is in byte copying. At low message rates this is not an issue, but under load the impact is significant. To avoid this we employ a standardized binary message format that is shared by the producer, the broker, and the consumer (so data chunks can be transferred without modification between them).  
另一个低效率的原因就是字节拷贝。在消息量少时这不是一个大问题，但是在系统负载不足时影响就意义非凡了。为了避免这字节拷贝导致的性能影响，我们使用了标准化的二进制消息格式，并在生产者，消费者，broker中共享。（所以数据块在传输时不需要进行修改）。

The message log maintained by the broker is itself just a directory of files, each populated by a sequence of message sets that have been written to disk in the same format used by the producer and consumer. Maintaining this common format allows optimization of the most important operation: network transfer of persistent log chunks. Modern unix operating systems offer a highly optimized code path for transferring data out of pagecache to a socket; in Linux this is done with the sendfile system call.  
broker维护的消息日志本身就是一个文件目录，每个文件都由写到磁盘的同样格式的有序的消息集组成，这些消息集被producer和consumer使用。维护这种统一的消息格式有利于在一些重要操作上进行优化：持久化log块的网络传输。现代操作系统提供一个高度优化了的code path来将数据从pagecache传输到socket。在linux中通过sendfile系统调用来实现。

To understand the impact of sendfile, it is important to understand the common data path for transfer of data from file to socket:

The operating system reads data from the disk into pagecache in kernel space
The application reads the data from kernel space into a user-space buffer
The application writes the data back into kernel space into a socket buffer
The operating system copies the data from the socket buffer to the NIC buffer where it is sent over the network
This is clearly inefficient, there are four copies and two system calls. Using sendfile, this re-copying is avoided by allowing the OS to send the data from pagecache to the network directly. So in this optimized path, only the final copy to the NIC buffer is needed.  
为了理解sendfile的影响力作用，了解数据从文件到socket的传输路径就非常必要。
1. OS从磁盘读取数据到pagecache（内核空间）
2. 应用从内核空间读取数据到用户空间缓存
3. 应用写回数据到内核空间的socket缓存
4. OS从socket缓存拷贝数据到NIC缓存（负责发送数据到网络中）
很明显，以上流程效率很低，因为有4次拷贝和2次系统调用。使用sendfile，这些重复的拷贝可以避免掉，sendfile允许OS直接从pagecache发送数据到网络。所以在这种优化过的传输路径下只需要最后一次NIC缓存的拷贝就够了。（4->1）


We expect a common use case to be multiple consumers on a topic. Using the zero-copy optimization above, data is copied into pagecache exactly once and reused on each consumption instead of being stored in memory and copied out to user-space every time it is read. This allows messages to be consumed at a rate that approaches the limit of the network connection.  
我们期望一个普通的场景，即一个topic被多个消费者消费。使用上述的zero-copy优化，数据每次读取时只会拷贝到pagecache一次并且通过pagecache可以被重复消费使用，而不是存储在内存中并且每一次都需要拷贝到user-space。通过这种方式消息能够以接近网络连接速度的进行消费。

This combination of pagecache and sendfile means that on a Kafka cluster where the consumers are mostly caught up you will see no read activity on the disks whatsoever as they will be serving data entirely from cache.  
pagecache和sendfiel的组合意味着在一个kafka集群中，大多数 consumer 消费时，您将看不到磁盘上的读取活动，因为数据将完全由缓存提供。

### End-to-end Batch Compression
In some cases the bottleneck is actually not CPU or disk but network bandwidth. This is particularly true for a data pipeline that needs to send messages between data centers over a wide-area network. Of course, the user can always compress its messages one at a time without any support needed from Kafka, but this can lead to very poor compression ratios as much of the redundancy is due to repetition between messages of the same type (e.g. field names in JSON or user agents in web logs or common string values). Efficient compression requires compressing multiple messages together rather than compressing each message individually.  
在某些场景下，系统的瓶颈既不是CPU也不是磁盘而是网络带宽。这对数据pipeline来说是非常正确的，因为他需要在基于广域网在数据中心之间发送消息。当然，用户可以在没有kafka的支持情况下自行压缩消息，但是这可能会导致比较差的压缩比和消息重复类型的冗余，比如 JSON 中的字段名称或者是或 Web 日志中的用户代理或公共字符串值。高性能的压缩是一次压缩多个消息，而不是压缩单个消息。

Kafka supports this with an efficient batching format. A batch of messages can be clumped together compressed and sent to the server in this form. This batch of messages will be written in compressed form and will remain compressed in the log and will only be decompressed by the consumer.  
kafka支持高效的批处理格式。一批消息可以压缩一起并发送到服务器。这批消息将以压缩后的格式写入日志，而且只会在 consumer 消费时解压缩。

Kafka supports GZIP, Snappy, LZ4 and ZStandard compression protocols. More details on compression can be found here.


### The Producer 
#### Load balancing
The producer sends data directly to the broker that is the leader for the partition without any intervening routing tier. To help the producer do this all Kafka nodes can answer a request for metadata about which servers are alive and where the leaders for the partitions of a topic are at any given time to allow the producer to appropriately direct its requests.  
producer直接发送数据给分区leader所在的broker，不需要任何中间的路由层。为了使producer能够这样做，kafka所有的节点都需要能够在任意时候响应获取metadata的请求，metadata里包含了那些server是存活的以及topic分区的leader所在位置，通过获取metadata以此producer可以直接向目标发送请求。

The client controls which partition it publishes messages to. This can be done at random, implementing a kind of random load balancing, or it can be done by some semantic partitioning function. We expose the interface for semantic partitioning by allowing the user to specify a key to partition by and using this to hash to a partition (there is also an option to override the partition function if need be). For example if the key chosen was a user id then all data for a given user would be sent to the same partition. This in turn will allow consumers to make locality assumptions about their consumption. This style of partitioning is explicitly designed to allow locality-sensitive processing in consumers.  
client控制发送消息到哪个分区。这可以通过random负载均衡方式实现，或者基于某种语义的分区函数实现。我们开放了语义分区接口来让用户进行指定的键值进行hash分区(当然也有选项可以重写分区函数)。

#### Asynchronous send 

Batching is one of the big drivers of efficiency, and to enable batching the Kafka producer will attempt to accumulate data in memory and to send out larger batches in a single request. The batching can be configured to accumulate no more than a fixed number of messages and to wait no longer than some fixed latency bound (say 64k or 10 ms). This allows the accumulation of more bytes to send, and few larger I/O operations on the servers. This buffering is configurable and gives a mechanism to trade off a small amount of additional latency for better throughput.  
批处理是提升性能的一个主要驱动，为了支持批处理，kafka producer会尝试在内存中积累数据并在一个请求中发送更大的数据。批处理可以配置成累积不超过特定固定数量的消息或者等待不超过特定的延迟时间（比如64k或者10ms）。这允许累积更多的bytes来发送，并且在server端形成更少的更大的io操作。这个producer的缓存机制是可配置的，它提供了一个机制来权衡少量的额外延迟来获取更高的吞吐量。

### The Consumer
The Kafka consumer works by issuing "fetch" requests to the brokers leading the partitions it wants to consume. The consumer specifies its offset in the log with each request and receives back a chunk of log beginning from that position. The consumer thus has significant control over this position and can rewind it to re-consume data if need be. 
Kafka consumer通过发送fetch请求到brokers来获取他想要消费的分区。consumer在每个请求中指定了log的offset，并收到一块从offset指定的位置开始的数据。consumer因此对offset的控制极其重要，如果有需要他可以通过倒带来重复消费数据。

#### Pull vs Push

An initial question we considered is whether consumers should pull data from brokers or brokers should push data to the consumer. In this respect Kafka follows a more traditional design, shared by most messaging systems, where data is pushed to the broker from the producer and pulled from the broker by the consumer. Some logging-centric systems, such as Scribe and Apache Flume, follow a very different push-based path where data is pushed downstream. There are pros and cons to both approaches. However, a push-based system has difficulty dealing with diverse consumers as the broker controls the rate at which data is transferred. The goal is generally for the consumer to be able to consume at the maximum possible rate; unfortunately, in a push system this means the consumer tends to be overwhelmed when its rate of consumption falls below the rate of production (a denial of service attack, in essence). A pull-based system has the nicer property that the consumer simply falls behind and catches up when it can. This can be mitigated with some kind of backoff protocol by which the consumer can indicate it is overwhelmed, but getting the rate of transfer to fully utilize (but never over-utilize) the consumer is trickier than it seems. Previous attempts at building systems in this fashion led us to go with a more traditional pull model.  
我们最开始考虑的问题是，消费者一个从broker中pull数据，还是由broker push数据到消费者中。在这方面kafka采用了在消息系统中最通用的设计方案，即消息由producer推送到broker，然后由consumer从broker中拉取。一些以日志记录为中心的系统，例如Scribe和Flume，采用基于push的路径来推送数据到下游。push和pull这两种方式都有各自的优缺点。然而，基于push的系统很难处理不同的消费组，因为broker控制着消息传输的速率。消息系统的目标是让消费组尽量能够以最大的速率来消费数据。不幸的是，在基于push的系统中，当消费速率小于生产速率时，消费者往往可能被压垮（dos攻击类似）。在基于pull的系统中，有更好的特性即消费者在消费速率落后的情况下可以在任何适当的时间追上去。还可以通过使用某种 backoff 协议来减少这种现象：即 consumer 可以通过 backoff 表示它已经不堪重负了，然而通过获得负载情况来充分使用 consumer（但永远不超载）这一方式实现起来比它看起来更棘手。前面以这种方式构建系统的尝试，引导着 Kafka 走向了更传统的 pull 模型。

Another advantage of a pull-based system is that it lends itself to aggressive batching of data sent to the consumer. A push-based system must choose to either send a request immediately or accumulate more data and then send it later without knowledge of whether the downstream consumer will be able to immediately process it. If tuned for low latency, this will result in sending a single message at a time only for the transfer to end up being buffered anyway, which is wasteful. A pull-based design fixes this as the consumer always pulls all available messages after its current position in the log (or up to some configurable max size). So one gets optimal batching without introducing unnecessary latency.  
另外一个pull-based系统的优势是它可以积极的对发送给消费者的数据进行批处理。push-based系统必须选择及时的发送消息或者推迟发送并累积更多的数据然后再发送（它并不知道下游消费者能否及时处理这些数据）。如果调整为低延迟则会导致每次只传输只发送一个消息，这很浪费。而pull-based的设计解决了这个问题，消费者总是拉取在当前位置后的所有可用的消息（或者配置的最大值）。所以使得数据能够得到最佳的处理而不会引入不必要的延迟。

The deficiency of a naive pull-based system is that if the broker has no data the consumer may end up polling in a tight loop, effectively busy-waiting for data to arrive. To avoid this we have parameters in our pull request that allow the consumer request to block in a "long poll" waiting until data arrives (and optionally waiting until a given number of bytes is available to ensure large transfer sizes).  
简单的pull-based的系统不足之处在于，如果broker上没用数据，那么consumer会一直进行紧密的poll循环，实际上忙着等待数据到达。为了阻止这种情况我们在poll请求中提供一个参数，允许consumer阻塞直到数据到达（还可以选择等待给定字节长度的数据来确保传输长度）。


You could imagine other possible designs which would be only pull, end-to-end. The producer would locally write to a local log, and brokers would pull from that with consumers pulling from them. A similar type of "store-and-forward" producer is often proposed. This is intriguing but we felt not very suitable for our target use cases which have thousands of producers. Our experience running persistent data systems at scale led us to feel that involving thousands of disks in the system across many applications would not actually make things more reliable and would be a nightmare to operate. And in practice we have found that we can run a pipeline with strong SLAs at large scale without a need for producer persistence.  


#### Consumer Position

Keeping track of what has been consumed is, surprisingly, one of the key performance points of a messaging system.  
令人惊奇的是，如何持续追踪消费的位置是消息系统性能的关键点。

Most messaging systems keep metadata about what messages have been consumed on the broker. That is, as a message is handed out to a consumer, the broker either records that fact locally immediately or it may wait for acknowledgement from the consumer. This is a fairly intuitive choice, and indeed for a single machine server it is not clear where else this state could go. Since the data structures used for storage in many messaging systems scale poorly, this is also a pragmatic choice--since the broker knows what is consumed it can immediately delete it, keeping the data size small.  
大多数消息系统保存消息消费位置的元数据信息在broker端。因此，如果一个消息被分发给consumer，broker要么立即在本地记录该事件，或者等到消费者的ack后记录该事件。这是一个相当直观的选择，而且事实上对于单机服务器来说，也没与其它地方能够存储这些状态信息。由于大多数消息系统用于存储的数据结构规模都很小，扩展性比较差，所以这也是一个很实用的选择——因为只要 broker 知道哪些消息被消费了，就可以在本地立即进行删除，一直保持较小的数据量。

What is perhaps not obvious is that getting the broker and consumer to come into agreement about what has been consumed is not a trivial problem. If the broker records a message as consumed immediately every time it is handed out over the network, then if the consumer fails to process the message (say because it crashes or the request times out or whatever) that message will be lost. To solve this problem, many messaging systems add an acknowledgement feature which means that messages are only marked as sent not consumed when they are sent; the broker waits for a specific acknowledgement from the consumer to record the message as consumed. This strategy fixes the problem of losing messages, but creates new problems. First of all, if the consumer processes the message but fails before it can send an acknowledgement then the message will be consumed twice. The second problem is around performance, now the broker must keep multiple states about every single message (first to lock it so it is not given out a second time, and then to mark it as permanently consumed so that it can be removed). Tricky problems must be dealt with, like what to do with messages that are sent but never acknowledged.  
还未明确的是，让broker和consumer对消费完成这个事件达成一致不是一个小问题。如果broker每次分发消息到网络中就标记为已消费，那么如果consumer再处理message时失败了(宕机或者请求超时)则消息就好丢失。为了解决这个问题，很多消息系统添加了ack机制，这意味着消息在分发后只会标记为已分发，然后broker等待consumer发送的特定的ack消息才标记成已消费状态。这个策略解决了消息丢失的问题，但是引出了新的问题。首先如果consumer成功处理了message，但是发送ack的时候失败了，那么这条消息会消费两次。第二个问题是关于性能，broker现在必须为一个消息管理多个状态（发送后标记lock，然后标记consumed，之后才可以被安全删除）。还有更棘手的问题要处理，比如如何处理已经发送但一直得不到确认的消息。


Kafka handles this differently. Our topic is divided into a set of totally ordered partitions, each of which is consumed by exactly one consumer within each subscribing consumer group at any given time. This means that the position of a consumer in each partition is just a single integer, the offset of the next message to consume. This makes the state about what has been consumed very small, just one number for each partition. This state can be periodically checkpointed. This makes the equivalent of message acknowledgements very cheap.  
kafka以不同的方式处理这个问题。我们的topic被切分成一些有序的partitions，每一个partition在任意时间都只会被消费组中一个consumer消费。这意味着每个partition的消费位置就只是一个简单的integer，即下一条要被消费的消息的offset。这就使得记录被状态信息很小，只要每个partition一个integer。这个状态信息还可以作为周期性的 checkpoint。这以非常低的代价实现了和消息确认机制等同的效果。

There is a side benefit of this decision. A consumer can deliberately rewind back to an old offset and re-consume data. This violates the common contract of a queue, but turns out to be an essential feature for many consumers. For example, if the consumer code has a bug and is discovered after some messages are consumed, the consumer can re-consume those messages once the bug is fixed.  
这样做还有一个好处。消费者可以回退到之前的offset并重新消费数据。这违背了通常的队列约定，但是对某些消费组来说这可能是一个非常有用的特性。比如consumer代码出现了bug，然而已经消费了一部分消息，此时consumer可以在bug修复后重新消费这些消息。


#### Offline Data Load
Scalable persistence allows for the possibility of consumers that only periodically consume such as batch data loads that periodically bulk-load data into an offline system such as Hadoop or a relational data warehouse.
可伸缩的持久化特性允许 consumer 只进行周期性的消费，例如批量数据加载，周期性将数据加载到诸如 Hadoop 和关系型数据库之类的离线系统中。

In the case of Hadoop we parallelize the data load by splitting the load over individual map tasks, one for each node/topic/partition combination, allowing full parallelism in the loading. Hadoop provides the task management, and tasks which fail can restart without danger of duplicate data—they simply restart from their original position.


#### Static Membership
Static membership aims to improve the availability of stream applications, consumer groups and other applications built on top of the group rebalance protocol. The rebalance protocol relies on the group coordinator to allocate entity ids to group members. These generated ids are ephemeral and will change when members restart and rejoin. For consumer based apps, this "dynamic membership" can cause a large percentage of tasks re-assigned to different instances during administrative operations such as code deploys, configuration updates and periodic restarts. For large state applications, shuffled tasks need a long time to recover their local states before processing and cause applications to be partially or entirely unavailable. Motivated by this observation, Kafka’s group management protocol allows group members to provide persistent entity ids. Group membership remains unchanged based on those ids, thus no rebalance will be triggered.  

static membership主要用于提高流式应用，消费组和其他构建在消费组rebalace协议之上的应用。rebalance协议基于group coordinator来分配实体id给消费组内成员。这些生成的id是朝生夕灭短暂的并且会在消息组成员重启并重新加入时候改变。对于基于consumer的应用，当管理操作，比如部署，配置更新，周期性的重启等等发生时候，这些动态的成员可能导致大量的task重新分配到不同的实例。对于大量状态的应用，洗牌后的任务在执行前需要大量时间来恢复他们的之前的状态，这导致应用周期性或整体的不可用。基于这一观察，kafka的group管理协议允许组内成员提供持久化的entity id。因此即使rebalance发生了，组成员基于他们持久化的id任然可以保持不变。

#### Message Delivery Semantics
Now that we understand a little about how producers and consumers work, let's discuss the semantic guarantees Kafka provides between producer and consumer. Clearly there are multiple possible message delivery guarantees that could be provided:

- At most once—Messages may be lost but are never redelivered.
- At least once—Messages are never lost but may be redelivered.
- Exactly once—this is what people actually want, each message is delivered once and only once.  


### Replication
Kafka replicates the log for each topic's partitions across a configurable number of servers (you can set this replication factor on a topic-by-topic basis). This allows automatic failover to these replicas when a server in the cluster fails so messages remain available in the presence of failures.  
kafka给每个topic partition的log进行备份，通过server上一个可配置的数量（可以全局或者topic级别设置）。当集群中某个server fail了，这就可以进行自动故障转移到备份，因此即使故障出现消息任然可用。 

Other messaging systems provide some replication-related features, but, in our (totally biased) opinion, this appears to be a tacked-on thing, not heavily used, and with large downsides: replicas are inactive, throughput is heavily impacted, it requires fiddly manual configuration, etc. Kafka is meant to be used with replication by default—in fact we implement un-replicated topics as replicated topics where the replication factor is one.  
其他消息系统提供备份相关的特性，但是在我们看来，这些特性是额外附加的，不常用的，而且有很大缺点：副本非活跃的，吞吐量受到严重影响，需要额外手动配置。kafka默认就会使用备份机制，我们通过指定备份数量为1来实现不需要备份的topic。

The unit of replication is the topic partition. Under non-failure conditions, each partition in Kafka has a single leader and zero or more followers. The total number of replicas including the leader constitute the replication factor. All reads and writes go to the leader of the partition. Typically, there are many more partitions than brokers and the leaders are evenly distributed among brokers. The logs on the followers are identical to the leader's log—all have the same offsets and messages in the same order (though, of course, at any given time the leader may have a few as-yet unreplicated messages at the end of its log).  
副本的单位是topic的分区。在不发生故障的条件下，每个分区都有一个leader和0-N个followers。副本的总数（包括leader）组成了replication factor。所有的读写都会作用于分区的leader上。通常来说，一般分区的数量会比broker数量多，leader也会均匀的分布在broker上。follower上的日志跟leader的日志完全一致，都有相同的offsets，相同顺序的messages。（当然，在任何时刻leader都会有一些没有备份好的消息在log的末端）

Followers consume messages from the leader just as a normal Kafka consumer would and apply them to their own log. Having the followers pull from the leader has the nice property of allowing the follower to naturally batch together log entries they are applying to their log.  
followers从leader上消费消息就像普通的消费者一样。因此follower有一个很好的特性那就是他们可以自然的批量的拉取log到他们自己的日志文件中。

As with most distributed systems automatically handling failures requires having a precise definition of what it means for a node to be "alive". For Kafka node liveness has two conditions

1. A node must be able to maintain its session with ZooKeeper (via ZooKeeper's heartbeat mechanism)
2. If it is a follower it must replicate the writes happening on the leader and not fall "too far" behind  
1. 节点必须跟zk保持回话（zk的heartbeat机制）
2. 如果节点是follower，他必须备份所有leader上的写操作并不能落后太多。

We refer to nodes satisfying these two conditions as being "in sync" to avoid the vagueness of "alive" or "failed". The leader keeps track of the set of "in sync" nodes. If a follower dies, gets stuck, or falls behind, the leader will remove it from the list of in sync replicas. The determination of stuck and lagging replicas is controlled by the replica.lag.time.max.ms configuration.  
我们把满足这两个条件的节点认为是“in sync”状态，用于区别于 alive 和failed状态。leader会保持追踪“in sync”状态的节点。如果follower挂了，阻塞了或者落后太多，leader会把它从“in sync”列表中移除。这通过 replica.lag.time.max.ms 配置来控制。

In distributed systems terminology we only attempt to handle a "fail/recover" model of failures where nodes suddenly cease working and then later recover (perhaps without knowing that they have died). Kafka does not handle so-called "Byzantine" failures in which nodes produce arbitrary or malicious responses (perhaps due to bugs or foul play).  

We can now more precisely define that a message is considered committed when all in sync replicas for that partition have applied it to their log. Only committed messages are ever given out to the consumer. This means that the consumer need not worry about potentially seeing a message that could be lost if the leader fails. Producers, on the other hand, have the option of either waiting for the message to be committed or not, depending on their preference for tradeoff between latency and durability. This preference is controlled by the acks setting that the producer uses. Note that topics have a setting for the "minimum number" of in-sync replicas that is checked when the producer requests acknowledgment that a message has been written to the full set of in-sync replicas. If a less stringent acknowledgement is requested by the producer, then the message can be committed, and consumed, even if the number of in-sync replicas is lower than the minimum (e.g. it can be as low as just the leader).  
现在, 我们可以更精确地定义, 只有当消息被所有的副本节点加入到日志中时, 才算是提交, 只有提交的消息才会被 consumer 消费, 这样就不用担心一旦 leader 挂掉了消息会丢失。另一方面， producer 也 可以选择是否等待消息被提交，这取决他们的设置在延迟时间和持久性之间的权衡，这个选项是由 producer 使用的 acks 设置控制。 请注意，Topic 可以设置同步备份的最小数量， producer 请求确认消息是否被写入到所有的备份时, 可以用最小同步数量判断。如果 producer 对同步的备份数没有严格的要求，即使同步的备份数量低于 最小同步数量（例如，仅仅只有 leader 同步了数据），消息也会被提交，然后被消费。

The guarantee that Kafka offers is that a committed message will not be lost, as long as there is at least one in sync replica alive, at all times.  
kafka提供的保证是，如果消息被committed那么它就不会丢失，只要还存在一个in sync的副本。

Kafka will remain available in the presence of node failures after a short fail-over period, but may not remain available in the presence of network partitions.  
kafka在节点挂掉后经过一个短暂的故障转移后会任然可用。但是在出现网络分区时可能不会生效。

### Replicated Logs: Quorums, ISRs, and State Machines (Oh my!)

