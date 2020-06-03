## Redis是什么？

Redis是一个开源的、**基于内存**的**数据结构存储器**，可以用作**数据库**、**缓存**和**消息中间件**。



## Redis是如何实现的？

redis作为一个缓存，如果要查询够快，当然要使用hash。**不管你用什么样的Map，它的背后都是key-value的Hash表结构，目的就是为了实现O(1)复杂度的查找算法，**Redis也是这样实现的，另一个常用的缓存框架Memcached也是。

## Redis是什么架构？

Redis是C/S架构，即客户端服务端模式。我们可以通过Redis的命令行，当然也可以**通过各种语言的Redis API**，在代码里面对Hash表进行操作，这些都是Redis客户端（Client），而Hash表所在的是Redis服务端（Server），也就是说Redis其实是一个C/S架构。

值得一提的是，**Redis的Server是单线程服务器**，基于**Event-Loop模式**来处理Client的请求，这一点和NodeJS很相似。使用单线程的好处包括：

- **不必考虑线程安全问题。**很多操作都不必加锁，既简化了开发，又提高了性能；
- **减少线程切换损耗的时间。**线程一多，CPU在线程之间切来切去是非常耗时的，单线程服务器则没有了这个烦恼；



## 当client增多，redis无法应对这么大的访问量，怎么办？

首先，client增多会带来两个问题：

- **Redis内存不足**：随着使用Redis的客户端越来越多，Redis上的缓存数据也越来越大，而一台机器的内存毕竟是有限的，放不了那么多数据；
- **Redis吞吐量低**：客户端变多了，可Redis还是只有一台，而且我们已经知道，Redis是单线程的！这就好比我开了一家饭店，一开始每天只有100位客人，我雇一位服务员就可以，后来生意好了，每天有1000位客人，可我还是只雇一位服务员。**一台机器的带宽和处理器都是有限的**，Redis自然会忙不过来，吞吐量已经不足以支撑我们越来越庞大的系统。

针对这样的问题，我们可以通过**集群**解决，即增加redis的数量。优点有：

- **扩大缓存容量；**
- **提升吞吐量；**



## 当其中一个或多个redis挂了性能变差怎么办？

现在我们已经把Redis升级到了集群，真可谓效果杠杠的，可运行了一段时间后，运维又过来反馈了两个问题：

- **数据可用性差**：如果其中一台Redis挂了，那么上面全部的缓存数据都会丢失，导致原来可以从缓存中获取的请求，都去访问数据库了，数据库压力陡增。
- **数据查询缓慢**：监测发现，每天有一段时间，Redis 1的访问量非常高，而且大多数请求都是去查一个相同的缓存数据，导致Redis 1非常忙碌，吞吐量不足以支撑这个高的查询负载。

问题分析完，要想解决可用性问题，我们第一个想到的，就是数据库里头经常用到的**Master-Slave模式**，于是，我们给每一台Redis都加上了一台Slave：

通过Master-Slave模式，我们又实现了两个特性：

- **数据高可用**：Master负责接收客户端的写入请求，将数据写到Master后，同步给Slave，实现数据备份。一旦Master挂了，可以将Slave提拔为Master；
- **提高查询效率**：一旦Master发现自己忙不过来了，**可以把一些查询请求，转发给Slave去处理**，也就是Master负责读写或者只负责写，Slave负责读；

如果使用一个master对应多个slave则可以发挥M/S模式更强的威力，但是master每次备份需要进行n次，工作量变大。所以可以考虑master/slave-chains架构，即一棵**二叉树**，根节点是master，其他节点都是slave。这样只需要进行两次备份。



事实上，Redis内部要处理的问题还有很多：

- **数据结构。**文章一开头提到了，Redis不仅仅是数据存储器，而是数据结构存储器。那是因为Redis支持客户端直接往里面塞各种类型的数据结构，比如String、List、Set、SortedSet、Map等等。你或许会问，这很了不起吗？我自己在Java里写一个HashTable不也可以放各种数据结构？呵呵，要知道你的HashTable只能放Java对象，人家那可是支持多语言的，不管你的客户端是Java还是Python还是别的，都可以往Redis塞数据结构。这一点也是Redis和Memcached相比，非常不同的一点。当然Redis要支持数据结构存储，是以牺牲更多内存为代价的，正所谓有利必有弊。关于Redis里头的数据结构，大家可以参考：[Redis Data Types](https://link.zhihu.com/?target=https%3A//redis.io/topics/data-types-intro)
- **剔除策略。**缓存数据总不能无限增长吧，总得剔除掉一些数据，好让新的缓存数据放进来吧？这就需要LRU算法了，大家可以参考：[Redis Lru Cache](https://link.zhihu.com/?target=https%3A//redis.io/topics/lru-cache)
- **负载均衡**。用到了集群，就免不了需要用到负载均衡，用什么负载均衡算法？在哪里使用负载均衡？这点大家可以参考：[Redis Partitioning](https://link.zhihu.com/?target=https%3A//redis.io/topics/partitioning)
- **Presharding。**如果一开始只有三台Redis服务器，后来发现需要加多一台才能满足业务需要，要怎么办？Redis提供了一种策略，叫：[Presharding](https://link.zhihu.com/?target=https%3A//redis.io/topics/partitioning%23presharding)
- **数据持久化。**如果我的机器突然全部断电了，我的缓存数据还能恢复吗？Redis说，相信我，可以的，不然我怎么用作数据库？去看看这个：[Redis Persistence](https://link.zhihu.com/?target=https%3A//redis.io/topics/persistence)
- **数据同步。**这篇文章里提到了主从复制，那么Redis是怎么进行主从复制的呢？根据CAP理论，既然我们已经选择了集群，也就是P，分区容忍性，那么剩下那两个，Consistency和Availability只能选择一个了，那么Redis到底是支持最终一致性还是强一致性呢？可以参考：[Redis Replication](https://link.zhihu.com/?target=https%3A//redis.io/topics/replication)





来自网页：https://zhuanlan.zhihu.com/p/37055648





