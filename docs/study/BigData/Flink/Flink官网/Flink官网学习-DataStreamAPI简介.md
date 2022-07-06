# Java tuples 和 POJOs
Flink 的原生序列化器可以高效地操作 tuples 和 POJOs
## Tuples 
对于 Java，Flink 自带有 Tuple0 到 Tuple25 类型。
```java
Tuple2<String, Integer> person = Tuple2.of("Fred", 35);

// zero based index!  
String name = person.f0;
Integer age = person.f1;
```
## POJOs
如果满足以下条件，Flink 将数据类型识别为 POJO 类型（并允许“按名称”字段引用）：

- 该类是公有且独立的（没有非静态内部类）
- 该类有公有的无参构造函数
- 类（及父类）中所有的所有不被 static、transient 修饰的属性要么是公有的（且不被 final 修饰），要么是包含公有的 getter 和 setter 方法，这些方法遵循 Java bean 命名规范。

# Stream执行环境
每个 Flink 应用都需要有执行环境，在该示例中为 env。流式应用需要用到 StreamExecutionEnvironment。

DataStream API 将你的应用构建为一个 job graph，并附加到 StreamExecutionEnvironment 。当调用 env.execute() 时此 graph 就被打包并发送到 JobManager 上，后者对作业并行处理并将其子任务分发给 Task Manager 来执行。每个作业的并行子任务将在 task slot 中执行。

注意，如果没有调用 execute()，应用就不会运行。

![](https://ci.apache.org/projects/flink/flink-docs-release-1.12/fig/distributed-runtime.svg)


