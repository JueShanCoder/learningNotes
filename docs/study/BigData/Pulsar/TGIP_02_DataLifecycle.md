# Pulsar TGIP_01 DataLifecycle
## Brokers + Bookies
前边我们提到过，broker是各零件之间进行交换的对象。
因 Pulsar 为分层架构模式，使用了 BookKeeper 作为额外的存储系统，bookies 就是 BookKeeper 里的存储节点。

Brokers+Bookies 构成 Pulsar 的两个层次，共同完成了 Pulsar 的数据服务。

> Broker：整个消息层的生产和消费，无存储状态 
> Bookie：数据持久化保存的节点，有存储状态

## 数据流动过程
Producer 通过生产消息到一个 topic ，一个 topic 中可能有 N 个 partition，每个 partition 给一个 broker 服务；那 consumer 会根据要消费的 topic 以及具体的 partition 找到所服务的 broker 并进行消费。

![](https://mmbiz.qpic.cn/mmbiz_png/vOensVS3icTqib3ibLSStAkvJHBdxdGoES8SqwTvJFGKkd6jtB9dDp2kibQElQicL7AIEvTnibZKAGP3bY7Ejl4CZhIw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### Write Path（写过程）
数据的生产是怎么发生的呢？首先客户端会调用 Pulsar 提供给客户端的 API，这就是 producer。创建消息后，可以放进 pillow 里，也可以指定 key，放在一条消息里面，这条消息通过应用客户端传给 producer。

生产端的内部有一个叫 MessageWriter 的类，这个 MessageWriter 默认是 round-robin 的过程，相当于每发一条消息会轮询的去找一个 partition 去发送，为了提高效率，在一定的时间内只会选择一个 partition。

![](https://mmbiz.qpic.cn/mmbiz_png/vOensVS3icTqib3ibLSStAkvJHBdxdGoES8Zac099svN3QMrGRCWNfutg67V4aKcm9lBN63fZ5ibGhNhgdlHicGMibEA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

一旦 partition 被选择出来，客户端会跟 broker 打交道，找出这个 partition 究竟是为哪个 broker 服务，这个寻找的过程叫做 Topic Discovery。Broker 收到消息后会调用 BookKeeper 的客户端并发去写多个副本。

整个并发写的流程是被封装到 BookKeeper 的客户端中，所以可以认为是 broker 调用 BookKeeper 的客户端去写 bookies，当 broker 收到了两个副本的 ACK 之后它会认为这条消息已经写成功，broker 会返回客户端，告知这条消息已经被持久化完成。

整个写的过程消息是从 producer 写到 broker，然后经过 broker 到 BookKeeper 上。整个过程中客户端都不会跟 ZooKeeper 打交道，你也不会跟 BookKeeper 打交道，唯一打交道的只有 broker。


