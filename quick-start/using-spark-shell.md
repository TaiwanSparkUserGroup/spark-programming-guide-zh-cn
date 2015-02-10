# [使用 Spark Shell](https://spark.apache.org/docs/latest/quick-start.html#interactive-analysis-with-the-spark-shell)

## 基礎

Spark 的 shell 作為一個强大的交互式數據分析工具，提供了一個簡單的方式來學習 API。它可以使用 Scala(在 Java 虛擬機上運行現有的 Java 庫的一个很好方式) 或 Python。在 Spark 目錄裡使用下面的方式開始運行：

```scala
./bin/spark-shell
```

Spark 最主要的抽象是叫Resilient Distributed Dataset(RDD) 的彈性分布式資料集。RDDs 可以使用 Hadoop InputFormats(例如 HDFS 文件)創建，也可以從其他的 RDDs 轉換。讓我們在 Spark 源代碼目錄從 README 文本文件中創建一個新的 RDD。

```scala
scala> val textFile = sc.textFile("README.md")
textFile: spark.RDD[String] = spark.MappedRDD@2ee9b6e3
```

RDD 的 [actions](https://spark.apache.org/docs/latest/programming-guide.html#actions) 從 RDD 中返回值，[transformations](https://spark.apache.org/docs/latest/programming-guide.html#transformations) 可以轉換成一個新 RDD 並返回它的引用。讓我們開始使用幾個操作：

```scala
scala> textFile.count() // RDD 的數據行數
res0: Long = 126

scala> textFile.first() // RDD 的第一行數據
res1: String = # Apache Spark
```

現在讓我們使用一個 transformation，我們將使用 [filter](https://spark.apache.org/docs/latest/programming-guide.html#transformations) 在這個文件裡返回一個包含子數據集的新 RDD。

```scala
scala> val linesWithSpark = textFile.filter(line => line.contains("Spark"))
linesWithSpark: spark.RDD[String] = spark.FilteredRDD@7dd4af09
```

我們可以把 actions 和 transformations 連接在一起：

```scala
scala> textFile.filter(line => line.contains("Spark")).count() // 有多少行包括 "Spark"?
res3: Long = 15
```

## 更多 RDD 操作

RDD actions 和 transformations 能被用在更多的複雜計算中。比方說，我們想要找到一行中最多的單詞數量：

```scala
scala> textFile.map(line => line.split(" ").size).reduce((a, b) => if (a > b) a else b)
res4: Long = 15
```

首先將行映射(map)成一個整數產生一個新 RDD。 在這個新的 RDD 上調用 `reduce` 找到行中最大的個數。 `map` 和 `reduce` 的參數是 Scala 的函數串(closures)，並且可以使用任何語言特性或者 Scala/Java 函式庫。例如，我们可以很方便地調用其他的函數。 我們使用 `Math.max()` 函數讓代碼更容易理解：

```scala
scala> import java.lang.Math
import java.lang.Math

scala> textFile.map(line => line.split(" ").size).reduce((a, b) => Math.max(a, b))
res5: Int = 15
```

Hadoop 流行的一個通用的數據流(data flow)模式是 MapReduce。Spark 能很容易地實現 MapReduce：

```scala
scala> val wordCounts = textFile.flatMap(line => line.split(" ")).map(word => (word, 1)).reduceByKey((a, b) => a + b)
wordCounts: spark.RDD[(String, Int)] = spark.ShuffledAggregatedRDD@71f027b8
```

這裡，我們結合 [flatMap](), [map]() 和 [reduceByKey]() 來計算文件裡每個單詞出現的數量，它的結果是包含一組(String, Int) 鍵值對的 RDD。我们可以使用 [collect] 操作在我們的 shell 中收集單詞的數量：

```scala
scala> wordCounts.collect()
res6: Array[(String, Int)] = Array((means,1), (under,2), (this,3), (Because,1), (Python,2), (agree,1), (cluster.,1), ...)
```

## 暫存(Caching)

Spark 支持把數據集拉到集群內的暫存記憶體中。當要重複取用資料時非常有用．例如當我们在一個小的熱(hot)數據集中查詢，或者運行一個像網頁搜索排序這樣的迭代演算法。作為一個簡單的例子，讓我们把 `linesWithSpark` 數據集標記在暫存中：

```scala
scala> linesWithSpark.cache()
res7: spark.RDD[String] = spark.FilteredRDD@17e51082

scala> linesWithSpark.count()
res8: Long = 15

scala> linesWithSpark.count()
res9: Long = 15
```

暫存 100 行的文本文件來研究 Spark 這看起来很傻。真正讓人感興趣的部分是我们可以在非常大型的數據集中使用同樣的函數，甚至在 10 個或者 100 個節點中交叉計算。你同樣可以使用 `bin/spark-shell` 連接到一个 cluster 來繼亂掉[编程指南](https://spark.apache.org/docs/latest/programming-guide.html#initializing-spark)中的方法進行交互操作。