# 一個快速的例子

在我們進入如何編寫Spark Streaming程式的細節之前，讓我們快速地瀏覽一個簡單的例子。在這個例子中，程式從監聽TCP Socket的資料伺服器取得文字資料，然後計算文字中包含的單字數。做法如下：

首先，我們導入Spark Streaming的相關類別以及一些從StreamingContext獲得的隱式轉換到我們的環境中，為我們所需的其他類別（如DStream）提供有用的函數。[StreamingContext](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.streaming.StreamingContext)
是Spark所有串流操作的主要入口。然後，我們創建了一個具有兩個執行執行緒以及1秒批次間隔時間(即以秒為單位分割資料串流)的本地StreamingContext。

```scala
import org.apache.spark._
import org.apache.spark.streaming._
import org.apache.spark.streaming.StreamingContext._
// Create a local StreamingContext with two working thread and batch interval of 1 second
val conf = new SparkConf().setMaster("local[2]").setAppName("NetworkWordCount")
val ssc = new StreamingContext(conf, Seconds(1))
```
利用這個StreamingContext，我們能夠創建一個DStream，它表示從TCP來源（主機位址localhost，port為9999）取得的資料串流。

```scala
// Create a DStream that will connect to hostname:port, like localhost:9999
val lines = ssc.socketTextStream("localhost", 9999)
```
這個`lines`變數是一個DStream，表示即將從資料伺服器獲得的資料串流。這個DStream的每條紀錄都代表一行文字。下一步，我們需要將DStream中的每行文字都切分為單字。

```scala
// Split each line into words
val words = lines.flatMap(_.split(" "))
```
`flatMap`是一個一對多的DStream操作，它通過把原DStream的每條紀錄都生成多條新紀錄來創建一個新的DStream。在這個例子中，每行文字都被切分成了多個單字，我們把切分
的單字串流用`words`這個DStream表示。下一步，我們需要計算單字的個數。

```scala
import org.apache.spark.streaming.StreamingContext._
// Count each word in each batch
val pairs = words.map(word => (word, 1))
val wordCounts = pairs.reduceByKey(_ + _)
// Print the first ten elements of each RDD generated in this DStream to the console
wordCounts.print()
```
`words`這個DStream被mapper(一對一轉換操作)成了一個新的DStream，它由（word，1）對組成。然後，我們就可以用這個新的DStream計算每批次資料的單字頻率。最後，我們用`wordCounts.print()`
印出每秒計算的單字頻率。

需要注意的是，當以上這些程式碼被執行時，Spark Streaming僅僅準備好了它要執行的計算，實際上並没有真正開始執行。在這些轉換操作準備好之後，要真正執行計算，需要調用以下的函數

```scala
ssc.start()             // Start the computation
ssc.awaitTermination()  // Wait for the computation to terminate
```
完整的例子可以在[NetworkWordCount](https://github.com/apache/spark/blob/master/examples/src/main/scala/org/apache/spark/examples/streaming/NetworkWordCount.scala)中找到。

如果你已經下載和建構了Spark環境，你就能夠用以下的函數執行這個例子。首先，你需要運行Netcat作為資料伺服器

```shell
$ nc -lk 9999
```
然後，在不同的terminal，你能夠用如下方式執行例子
```shell
$ ./bin/run-example streaming.NetworkWordCount localhost 9999
```
