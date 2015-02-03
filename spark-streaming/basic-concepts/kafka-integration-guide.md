# kafka整合指南

[Apache kafka](http://kafka.apache.org/)是一個分散式的發布-订阅訊息系统，它可以分散式的、可分區的、可重複提交的方式讀寫日誌資料。下面我們將具體介紹Spark Streaming怎樣從kafka中
接收資料。

- 連接：在你的SBT或者Maven項目定義中，引用下面的组件到串流應用程式中。

```
 groupId = org.apache.spark
 artifactId = spark-streaming-kafka_2.10
 version = 1.1.1
```

- 編程：在應用程式程式碼中，引入`FlumeUtils`創建輸入DStream。

```scala
 import org.apache.spark.streaming.kafka._
 val kafkaStream = KafkaUtils.createStream(
 	streamingContext, [zookeeperQuorum], [group id of the consumer], [per-topic number of Kafka partitions to consume])
```

有兩點需要注意的地方：

   1. kafka的topic分區(partition)和由Spark Streaming生成的RDD分區不相關。所以在`KafkaUtils.createStream()`函數中，增加特定topic的分區數只能夠增加單個`receiver`消費這個
    topic的執行緒數，不能增加Spark處理資料的並發數。

   2. 藉由不同的group和topic，可以創建多個輸入DStream，從而利用多個`receiver`並發的接收資料。

- 部署：將`spark-streaming-kafka_2.10`和它的Dependencies（除了`spark-core_2.10`和`spark-streaming_2.10`）打包到應用程式的jar檔中。然後用`spark-submit`函數啟動你的應用程式。