# 彈性分布式資料集 (RDDs)

Spark 核心概念是 _Resilient Distributed Dataset (RDD)_ ，你可以將它視為一個可以併型操作、有容錯機制的資料集和。目前有 2 種方式可以建立 RDDs：第一種是在你執行的驅動程式中平行化一個已經存在集合；另外一個方式是引用外部儲存系統的資料集，例如共享文件系統，HDFS，HBase或其他 Hadoop 資料格式的資料來源。

* [平行集合](parallelized-collections.md)
* [外部資料集](external-datasets.md)
* [RDD 操作](rdd-operations.md)
  * [傳送函數到 Spark](passing-functions-to-spark.md)
  * [使用鍵值對](working-with-key-value-pairs.md)
  * [Transformations](transformations.md)
  * [Actions](actions.md)
* [RDD 持久化](rdd_persistence.md)
