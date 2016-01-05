# The world beyond bacth: dataflow model
* 作者：张磊（<hbjerryzl@gmail.com>）

## 序
Tyler写了第一篇文章（[Streaming 101](https://github.com/hbjerry/blog/blob/master/streaming/the_world_beyond_batch.md)）,发现没有后续的第二篇，我根据几份文档提炼了第二篇的内容，可能没有他原创的好，但是系统能帮助大家理解。

这两篇是我主要的参考资料：
1. [The Dataflow Model: A Practical Approach to Balancing Correctness, Latency, and Cost in Massive-Scale, Unbounded, Out-of-Order Data Processing](http://www.vldb.org/pvldb/vol8/p1792-Akidau.pdf)
2. [Say Goodbay to Batch](https://docs.google.com/presentation/d/1WyFSdiPadm9rXnPiccOrwiv7Oo_fBaugJ5xlL6mWKaY/edit#slide=id.g43678ebc6_01567)

## Single Unified Model 统一模型
### 现有系统问题
1. Batch系统（比如MapReduce，Pig，Hive，FlumeJava，Spark）：延迟问题，需要等待所有数据到齐后处理
2. Aurora, TelegraphCQ, Niagara, Esper：异常处理不行
3. Storm, Samza, Pulsar：缺少exactly-once只有一次语义
4. Spark Streaming, Sonora, Trident：只提供对Processing Time分块
5. SQLStream：支持Event Time分块，但是要求数据有序
6. Stratosphere/Flink：有Event Time语义，但是只有有限的triggering，triggering是指在什么情况让下游感知计算结果，后面有详述
7. CEDR， Trill：有丰富的triggering和增量模式，但是分块语义不支持session
8. MillWheel and Spark Streaming：缺少更高的语言抽象，对Event time分session不是很友善
9. Pulsar：提供很好的分块语义，但是不提供正确性
10. Lambda架构：太负责，需要维护两套系统
11. Summingbird：抽象batch和streaming，而且实现不复杂，但也因此不支持某些操作，运维复杂度也高

现在越来越多的数据是无穷无序的，而且各种各样的需要也要求streaming系统提供更强的语义，所以我们需要一种更简单，并且能平衡延迟和正确性的工具。我们认为下面的模型能解决这个问题：

1. 灵活分块，由输入数据控制分块策略，不只是按时间分
2. 所有数据处理都能自己控制下面4个维度
   > What: 什么样计算结果，计算逻辑
   
   > When: 何时开始计算逻辑
   
   > Where：何时发布计算结果
   
   > How：迟到的数据是否会更新前面的结果
3. 分隔数据处理逻辑和物理实现，可以自己选择streaming，batch或者micro-batch来平衡正确性，延迟和开销。

