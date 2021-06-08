# Pulsar TGIP_01 Pulsar Basic
## Pub/Sub
### Producer
身为一个 Pub/Sub 系统，首先的存在要素必然是 Producer（生产者）。何为 producer？即消息生产方，所有消息调用生产方的接口，来将消息发送给 Pulsar。
![](https://mmbiz.qpic.cn/mmbiz_png/vOensVS3icTohMjCs2ribkVWHvbjQ0F3Ml3nlerz10IMHMVnp3ueh8YYkaZDZySofL4H8ldBHWktVrh6CFevgqXQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

Producer 的作用是与本身的应用程序有关。Producer 往 Pulsar 里发送消息时，相应的数据会带上 schema 的信息。Pulsar 会确保一个 producer 往
topic 发送的消息是满足一定的 schema 格式。

### Topic
![](https://mmbiz.qpic.cn/mmbiz_png/vOensVS3icTohMjCs2ribkVWHvbjQ0F3MlPHqKngicNDQ7KA6EOL2ENW8mCIsj92Pfqgjd1tNiaG80ccRhqM0YGKaw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

上文提到了，producer 会往 topic 里发送消息。那什么是 topic 呢？它是一个消息的集合，所有生产者的消息，都会归属到指定的 topic 里。所有在 topic
里的消息，会按照一定的规则，被切分成不同的分区（Partition）。一个分区会落靠在某一个服务器上，原理类似于 Kafka Topic Partition。

### Broker
分区落靠的服务器，就是 Broker。Broker 用来接收与发送消息，生产方连接到 broker 去生产消息，消费方连接到 broker 去消费消息。
![](https://mmbiz.qpic.cn/mmbiz_png/vOensVS3icTohMjCs2ribkVWHvbjQ0F3Ml5wKjWm9p5pmJ6BHtVkW9ibCiaZWL1bbXM6uXb7sps9ziboOouXPT6QN1A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

数据不会真正存储在 broker，这就是 Pulsar 与其他中间栈的区别：Pulsar 里的 broker 是没有存储状态的。

### Subscription
生产者产出的消息到了某一个 broker 上，就应该出现另一端的消费者（Consumer）了。

Consumer 作为消息的接收方，连接到 broker 接收消息。在 Pulsar 里将 consumer 接收消息的过程称之为：
- Subscription（订阅），类似于 Kafka 的 consumer group。一个订阅里的所有 consumer，会作为一个整体去消费这个 topic 里的所有消息。

#### Subscription Mode
Pulsar 里每一个订阅都会有不同的模式。目前 Pular 的订阅模式主要是以下四种：
- Exclusive（独占订阅）：不管有多少个 consumer 同时存在，只会有一个 consumer 是活跃的，也就是只有这一个 consumer 可以接收到这个 topic 的所有消息。
- Failover（故障转移订阅）：多个 consumer 可以附加到同一订阅。但是，对于给定的主题分区，将选择一个 consumer 作为该主题分区的主使用者，其他 consumer 将被指定为故障转移消费者，当主消费者断开连接时，分区将被重新分配给其中一个故障转移消费者，而新分配的消费者将成为新的主消费者。发生这种情况时，所有未确认的消息都将传递给新的主消费者，这类似于 Apache Kafka 中的使用者分区重新平衡。
- Shared（共享订阅）：将所需数量的 consumer 附加到同一订阅。消息以多个 consumer 的循环尝试分发形式传递，并且任何给定的消息仅传递给一个 consumer。当消费者断开连接时，所有传递给它并且未被确认的消息将被重新安排，以便发送给该订阅上剩余的 consumer。
- Key_Shared（Key 保序共享订阅）：类似于共享订阅，但又不是按照循环模式，是按照 key 进行分发，比如同一特征（奇数、偶数等）。总的来说是融合了 Failover 的有序性和 Shared 的消费扩展性、更均衡的一种订阅模式。

### Partition
Pulsar 给用户提供了 partition 的逻辑抽象，底层物理存储将逻辑的 partition 划分为多个分片（Segment），均匀存储在所有节点上。利用 Apache BookKeeper 的存储，按照一定的规则生成新的 segment，比较灵活。

Segment 由多条 entry 组成，entry 就是真正意义上组织和存储的力度，entry 里是由更多的消息（Message）通过匹配进行批量组成的。
![](https://mmbiz.qpic.cn/mmbiz_png/vOensVS3icTohMjCs2ribkVWHvbjQ0F3MlDl8Dn9GbUyRKFqrd2h6rWdfSWxxuiboWWVabpMHSGhyox9t2YKZykwA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

而最底层的 message 通常包含 Message ID，字段则一般由这几个：ledger-id（在哪个 segment）、entry-id（entry 在这个 segment 的位置）、batch-id（消息被匹配后的位置）、partition-index（消息在 topic 的哪个 partition）。
![](https://mmbiz.qpic.cn/mmbiz_png/vOensVS3icTohMjCs2ribkVWHvbjQ0F3MljELT36vLCtrnJ5hrE5giaCNpibqAbwK3JQm9GM3nAuDXRR8DicazPWvew/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### Cursor
Cursor 在消费者端，代表了每个订阅组的消费状态。所有订阅状态的管理，都给到了 broker，追踪每个订阅消费到了哪里，并存储到 cursor。然后提供到客户端接口 Acknowledge Cumulatively，后续再进行相应的操作，比如移动或重置。
![](https://mmbiz.qpic.cn/mmbiz_png/vOensVS3icTohMjCs2ribkVWHvbjQ0F3MlT9dU9yqOoAiaZqfBTc8Ndr7xR97AANib1kQxP4FNyM6w4KhpELWhicp2Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### Reader 
Cursor 相当于 Pulsar 管理你的消费状态，但是有时你不需要 Pulsar 帮你管理，所以就引入了 Reader 这个概念（Non-durable Cursor），即它的消费状态不被持久化的消费者进行消费。它的消费状态只在内存里出现，比如当服务器重启时不会丢失。

![](https://mmbiz.qpic.cn/mmbiz_png/vOensVS3icTohMjCs2ribkVWHvbjQ0F3MlrMHGWLLIqDRiaNvl0Niblibtb4rVia22jfpIYu51HZd1V1KRdsOuRBdvVg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### Tenant & Namespace
Pulsar 与其他中间件还不一样的地方，就是它是一个层级化管理结构，也就是 Tenant，Tenant 下有 namespace（命名空间），然后再往下走就是 topic。
![](https://mmbiz.qpic.cn/mmbiz_png/vOensVS3icTohMjCs2ribkVWHvbjQ0F3MlKicD2EncI1D90Hf6ZDdz23VpkGtKYdxoZibzmFVetqL9Su9W7X69qCsQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

Pulsar 为了支持统一化的消息平台，引入了 Topic Domain 的概念，默认消息是可持久化的。所以通过这种层级化结构，可以使 Pulsar 更适配不同的应用场景。



