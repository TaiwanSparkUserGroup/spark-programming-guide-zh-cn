# DStream中的轉換（transformation）

和RDD類似，transformation允許從輸入DStream來的資料被修改。DStreams支持很多在RDD中可用的transformation操作。一些常用的操作如下所示：

Transformation | Meaning
--- | ---
map(func) | 利用函數`func`處理原DStream的每個元素，返回一個新的DStream
flatMap(func) | 與map相似，但是每個輸入項可用被映射為0個或者多個輸出項
filter(func) | 返回一個新的DStream，它僅僅包含來源DStream中滿足函數func的項
repartition(numPartitions) | 藉由創建更多或者更少的partition改變這個DStream的並行級别(level of parallelism)
union(otherStream) | 返回一個新的DStream,它包含來源DStream和otherStream的联合元素
count() | 藉由計算來源DStream中每個RDD的元素數量，返回一個包含單元素(single-element)RDDs的新DStream
reduce(func) | 利用函數func聚集來源DStream中每個RDD的元素，返回一個包含單元素(single-element)RDDs的新DStream。函數應該是相連接的，以使計算可以並行化
countByValue() | 這個操作應用於元素類型為K的DStream上，返回一個（K,long）對的新DStream，每個键的值是在原DStream的每個RDD中的频率。
reduceByKey(func, [numTasks]) | 當在一個由(K,V)對組成的DStream上調用這個操作，返回一個新的由(K,V)對組成的DStream，每一個key的值均由给定的reduce函數聚集起來。注意：在預設情況下，這個操作利用了Spark預設的並發任務數去分组。你可以用`numTasks`參數設定不同的任務數
join(otherStream, [numTasks]) | 當應用於兩個DStream（一個包含（K,V）對,一個包含(K,W)對），返回一個包含(K, (V, W))對的新DStream
cogroup(otherStream, [numTasks]) | 當應用於兩個DStream（一個包含（K,V）對,一個包含(K,W)對），返回一個包含(K, Seq[V], Seq[W])的元组
transform(func) | 藉由對來源DStream的每個RDD應用RDD-to-RDD函數，創建一個新的DStream。這個可以在DStream中的任何RDD操作中使用
updateStateByKey(func) | 利用给定的函數更新DStream的狀態，返回一個新"state"的DStream。

最後兩個transformation操作需要重點介紹一下：

## UpdateStateByKey操作


updateStateByKey操作允許不斷用新訊息更新它的同時保持任意狀態。你需要藉由兩步來使用它

- 定義狀態-狀態可以是任何的資料類型
- 定義狀態更新函數-怎樣利用更新前的狀態和從輸入串流裡面獲取的新值更新狀態

讓我們舉個例子說明。在例子中，你想保持一個文本資料串流中每個單字的運行次數，運行次數用一個state表示，它的類型是整數

```scala
def updateFunction(newValues: Seq[Int], runningCount: Option[Int]): Option[Int] = {
    val newCount = ...  // add the new values with the previous running count to get the new count
    Some(newCount)
}
```

這個函數被用到了DStream包含的單字上

```scala
import org.apache.spark._
import org.apache.spark.streaming._
import org.apache.spark.streaming.StreamingContext._
// Create a local StreamingContext with two working thread and batch interval of 1 second
val conf = new SparkConf().setMaster("local[2]").setAppName("NetworkWordCount")
val ssc = new StreamingContext(conf, Seconds(1))
// Create a DStream that will connect to hostname:port, like localhost:9999
val lines = ssc.socketTextStream("localhost", 9999)
// Split each line into words
val words = lines.flatMap(_.split(" "))
// Count each word in each batch
val pairs = words.map(word => (word, 1))
val runningCounts = pairs.updateStateByKey[Int](updateFunction _)
```

