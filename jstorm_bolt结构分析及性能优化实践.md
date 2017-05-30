##jstorm Bolt结构分析及性能优化实践

###1、Jstorm Bolt结构示意图
![Mou icon](file:///Users/xiaoleixl/Documents/单Bolt模型.jpg)

###性能问题总体原则
* 大部分情况下系统的IO是处理的瓶颈，减少IO是重中之重。
* jstorm流计算拓扑深度需要控制，越短越好，减少bolt间的IO量。
* 减少数据库调用次数和rpc调用次数，减少外部缓存调用次数，推荐使用guava等内部缓存，以减少IO量。
* 对于精确性要求不高的场景，不使用acker机制，以减少IO量。


###性能调优经验总结
* 由于默认设置netty-server和netty-client线程为1个，在流量大的时候，netty的socket有时很难read和write完。表现：出现batch_trans_time监控项巨大。解决方案：storm.messaging.netty.server_worker_threads和storm.messaging.netty.client_worker_threads参数设置调大。

* 由于一个worker只有1个反序列化和序列化线程，而很有可能1个worker承接用户多个并发，导致数据序列化反序列化太慢。表现：出现序列化反序列化队列满。解决方案：2.2版本以上jstorm可以配置序列化反序列化线程数量。（若出现此现象优先查看是否其他原因导致）

* 用户自己写的代码中有性能问题。表现：执行队列满。解决方案：并发当中高频使用部分使用线程池。数据库连接，rpc请求使用高性能缓存。

* 使用fieldsGrouping时出现巨大的数据倾斜，导致并发中使用线程池也不能满足需求。表现：Bolt下各个并发recv tps数量相差巨大。解决方案：

* 数据库连接一定要设置connecttimeout和sockettimeout参数，以免导致dead线程。

* 内存old区内存占满，频繁触发fullgc从而性能大幅下降。表现：进程old区内存占满，fullgc频繁触发。通过jstat -gcutil pid 1s 100来观测gc情况。解决方案：属于内存泄漏，jmap -histo:live pid查询内存占据最多的类，并分析原因，如果业务需要占据更多内存设置worker.memory.size来设置提高worker内存数量；同时设置pending num以限流。

* 消费速度周期性变慢，这是由于缓存过期时间一致导致。需要每个并发的本地缓存过期时间错开，增加一个随机数。