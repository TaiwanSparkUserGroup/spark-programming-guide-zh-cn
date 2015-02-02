# 輸入DStreams和receivers

輸入DStreams表示從資料來源獲取輸入資料串流的DStreams。在[快速例子](../a-quick-example.md)中，`lines`表示輸入DStream，它代表從netcat服務器獲取的資料串流。每一個輸入串流DStream
和一個`Receiver`對象相連接，這個`Receiver`從來源中獲取資料，並將資料存入記憶體中用於處理。

輸入DStreams表示從資料來源獲取的原始資料串流。Spark Streaming擁有兩類資料來源
- 基本來源（Basic sources）：這些來源在StreamingContext API中直接可用。例如檔案系統、socket連接、Akka的actor等。
- 高級來源（Advanced sources）：這些來源包括Kafka,Flume,Kinesis,Twitter等等。它們需要藉由额外的類來使用。我們在[連接](linking.md)那一節討論了類Dependencies。

需要注意的是，如果你想在一個串流應用中平行地創建多個輸入DStream來接收多個資料串流，你能夠創建多個輸入串流（這將在[性能調教](../performance-tuning/README.md)那一節介紹）
。它將創建多個Receiver同時接收多個資料串流。但是，`receiver`作為一個長期運行的任務運行在Spark worker或executor中。因此，它占有一個核，這個核是分配给Spark Streaming應用程式的所有
核中的一個（it occupies one of the cores allocated to the Spark Streaming application）。所以，為Spark Streaming應用程式分配足夠的核（如果是本地運行，那麼是執行緒）
用以處理接收的資料並且運行`receiver`是非常重要的。

幾點需要注意的地方：
- 如果分配给應用程式的核的數量少於或者等於輸入DStreams或者receivers的數量，系统只能夠接收資料而不能處理它們。
- 當運行在本地，如果你的master URL被設定成了“local”，這樣就只有一個核運行任務。這對程式來說是不足的，因為作為`receiver`的輸入DStream將會占用這個核，這樣就没有剩餘的核來處理資料了。

## 基本來源

我們已經在[快速例子](../a-quick-example.md)中看到，`ssc.socketTextStream(...)`函數用來把從TCPsocket獲取的文本資料創建成DStream。除了socket，StreamingContext API也支援把文件
以及Akka actors作為輸入來源創建DStream。

- 檔案串流（File Streams）：從任何與HDFS API兼容的檔案系統中讀取資料，一個DStream可以藉由如下方式創建

```scala
streamingContext.fileStream[keyClass, valueClass, inputFormatClass](dataDirectory)
```
Spark Streaming將會監控`dataDirectory`目錄，並且處理目錄下生成的任何文件（嵌套目錄不被支援）。需要注意一下三點：

    1 所有文件必須具有相同的資料格式
    2 所有文件必須在`dataDirectory`目錄下創建，文件是自動的移動和重命名到資料目錄下
    3 一旦移動，文件必須被修改。所以如果文件被持續的附加資料，新的資料不會被讀取。

對於簡單的文本文件，有一個更簡單的函數`streamingContext.textFileStream(dataDirectory)`可以被調用。檔案串流不需要運行一個receiver，所以不需要分配核。

在Spark1.2中，`fileStream`在Python API中不可用，只有`textFileStream`可用。

