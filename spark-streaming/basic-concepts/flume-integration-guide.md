# flume整合指南

flume是一個分散式的、穩定的、有效的服務，它能夠高效的收集、聚集以及移動大量的日誌資料。flume的架構如下圖所示。

![flume](../../img/flume.png)

本節將介紹怎樣配置flume以及Spark Streaming如何從flume中接收資料。主要有兩種函數可用。
## 函數一：基於推送的flume風格的函數

flume被設計用來在flume agent間推送資料。在這個函數中，Spark Streaming本質上是建立一個`receiver`,這個`receiver`充當一個Avro代理，用於接收flume推送的資料。下面是配置的過程。

### 一般需求

在你的集群中，選擇一台滿足下面條件的機器：

- 當你的flume和Spark Streaming應用程式啟動以後，必須有一個Spark worker運行在這台機器上面
- 可以配置flume推送資料到這台機器的某個端口

因為是推送模式，安排好`receiver`並且監聽選中端口的串流應用程式必須是開啟的，以使flume能夠發送資料。

### 配置flume

藉由下面的配置文件配置flume agent，發送資料到一個Avro sink。

```
agent.sinks = avroSink
agent.sinks.avroSink.type = avro
agent.sinks.avroSink.channel = memoryChannel
agent.sinks.avroSink.hostname = <chosen machine's hostname>
agent.sinks.avroSink.port = <chosen port on the machine>
```
查看[flume文件](https://flume.apache.org/documentation.html)了解更多的訊息。

### 配置Spark Streaming應用程式

- 連接：在你的SBT或者Maven項目定義中，引用下面的组件到串流應用程式中。
```
 groupId = org.apache.spark
 artifactId = spark-streaming-flume_2.10
 version = 1.1.1
```
- 編程：在應用程式程式碼中，引入`FlumeUtils`創建輸入DStream。
```
 import org.apache.spark.streaming.flume._
 val flumeStream = FlumeUtils.createStream(streamingContext, [chosen machine's hostname], [chosen port])
```
查看[API文件](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.streaming.flume.FlumeUtils$)和[例子](https://github.com/apache/spark/tree/master/examples/src/main/scala/org/apache/spark/examples/streaming/FlumeEventCount.scala)

注意，hostname必須和集群(Mesos,YARN或者Spark Standalone)的resource manager所使用的機器的hostname是同一個，這樣就可以根據名稱分配资料來源，在正確的機器上啟動`receiver`。

- 部署：將`spark-streaming-flume_2.10`和它的Dependencies（除了`spark-core_2.10`和`spark-streaming_2.10`）打包到應用程式jar包中。然後用`spark-submit`函數啟動你的應用程式。


## 函數2：利用自定義sink的基於pull的函數

作為直接推送資料到Spark Streaming函數的替代函數，這個函數運行一個自定義的flume sink用於滿足下面兩點功能。

- flume推送資料到sink，該資料被暫存在sink中。
- Spark Streaming利用transactions從sink中拉取資料。只有資料收到了並且被Spark Streaming複製了之後，transactions才算成功。這使得這個函數比之前的函數具有更好的穩定性和容錯性。然而，這個函數需要flume去
運行一個自定義sink。下面是配置的過程。

### 一般需求

選擇一台機器在flume agent中運行自定義的sink，配置餘下的flume管道(pipeline)發送資料到agent中。集群中的其它機器應該能夠訪問運行自定義sink的機器。

### 配置flume

在選定的機器上面配置flume需要以下兩個步骤。

- Sink Jars：添加下面的jar文件到flume的classpath目錄下面
    - 自定義sink jar：藉由下面的方式下載jar（或者[這裡](http://search.maven.org/remotecontent?filepath=org/apache/spark/spark-streaming-flume-sink_2.10/1.1.1/spark-streaming-flume-sink_2.10-1.1.1.jar)）
    ```
     groupId = org.apache.spark
     artifactId = spark-streaming-flume-sink_2.10
     version = 1.1.1
    ```
    - scala library jar:下載scala 2.10.4函式庫，你能夠藉由下面的方式下載(或者[這裡](http://search.maven.org/remotecontent?filepath=org/scala-lang/scala-library/2.10.4/scala-library-2.10.4.jar))
    ```
     groupId = org.scala-lang
     artifactId = scala-library
     version = 2.10.4
    ```
- 配置文件：藉由下面的配置文件配置flume agent用於發送資料到Avro sink。

```
 agent.sinks = spark
 agent.sinks.spark.type = org.apache.spark.streaming.flume.sink.SparkSink
 agent.sinks.spark.hostname = <hostname of the local machine>
 agent.sinks.spark.port = <port to listen on for connection from Spark>
 agent.sinks.spark.channel = memoryChannel
```
要確保配置的逆流flume管道（upstream Flume pipeline）運行這個sink發送資料到flume代理。

### 配置Spark Streaming應用程式

- 連接：在你的SBT或者Maven項目定義中，引入`spark-streaming-flume_2.10`组件
- 編程：在應用程式程式碼中，引入`FlumeUtils`創建輸入DStream。

```
 import org.apache.spark.streaming.flume._
 val flumeStream = FlumeUtils.createPollingStream(streamingContext, [sink machine hostname], [sink port])
```

可以查看用例[FlumePollingEventCount](https://github.com/apache/spark/tree/master/examples/src/main/scala/org/apache/spark/examples/streaming/FlumePollingEventCount.scala)

注意，每個輸入DStream都可以配置為從多個sink接收資料。

- 部署：將`spark-streaming-flume_2.10`和它的Dependencies（除了`spark-core_2.10`和`spark-streaming_2.10`）打包到應用程式的jar包中。然後用`spark-submit`函數啟動你的應用程式。



