# 減少批次資料的執行時間

在Spark中有幾個優化可以減少批次處理的時間。這些可以在[優化指南](../../other/tuning-spark.md)中作了討論。這節重點討論幾個重要的。

## 資料接收的平行化程度

藉由網路(如kafka，flume，socket等)接收資料需要這些資料反序列化並被保存到Spark中。如果資料接收成為系统的瓶頸，就要考慮平行地接收資料。注意，每個輸入DStream創建一個`receiver`（運行在worker機器上）
接收單個資料串流。創建多個輸入DStream並配置它們可以從來源中接收不同分區的資料串流，從而實作多資料串流接收。例如，接收兩個topic資料的單個輸入DStream可以被切分為兩個kafka輸入串流，每個接收一個topic。這將
在兩個worker上運行兩個`receiver`，因此允許資料平行接收，提高整體的吞吐量。多個DStream可以被合並生成單個DStream，這樣運用在單個輸入DStream的transformation操作可以運用在合並的DStream上。

```scala
val numStreams = 5
val kafkaStreams = (1 to numStreams).map { i => KafkaUtils.createStream(...) }
val unifiedStream = streamingContext.union(kafkaStreams)
unifiedStream.print()
```

另外一個需要考慮的參數是`receiver`的阻塞時間。對於大部分的`receiver`，在存入Spark記憶體之前，接收的資料都被合並成了一個大資料區塊。每批次資料中區塊的個數決定了任務的個數。這些任務是用類
似map的transformation操作接收的資料。阻塞間隔由配置參數`spark.streaming.blockInterval`決定，預設的值是200毫秒。

多輸入串流或者多`receiver`的可選的函數是明確地重新分配輸入資料串流（利用`inputStream.repartition(<number of partitions>)`），在進一步操作之前，藉由集群的機器數分配接收的批次資料。

## 資料處理的平行化程度

如果運行在計算stage上的並發任務數不足夠大，就不會充分利用集群的資料來源。例如，對於分散式reduce操作如`reduceByKey`和`reduceByKeyAndWindow`，預設的並發任務數藉由配置属性來確定（configuration.html#spark-properties）
`spark.default.parallelism`。你可以藉由參數（`PairDStreamFunctions` (api/scala/index.html#org.apache.spark.streaming.dstream.PairDStreamFunctions)）傳遞平行度，或者設定參數
`spark.default.parallelism`修改預設值。

## 資料序列化

資料序列化的總開销是平常大的，特别是當sub-second級的批次資料被接收時。下面有兩個相關點：

- Spark中RDD資料的序列化。關於資料序列化請参照[Spark優化指南](../../other/tuning-spark.md)。注意，與Spark不同的是，預設的RDD會被持續化為序列化的位元組陣列，以減少與垃圾回收相關的暫停。
- 輸入資料的序列化。從外部獲取資料存到Spark中，獲取的byte資料需要從byte反序列化，然後再按照Spark的序列化格式重新序列化到Spark中。因此，輸入資料的反序列化花費可能是一個瓶頸。

## 任務的啟動開支

每秒鐘啟動的任務數是非常大的（50或者更多）。發送任務到slave的花費明顯，這使請求很難獲得亞秒（sub-second）級别的反應。藉由下面的改變可以減小開支

- 任務序列化。運行kyro序列化任何可以減小任務的大小，從而減小任務發送到slave的時間。
- 執行模式。在Standalone模式下或者粗粒度的Mesos模式下運行Spark可以在比細粒度Mesos模式下運行Spark獲得更短的任務啟動時間。可以在[在Mesos下運行Spark](../../deploying/running-spark-on-mesos.md)中獲取更多訊息。

These changes may reduce batch processing time by 100s of milliseconds, thus allowing sub-second batch size to be viable.

