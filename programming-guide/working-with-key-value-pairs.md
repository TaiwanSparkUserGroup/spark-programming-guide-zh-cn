# 使用鍵值對

即使很多 Spark 的操作視在任意類型物件的 RDDs 上，仍有少數幾個特殊操作僅在鍵值(key-value) 對 RDDs 上使用。最常見的是分布式 "shuffle" 操作，例如根據一個 key 對一組資料進行分組跟整合。

在 Scala 中，這些操作包含[二元组(Tuple2)](http://www.scala-lang.org/api/2.10.4/index.html#scala.Tuple2)(在語言的內建元祖中，簡單的寫 (a, b) 即可建立) 的 RDD 上自動變成可用元素，只要在你的程式中匯入 `org.apache.spark.SparkContext._` 來啟動 Spark 的隱式轉換(implicit conversions)。在 PairRDDFunctions 的類別鍵值對操作中可以使用，如果你匯入隱式轉換，它會自動包成元組 (tuple) RDD。

舉例，下面的程式在鍵值對上使用 `reduceByKey` 來統計一個文件裡面每一行內容出現的次數：

```scala
val lines = sc.textFile("data.txt")
val pairs = lines.map(s => (s, 1))
val counts = pairs.reduceByKey((a, b) => a + b)
```

也可以使用 `counts.sortByKey()`，例如，將鍵值對按照字母做排序，最後用 `counts.collect()` 輸出成結果陣列送回驅動程式。

注意：使用一個自行定義物件作為 key 在使用鍵值對操作的時候，需要確保定義的`equals()` 方法和 `hashCode()` 方法是匹配的。細節請看 [Object.hashCode() 文件](http://docs.oracle.com/javase/7/docs/api/java/lang/Object.html#hashCode()) 內的描述。
