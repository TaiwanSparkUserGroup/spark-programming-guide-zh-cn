# 連接

與Spark類似，Spark Streaming也可以利用maven函式庫。編寫你自己的Spark Streaming程式，你需要引入下面的Dependencies到你的SBT或者Maven項目中

```maven
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-streaming_2.10</artifactId>
    <version>1.2</version>
</dependency>
```
為了從Kafka, Flume和Kinesis這些不在Spark核心API中提供的來源獲取資料，我們需要添加相關的模區塊`spark-streaming-xyz_2.10`到Dependencies中。例如，一些通用的组件如下表所示：

Source | Artifact
--- | ---
Kafka | spark-streaming-kafka_2.10
Flume | spark-streaming-flume_2.10
Kinesis | spark-streaming-kinesis-asl_2.10
Twitter | spark-streaming-twitter_2.10
ZeroMQ | spark-streaming-zeromq_2.10
MQTT | spark-streaming-mqtt_2.10

為了獲取最新的列表，請訪問[Apache repository](http://search.maven.org/#search%7Cga%7C1%7Cg%3A%22org.apache.spark%22%20AND%20v%3A%221.2.0%22)
