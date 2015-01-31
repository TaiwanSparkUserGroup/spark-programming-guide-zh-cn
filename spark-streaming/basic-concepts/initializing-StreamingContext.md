# 初始化StreamingContext

為了初始化Spark Streaming程式，一個StreamingContext對象必需被創建，它是Spark Streaming所有串流操作的主要入口。一個[StreamingContext](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.streaming.StreamingContext)
對象可以用[SparkConf](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.SparkConf)對象創建。

```scala
import org.apache.spark._
import org.apache.spark.streaming._
val conf = new SparkConf().setAppName(appName).setMaster(master)
val ssc = new StreamingContext(conf, Seconds(1))
```

`appName`表示你的應用程式顯示在集群UI上的名字，`master`是一個[Spark、Mesos、YARN](https://spark.apache.org/docs/latest/submitting-applications.html#master-urls)集群URL
或者一個特殊字串“local[*]”，它表示程式用本地模式運行。當程式運行在集群中時，你並不希望在程式中硬編碼`master`，而是希望用`spark-submit`啟動應用程式，並從`spark-submit`中得到
`master`的值。對於本地測試或者單元測試，你可以傳遞“local”字串在同一個進程内運行Spark Streaming。需要注意的是，它在内部創建了一個SparkContext對象，你可以藉由` ssc.sparkContext`
訪問這個SparkContext對象。

批次時間片需要根據你的程式的潛在需求以及集群的可用資源來設定，你可以在[性能調教](../performance-tuning/README.md)那一節獲取詳細的訊息。

可以利用已經存在的`SparkContext`對象創建`StreamingContext`對象。

```scala
import org.apache.spark.streaming._
val sc = ...                // existing SparkContext
val ssc = new StreamingContext(sc, Seconds(1))
```

當一個StreamingContext（context）定義之後，你必須按照以下幾步進行操作

- 定義輸入來源；
- 準備好串流計算指令；
- 利用`streamingContext.start()`函數接收和處理資料；
- 處理過程將一直持續，直到`streamingContext.stop()`函數被調用。

幾點需要注意的地方：

- 一旦一個context已經啟動，就不能有新的串流操作建立或者是添加到context中。
- 一旦一個context已經停止，它就不能再重新啟動
- 在JVM中，同一時間只能有一個StreamingContext处於活躍狀態
- 在StreamingContext上調用`stop()`函數，也會關閉SparkContext對象。如果只想僅關閉StreamingContext對象，設定`stop()`的可選參數為false
- 一個SparkContext對象可以重複利用去創建多個StreamingContext對象，前提條件是前面的StreamingContext在後面StreamingContext創建之前關閉（不關閉SparkContext）。