# 初始化 Spark

使用 Spark 的第一步是創建[SparkContext](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.SparkContext) 物件，讓 Spark 知道如何找到集群。而建立 `SparkContext` 之前，還需建立 [SparkConf](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.SparkConf) 物件，而這個物件則包含你的應用程式資訊。

```scala
val conf = new SparkConf().setAppName(appName).setMaster(master)
new SparkContext(conf)
```

`appName`參數是你自定的程式名稱，會顯示在 cluster UI 上。`master` 是[Spark, Mesos 或 YARN 集群的 URL](https://spark.apache.org/docs/latest/submitting-applications.html#master-urls)，或是運行本地模式(standalone) 時可使用 “local”。實際上，當你的程式在集群上運作時，你不需要把 `master` 放在程式中，因此可以[用 spark-submit 啟動你的應用程式](https://spark.apache.org/docs/latest/submitting-applications.html)。當然，你也可以在 Spark 程式中使用 “local” 做本地測試或是單元測試。


## 使用 Shell

在 Spark shell 裡，有一個內建的 SparkContext，名為 `sc`。你可以用 `--master` 設定 SparkContext 欲連結的集群，用 `--jars`來指定需要加到 classpath 中的 JAR 包，倘若有多個 JAR，可使用**逗號**分隔符號來連結他們。例如：想在一個擁有 4 個CPU 的環境上執行 `bin/spark-shell`：

```
$ ./bin/spark-shell --master local[4]
```

或是想在 classpath 中增加 `code.jar`，你可以這樣寫：

```
$ ./bin/spark-shell --master local[4] --jars code.jar
```

此外，你可以執行 `spark-shell --help` 得到完整的參數列表。目前，使用 `spark-submit` 會比 [spark-shell ](https://spark.apache.org/docs/latest/submitting-applications.html)普遍。
