# 死磕Flink - Time深度解析 章
## Flink时间语义
在不同的应用场景中时间语义是各不相同的，Flink 作为一个先进的分布式流处理引擎，它本身支持不同的时间语义。其核心是 Processing Time 和 Event Time（Row Time），这两类时间主要的不同点如下表所示：

![](https://mmbiz.qpic.cn/mmbiz_png/8AsYBicEePu7uXY232qxDpYFIDFiaABrC6sIgFWSAB5tg2MqOMiavYibXXKuAp73XIlAvO1iaqPJ8eRSeAGLyK966mw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

Processing Time 是来模拟我们真实世界的时间，其实就算是处理数据的节点本地时间也不一定是完完全全的真实世界的时间，所以说它是用来模拟真实世界的时间。而 Event Time 是数据世界的时间，即我们要处理的数据流世界里的时间。关于他们的获取方式，Process Time 是通过直接去调用本地机器的时间，而 Event Time 则是根据每一条处理记录所携带的时间戳来判定。

因此在判断应该使用 Processing Time 还是 Event Time 的时候，可以遵循一个原则：当你的应用遇到某些问题要从上一个 checkpoint 或者 savepoint 进行重放，是不是希望结果完全相同。如果希望结果完全相同，就只能用 Event Time；如果接受结果不同，则可以用 Processing Time。Processing Time 的一个常见的用途是，根据现实时间来统计整个系统的吞吐，比如要计算现实时间一个小时处理了多少条数据，这种情况只能使用 Processing Time。