- 基於自定義actor的串流：DStream可以調用`streamingContext.actorStream(actorProps, actor-name)`函數從Akka actors獲取的資料串流來創建。具體的訊息見[自定義receiver指南](https://spark.apache.org/docs/latest/streaming-custom-receivers.html#implementing-and-using-a-custom-actor-based-receiver)
`actorStream`在Python API中不可用。
- RDD隊列作為資料串流：為了用測試資料測試Spark Streaming應用程式，人們也可以調用`streamingContext.queueStream(queueOfRDDs)`函數基於RDD隊列創建DStreams。每個push到隊列的RDD都被
當做DStream的批次資料，像串流一樣處理。

關於從socket、文件和actor中獲取串流的更多細節，請看[StreamingContext](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.streaming.StreamingContext)和
[JavaStreamingContext](https://spark.apache.org/docs/latest/api/java/index.html?org/apache/spark/streaming/api/java/JavaStreamingContext.html)

## 高級來源

這類來源需要非Spark函式庫介面，並且它們中的部分還需要複雜的Dependencies（例如kafka和flume）。為了減少Dependencies的版本冲突问题，從這些來源創建DStream的功能已經被移到了獨立的函式庫中，你能在[連接](linking.md)查看
細節。例如，如果你想用來自推特的流資料創建DStream，你需要按照如下步骤操作：

- 連接：添加`spark-streaming-twitter_2.10`到SBT或maven項目的Dependencies中
- 編寫：導入`TwitterUtils`類，用`TwitterUtils.createStream`函數創建DStream,如下所示
```scala
import org.apache.spark.streaming.twitter._
TwitterUtils.createStream(ssc)
```
- 部署：將編寫的程式以及其所有的Dependencies（包括spark-streaming-twitter_2.10的Dependencies以及它的傳遞Dependencies）包裝為jar檔，然後部署。這在[部署章節](deploying-applications.md)將會作更進一步的介紹。

需要注意的是，這些高級的來源在`spark-shell`中不能被使用，因此基於這些來源的應用程式無法在shell中測試。

下面將介紹部分的高級來源：

- Twitter：Spark Streaming利用`Twitter4j 3.0.3`獲取公共的推文流，這些推文藉由[推特流API](https://dev.twitter.com/docs/streaming-apis)獲得。認證訊息可以藉由Twitter4J函式庫支援的
任何[函數](http://twitter4j.org/en/configuration.html)提供。你既能夠得到公共串流，也能夠得到基於關键字過濾後的串流。你可以查看API文件（[scala](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.streaming.twitter.TwitterUtils$)和[java](https://spark.apache.org/docs/latest/api/java/index.html?org/apache/spark/streaming/twitter/TwitterUtils.html)）
和例子（[TwitterPopularTags](https://github.com/apache/spark/blob/master/examples/src/main/scala/org/apache/spark/examples/streaming/TwitterPopularTags.scala)和[TwitterAlgebirdCMS](https://github.com/apache/spark/blob/master/examples/src/main/scala/org/apache/spark/examples/streaming/TwitterAlgebirdCMS.scala)）
- Flume：Spark Streaming 1.2能夠從flume 1.4.0中獲取資料，可以查看[flume整合指南](flume-integration-guide.md)了解詳細訊息
- Kafka：Spark Streaming 1.2能夠從kafka 0.8.0中獲取資料，可以查看[kafka整合指南](kafka-integration-guide.md)了解詳細訊息
- Kinesis：查看[Kinesis整合指南](kinesis-integration.md)了解詳細訊息

## 自定義來源

在Spark 1.2中，這些來源不被Python API支援。
輸入DStream也可以藉由自定義來源創建，你需要做的是實作用戶自定義的`receiver`，這個`receiver`可以從自定義來源接收資料以及將資料推到Spark中。藉由[自定義receiver指南](custom-receiver.md)了解詳細訊息

## Receiver可靠性

基於可靠性有兩類資料來源。來源(如kafka、flume)允許。如果從這些可靠的來源獲取資料的系统能夠正確的響應所接收的資料，它就能夠確保在任何情況下不丢失資料。這樣，就有兩種類型的receiver：

- Reliable Receiver：一個可靠的receiver正確的響應一個可靠的來源，資料已經收到並且被正確地複製到了Spark中。
- Unreliable Receiver ：這些receivers不支援響應。即使對於一個可靠的來源，開發者可能實作一個非可靠的receiver，這個receiver不會正確響應。

怎樣編寫可靠的Receiver的細節在[自定義receiver](custom-receiver.md)中有詳細介紹。
