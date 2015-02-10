# Spark SQL

Spark SQL允許Spark執行用SQL, HiveQL或者Scala表示的關係查詢。這個模組的核心是一個新類型的RDD-[SchemaRDD](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.SchemaRDD)。
SchemaRDDs由[行](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.package@Row:org.apache.spark.sql.catalyst.expressions.Row.type)物件組成，行物件用有一個模式（scheme）
來描述行中每一列的數據類型。SchemaRDD與關聯式資料庫中的表(table)很相似。可以通過存在的RDD、一個[Parquet](http://parquet.io/)文件、一個JSON數據庫或者對儲存在[Apache Hive](http://hive.apache.org/)中的數據執行HiveSQL查詢中創建。

本章的所有例子都利用了Spark分布式系统中的樣本資料，可以在`spark-shell`中運行它們。

* [開始](getting-started.md)
* [數據源](data-sources/README.md)
  * [RDDs](data-sources/rdds.md)
  * [parquet文件](data-sources/parquet-files.md)
  * [JSON數據集](data-sources/jSON-datasets.md)
  * [Hive表](data-sources/hive-tables.md)
* [性能優化](performance-tuning.md)
* [其它SQL接口](other-sql-interfaces.md)
* [編寫語言整合(Language-Integrated)的相關查詢](writing-language-integrated-relational-queries.md)
* [Spark SQL數據類型](spark-sql-dataType-reference.md)
