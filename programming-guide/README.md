# 概論

<<<<<<< HEAD
在結構中，每隻 Spark 應用程式都由一隻*驅動程式(driver program)*構成，驅動程序在集群上運行用户的 `main` 函數来執行各式各樣的*併行操作(parallel operations)*。Spark 的主要抽象是提供一個*彈性分布式資料庫(RDD)*，RDD 是指能横跨集群所有節點進行併行計算的分區元素集合。RDDs 從 Hadoop 的文件系统中的一個文件中產生而來(或其他 Hadoop 支持的文件系统)，或者從一個已有的 Scala 集合轉換得到。用戶可以將 Spark RDD *持久化(persist)*到記憶體中，讓它在併行計算中有效率的被重複使用。而且，RDDs 能在節點失敗中自動恢復。
=======
每隻 Spark 應用程式都由一隻*驅動程式(driver programe)*構成，驅動程序在集群上運行用户的 `main` 函數来執行各式各樣的*併行操作(parallel operations)*。Spark 的主要抽象是提供一個*彈性分布式資料集(RDD)*，RDD 是指能横跨集群所有節點進行併行計算的分區元素集合。RDDs 從 Hadoop 的文件系统中的一個文件中產生而來(或其他 Hadoop 支持的文件系统)，或者從一個已有的 Scala 集合轉換得到。用戶可以將 Spark RDD *持久化(persist)*到記憶體中，讓它在併行計算中有效率的被重複使用。而且，RDDs 能在節點失敗中自動恢復。
>>>>>>> 096283ef3499c9ec7158e4a20e1f55688521d802

Spark 的第二個抽象是*共享變數(shared variables)*，共享變數被運行在併行運算中。默認情況下，當 Spark 運行一個併行函數時，這個併行函數會作為一個任務集在不同的節點上運行，它會把函數裡使用到的每個變數都複製移動到每個任務中。有時，一個變數需被共享到交叉任務中或驅動程式和任務之間。Spark 支持 2 種類型的共享變數：*廣播變數(broadcast variables)*，使用在所有節點的記憶體中快取一個值；累加器(accumulators)，只能執行“增加(added)”操作，例如：計數器(counters)和加總(sums)。

這個指南會在 Spark 支持的所有語言中展示它的每一個特性。簡單的操作 Spark 互動式 shell - `bin/spark-shell` 操作 Scala shell，或 `bin/pyspark` 啟動一個 Python shell。

* [引入 Spark](linking-with-spark.md)
* [初始化 Spark](initializing-spark.md)
* [Spark RDDs](rdds/README.md)
* [共享變數](shared-variables.md)
* [從這裡開始](from-here.md)