更新函數將會被每個單字調用，`newValues`擁有一系列的1（從 (词, 1)對而來），runningCount擁有之前的次數。要看完整的程式碼，見[例子](https://github.com/apache/spark/blob/master/examples/src/main/scala/org/apache/spark/examples/streaming/StatefulNetworkWordCount.scala)

## Transform操作

`transform`操作（以及它的變化形式如`transformWith`）允許在DStream運行任何RDD-to-RDD函數。它能夠被用來應用任何没在DStream API中提供的RDD操作（It can be used to apply any RDD operation that is not exposed in the DStream API）。
例如，連接資料串流中的每個批（batch）和另外一個資料集的功能並没有在DStream API中提供，然而你可以簡單的利用`transform`函數做到。如果你想藉由連接带有預先計算的垃圾邮件訊息的輸入資料串流
來清理即時資料，然後過了它們，你可以按如下函數來做：

```scala
val spamInfoRDD = ssc.sparkContext.newAPIHadoopRDD(...) // RDD containing spam information

val cleanedDStream = wordCounts.transform(rdd => {
  rdd.join(spamInfoRDD).filter(...) // join data stream with spam information to do data cleaning
  ...
})
```

事實上，你也可以在`transform`函數中用[機器学习](https://spark.apache.org/docs/latest/mllib-guide.html)和[圖計算](https://spark.apache.org/docs/latest/graphx-programming-guide.html)算法

## 窗口(window)操作

Spark Streaming也支持窗口計算，它允許你在一個滑動窗口資料上應用transformation操作。下圖阐明了這個滑動窗口。

![滑動窗口](../../img/streaming-dstream-window.png)

如上圖顯示，窗口在來源DStream上滑動，合並和操作落入窗内的來源RDDs，產生窗口化的DStream的RDDs。在這個具體的例子中，程式在三個時間單元的資料上進行窗口操作，並且每兩個時間單元滑動一次。
這說明，任何一個窗口操作都需要指定兩個參數：

- 窗口長度：窗口的持續時間
- 滑動的時間間隔：窗口操作執行的時間間隔

這兩個參數必須是來源DStream的批時間間隔的倍數。

下面舉例說明窗口操作。例如，你想擴充前面的[例子](../a-quick-example.md)用來計算過去30秒的词频，間隔時間是10秒。為了達到這個目的，我們必須在過去30秒的`pairs` DStream上應用`reduceByKey`
操作。用函數`reduceByKeyAndWindow`實作。

```scala
// Reduce last 30 seconds of data, every 10 seconds
val windowedWordCounts = pairs.reduceByKeyAndWindow((a:Int,b:Int) => (a + b), Seconds(30), Seconds(10))
```

一些常用的窗口操作如下所示，這些操作都需要用到上文提到的兩個參數：窗口長度和滑動的時間間隔

Transformation | Meaning
--- | ---
window(windowLength, slideInterval) | 基於來源DStream產生的窗口化的批次資料計算一個新的DStream
countByWindow(windowLength, slideInterval) | 返回流中元素的一個滑動窗口數
reduceByWindow(func, windowLength, slideInterval) | 返回一個單元素流。利用函數func聚集滑動時間間隔的流的元素創建這個單元素流。函數必須是相連接的以使計算能夠正確的並行計算。
reduceByKeyAndWindow(func, windowLength, slideInterval, [numTasks]) | 應用到一個(K,V)對組成的DStream上，返回一個由(K,V)對組成的新的DStream。每一個key的值均由给定的reduce函數聚集起來。注意：在預設情況下，這個操作利用了Spark預設的並發任務數去分组。你可以用`numTasks`參數設定不同的任務數
reduceByKeyAndWindow(func, invFunc, windowLength, slideInterval, [numTasks]) | A more efficient version of the above reduceByKeyAndWindow() where the reduce value of each window is calculated incrementally using the reduce values of the previous window. This is done by reducing the new data that enter the sliding window, and "inverse reducing" the old data that leave the window. An example would be that of "adding" and "subtracting" counts of keys as the window slides. However, it is applicable to only "invertible reduce functions", that is, those reduce functions which have a corresponding "inverse reduce" function (taken as parameter invFunc. Like in reduceByKeyAndWindow, the number of reduce tasks is configurable through an optional argument.
countByValueAndWindow(windowLength, slideInterval, [numTasks]) | 應用到一個(K,V)對組成的DStream上，返回一個由(K,V)對組成的新的DStream。每個key的值都是它們在滑動窗口中出現的频率。