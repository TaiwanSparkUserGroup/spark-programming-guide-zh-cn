# Kinesis整合指南

Amazon Kinesis是一個即時處理大規模串流資料的全托管服務。Kinesis receiver應用Kinesis客戶端函式庫（KCL）創建一個輸入DStream。KCL由Amazon 提供，它擁有Amazon 软件許可证(ASL)。KCL构建在
apache 2.0許可的AWS java SDK之上，藉由Workers、檢查點(Checkpoints)和Shard Leases等概念提供了負載均衡、容錯、檢查點機制。下面將詳細介紹怎樣配置Spark Streaming從Kinesis獲取
資料。

## 配置Kinesis

一個Kinesis串流可以通用一個擁有一個或者多個shard的有效Kinesis端點(endpoint)來建立，詳情請見[指南](http://docs.aws.amazon.com/kinesis/latest/dev/step-one-create-stream.html)

## 配置Spark Streaming應用程式

1、連接：在你的SBT或者Maven項目定義中，引用下面的组件到串流應用程式中

```
 groupId = org.apache.spark
 artifactId = spark-streaming-kinesis-asl_2.10
 version = 1.1.1
```
需要注意的是，連接這個artifact，你必須將[ASL](https://aws.amazon.com/asl/)認證程式碼加入到你的應用程式中。

2、編程：在你的應用程式程式碼中，藉由引入`KinesisUtils`創建DStream

```scala
 import org.apache.spark.streaming.Duration
 import org.apache.spark.streaming.kinesis._
 import com.amazonaws.services.kinesis.clientlibrary.lib.worker.InitialPositionInStream
 val kinesisStream = KinesisUtils.createStream(
 	streamingContext, [Kinesis stream name], [endpoint URL], [checkpoint interval], [initial position])
```

可以查看[API文件](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.streaming.kinesis.KinesisUtils$)和[例子](https://github.com/apache/spark/tree/master/extras/kinesis-asl/src/main/scala/org/apache/spark/examples/streaming/KinesisWordCountASL.scala)。

  - streamingContext：streamingContext包含一個應用程式名稱，這個名稱連接Kinesis應用程式和Kinesis串流。
  - Kinesis stream name：這個串流應用程式的Kinesis串流獲取名稱滿足一下幾點：
    - 在流StreamingContext中使用的應用程式名稱可以作為Kinesis應用程式名稱
    - 對於某個地區的同一帳戶，應用程式名稱必須唯一
    - Kinesis的後端藉由一個DynamoDB表（一般情況下在us-east-1 region）自動的連接應用程式名稱和Kinesis串流，這個DynamoDB表由Kinesis串流初始化
    - 在某些情況下，改變應用程式名稱或者流名稱可能導致Kinesis錯誤，如果你發现了錯誤，你可能需要手動删除DynamoDB表
  - endpoint URL：合法的Kinesis endpoint URL能夠在[這裡](http://docs.aws.amazon.com/general/latest/gr/rande.html#ak_region)找到。
  - checkpoint interval：KCL在流中保存檢查點位置的時間間隔，對於初学者，可以將其和串流應用程式的批時間間隔設定得一致。
  - initial position：可以是`InitialPositionInStream.TRIM_HORIZON`也可以是`InitialPositionInStream.LATEST`(可以查看Kinesis checkpoint和Amazon Kinesis API文件了解詳細訊息)

3、部署：將`spark-streaming-kinesis-asl_2.10`和它的Dependencies（除了`spark-core_2.10`和`spark-streaming_2.10`）打包到應用程式的jar包中。然後用`spark-submit`函數啟動你的應用程式。

在運行過程中需要注意一下幾點:

  - Kinesis的每個分區的資料處理都是有序的，每一条訊息至少出現一次
  - 多個應用程式可以從相同的Kinesis串流讀取資料，Kinesis將會保存特定程式的shard和checkpoint到DynamodDB中
  - 在某一時間單個的Kinesis串流shard只能被一個輸入DStream處理
  - 單個的Kinesis DStream藉由創建多個`KinesisRecordProcessor`執行緒，可以從Kinesis串流的多個shard中讀取資料
  - 分開運行在不同的processes或者instances中的多個輸入DStream能夠從Kinesis串流中讀到
  - Kinesis輸入DStream的數量不應比Kinesis shard的數量多，這是因為每個輸入DStream都將創建至少一個`KinesisRecordProcessor`執行緒去處理單個的shard
  - 藉由添加或者删除DStreams（在單個處理器或者多個processes/instance之間）可以獲得水平擴充，直到擴充到Kinesis shard的數量。
  - Kinesis輸入DStream將會平衡所有DStream的負載，甚至是跨processes/instance的DStream
  - Kinesis輸入DStream將會平衡由於變化引起的re-shard事件（合並和切分）的負載
  - 作為一個最佳實踐，建議避免使用過度的re-shard
  - 每一個Kinesis輸入DStream都包含它們自己的checkpoint訊息
  - Kinesis串流shard的數量與RDD分區（在Spark輸入DStream處理的過程中產生）的數量之間没有關系。它們是兩種獨立的分區模式

![Spark流Kinesis架構](../../img/streaming-kinesis-arch.png)

## 運行實例

- 下載Spark 來源程式碼，然後按照下面的函數build Spark。
```
mvn -Pkinesis-asl -DskipTests clean package
```
- 在AWS中設定Kinesis串流。注意Kinesis串流的名字以及endpoint URL與流創建的地區相連接
- 在你的AWS证书中設定`AWS_ACCESS_KEY_ID`和`AWS_SECRET_KEY`環境變量
- 在Spark根目錄下面，運行例子

```
bin/run-example streaming.KinesisWordCountASL [Kinesis stream name] [endpoint URL]
```
這個例子將會等待從Kinesis串流中獲取資料

- 在另外一個终端，為了生成生成隨機的字串資料到Kinesis串流中，運行相關的Kinesis資料生產者
```
bin/run-example streaming.KinesisWordCountProducerASL [Kinesis stream name] [endpoint URL] 1000 10
```
這步將會每秒推送1000行，每行带有10個隨機數字的資料到Kinesis串流中，這些資料將會被運行的例子接收和處理

## Kinesis Checkpointing

- 每一個Kinesis輸入DStream定期的儲存流的當前位置到後台的DynamoDB表中。這允許系统從錯誤中恢復，继續執行DStream留下的任務。
- Checkpointing太频繁將會造成AWS檢查點儲存層過載，並且可能導致AWS節流(throttling)。提供的例子藉由隨機回退重试(random-backoff-retry)策略解决這個節流问题
- 當輸入DStream啟動時，如果没有Kinesis checkpoint訊息存在。它將會從最老的可用的紀錄（InitialPositionInStream.TRIM_HORIZON）或者最近的紀錄（InitialPostitionInStream.LATEST）啟動。
- 如果資料添加到流中的時候還没有輸入DStream在運行，InitialPositionInStream.LATEST可能導致丢失紀錄。
- InitialPositionInStream.TRIM_HORIZON可能導致紀錄的重複處理，這個錯誤的影響Dependencies於checkpoint的频率以及處理的幂等性。
