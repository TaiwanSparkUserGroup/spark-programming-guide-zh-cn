# Spark 編程指南繁體中文版
=============================
如果你是個讀者，這邊有更容易閱讀的[Gitbook版本](http://taiwansparkusergroup.gitbooks.io/spark-programming-guide-zh-tw/)

## 貢獻方式
請有意願加入的同好參考(https://github.com/TaiwanSparkUserGroup/spark-programming-guide-zh-tw/blob/master/CONTRIBUTING.rst)

## 大綱

* [簡介](README.md)
* [快速上手](quick-start/README.md)
  * [Spark Shell](quick-start/using-spark-shell.md)
  * [獨立應用程序](quick-start/self-contained-applications.md)
  * [開始翻滾吧!](quick-start/where-to-go-from-here.md)
* [編程指南](programming-guide/README.md)
  * [引入 Spark](programming-guide/linking-with-spark.md)
  * [初始化 Spark](programming-guide/initializing-spark.md)
  * [Spark RDDs](programming-guide/rdds/README.md)
    * [平行集合](programming-guide/rdds/parallelized-collections.md)
    * [外部數據集](programming-guide/rdds/external-datasets.md)
    * [RDD 操作](programming-guide/rdds/rdd-operations.md)
      * [傳遞函數到 Spark](programming-guide/rdds/passing-functions-to-spark.md)
      * [使用鍵值對](programming-guide/rdds/working-with-key-value-pairs.md)
      * [轉換](programming-guide/rdds/transformations.md)
      * [行動](programming-guide/rdds/actions.md)
    * [RDD持續化](programming-guide/rdds/rdd-persistences.md)
  * [共享變數](programming-guide/shared-variables.md)
  * [從這裡開始](programming-guide/from-here.md)
* [Spark Streaming](spark-streaming/README.md)
  * [一個快速的例子](spark-streaming/a-quick-example.md)
  * [基本概念](spark-streaming/basic-concepts/README.md)
    * [連接](spark-streaming/basic-concepts/linking.md)
    * [初始化StreamingContext](spark-streaming/basic-concepts/initializing-StreamingContext.md)
    * [離散化串流](spark-streaming/basic-concepts/discretized-streams.md)
    * [输入DStreams](spark-streaming/basic-concepts/input-DStreams.md)
    * [DStream中的轉換](spark-streaming/basic-concepts/transformations-on-DStreams.md)
    * [DStream的輸出操作](spark-streaming/basic-concepts/output-operations-on-DStreams.md)
    * [暫存或持續化](spark-streaming/basic-concepts/caching-persistence.md)
    * [Checkpointing](spark-streaming/basic-concepts/checkpointing.md)
    * [部署應用程序](spark-streaming/basic-concepts/deploying-applications.md)
    * [監控應用程序](spark-streaming/basic-concepts/monitoring-applications.md)
  * [性能優化](spark-streaming/performance-tuning/README.md)
    * [減少處理時間](spark-streaming/performance-tuning/reducing-processing-time.md)
    * [設置正確的的批次大小](spark-streaming/performance-tuning/setting-right-batch-size.md)
    * [記憶體優化](spark-streaming/performance-tuning/memory-tuning.md)
  * [容錯語意](spark-streaming/fault-tolerance-semantics/README.md)
* [Spark SQL](spark-sql/README.md)
  * [開始](spark-sql/getting-started.md)
  * [資料來源](spark-sql/data-sources/README.md)
    * [RDDs](spark-sql/data-sources/rdds.md)
    * [parquet文件](spark-sql/data-sources/parquet-files.md)
    * [JSON數據集](spark-sql/data-sources/jSON-datasets.md)
    * [Hive表](spark-sql/data-sources/hive-tables.md)
  * [性能優化](spark-sql/performance-tuning.md)
  * [其它SQL接口](spark-sql/other-sql-interfaces.md)
  * [編寫語言集成(Language-Integrated)的相關查詢](spark-sql/writing-language-integrated-relational-queries.md)
  * [Spark SQL術劇類型](spark-sql/spark-sql-dataType-reference.md)
* [MLlib](mllib/README.md)
  * [數據類型](mllib/data_type.md)
    * [本地向量](mllib/local_vector.md)

* [GraphX編程指南](graphx-programming-guide/README.md)
  * [開始](graphx-programming-guide/getting-started.md)
  * [屬性圖](graphx-programming-guide/property-graph.md)
  * [圖操作](graphx-programming-guide/graph-operators.md)
  * [Pregel API](graphx-programming-guide/pregel-api.md)
  * [圖建立者](graphx-programming-guide/graph-builders.md)
  * [頂點和邊RDDs](graphx-programming-guide/vertex-and-edge-rdds.md)
  * [圖算法](graphx-programming-guide/graph-algorithms.md)
  * [例子](graphx-programming-guide/examples.md)
* [部署](deploying/submitting-applications.md)
  * [提交應用程序](deploying/submitting-applications.md)
  * [獨立運行Spark](deploying/spark-standalone-mode.md)
  * [在yarn上運行Spark](deploying/running-spark-on-yarn.md)
* [更多文檔](more/spark-configuration.md)
  * [Spark配置](more/spark-configuration.md)
    * [RDD持續化](programming-guide/rdds/rdd_persistence.md)

## Copyright

本文翻譯自

[Spark 官方手冊](https://spark.apache.org/docs/latest/)

* Reference:

[Spark 编程指南简体中文版](https://github.com/aiyanbo/spark-programming-guide-zh-cn)

## License

本文使用的許可請查看[這裡](LICENSE)
