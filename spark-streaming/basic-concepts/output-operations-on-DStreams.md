# DStreams上的輸出操作

輸出操作允許DStream的操作推到如資料函式庫、檔案系統等外部系统中。因為輸出操作實际上是允許外部系统消費轉換後的資料，它們触發的實际操作是DStream轉換。目前，定義了下面幾種輸出操作：

Output Operation | Meaning
--- | ---
print() | 在DStream的每個批次資料中印出前10条元素，這個操作在開發和調试中都非常有用。在Python API中調用`pprint()`。
saveAsObjectFiles(prefix, [suffix]) | 保存DStream的内容為一個序列化的文件`SequenceFile`。每一個批間隔的文件的文件名基於`prefix`和`suffix`生成。"prefix-TIME_IN_MS[.suffix]"，在Python API中不可用。
saveAsTextFiles(prefix, [suffix]) | 保存DStream的内容為一個文本文件。每一個批間隔的文件的文件名基於`prefix`和`suffix`生成。"prefix-TIME_IN_MS[.suffix]"
saveAsHadoopFiles(prefix, [suffix]) | 保存DStream的内容為一個hadoop文件。每一個批間隔的文件的文件名基於`prefix`和`suffix`生成。"prefix-TIME_IN_MS[.suffix]"，在Python API中不可用。
foreachRDD(func) | 在從流中生成的每個RDD上應用函數`func`的最通用的輸出操作。這個函數應該推送每個RDD的資料到外部系统，例如保存RDD到文件或者藉由網路寫到資料函式庫中。需要注意的是，`func`函數在驅動程式中執行，並且通常都有RDD action在裡面推動RDD流的計算。

## 利用foreachRDD的設計模式

dstream.foreachRDD是一個强大的原语，發送資料到外部系统中。然而，明白怎樣正確地、有效地用這個原语是非常重要的。下面幾點介紹了如何避免一般錯誤。
- 经常寫資料到外部系统需要建一個連接對象（例如到远程服務器的TCP連接），用它發送資料到远程系统。為了達到這個目的，開發人员可能不经意的在Spark驱動中創建一個連接對象，但是在Spark worker中
嘗試調用這個連接對象保存紀錄到RDD中，如下：

```scala
  dstream.foreachRDD(rdd => {
      val connection = createNewConnection()  // executed at the driver
      rdd.foreach(record => {
          connection.send(record) // executed at the worker
      })
  })
```

這是不正確的，因為這需要先序列化連接對象，然後將它從driver發送到worker中。這樣的連接對象在機器之間不能传送。它可能表现為序列化錯誤（連接對象不可序列化）或者初始化錯誤（連接對象應該
在worker中初始化）等等。正確的解决办法是在worker中創建連接對象。

- 然而，這會造成另外一個常見的錯誤-為每一個紀錄創建了一個連接對象。例如：

```
  dstream.foreachRDD(rdd => {
      rdd.foreach(record => {
          val connection = createNewConnection()
          connection.send(record)
          connection.close()
      })
  })
```

通常，創建一個連接對象有資源和時間的開支。因此，為每個紀錄創建和销毁連接對象會導致非常高的開支，明顯的減少系统的整體吞吐量。一個更好的解决办法是利用`rdd.foreachPartition`函數。
為RDD的partition創建一個連接對象，用這個兩件對象發送partition中的所有紀錄。

```
 dstream.foreachRDD(rdd => {
      rdd.foreachPartition(partitionOfRecords => {
          val connection = createNewConnection()
          partitionOfRecords.foreach(record => connection.send(record))
          connection.close()
      })
  })
```
這就將連接對象的創建開销分摊到了partition的所有紀錄上了。

- 最後，可以藉由在多個RDD或者批次資料間重用連接對象做更進一步的優化。開發者可以保有一個静态的連接對象池，重複使用池中的對象將多批次的RDD推送到外部系统，以進一步節省開支。

```
  dstream.foreachRDD(rdd => {
      rdd.foreachPartition(partitionOfRecords => {
          // ConnectionPool is a static, lazily initialized pool of connections
          val connection = ConnectionPool.getConnection()
          partitionOfRecords.foreach(record => connection.send(record))
          ConnectionPool.returnConnection(connection)  // return to the pool for future reuse
      })
  })
```

需要注意的是，池中的連接對象應該根據需要延遲創建，並且在空闲一段時間後自動超時。這樣就獲取了最有效的方式發生資料到外部系统。

其它需要注意的地方：

- 輸出操作藉由懒執行的方式操作DStreams，正如RDD action藉由懒執行的方式操作RDD。具體地看，RDD actions和DStreams輸出操作接收資料的處理。因此，如果你的應用程式没有任何輸出操作或者
用於輸出操作`dstream.foreachRDD()`，但是没有任何RDD action操作在`dstream.foreachRDD()`裡面，那麼什麼也不會執行。系统僅僅會接收輸入，然後丢棄它們。
- 預設情況下，DStreams輸出操作是分時執行的，它們按照應用程式的定義順序按序執行。


