# [Hive](https://spark.apache.org/docs/latest/sql-programming-guide.html#hive-tables)

Spark SQL也支援從 Apache Hive 中讀取和寫入資料。然而，Hive 有大量的相依套件，所以它不包含在 Spark 工具中。可以透過 `-Phive` 和 `-Phive-thriftserver` 參數建構 Spark，使其
支持Hive。注意這個重新建構的 jar 檔必須存在於所有的worker節點中，因為它們需要透過 Hive 的序列化和反序列化來存取儲存在 Hive 中的資料。

當和 Hive 一起工作時，開發者需要提供 HiveContext。 HiveContext 從 SQLContext 繼承而来，它增加了在 MetaStore 中發現表以及利用 HiveSql 寫查詢的功能。没有 Hive 部署的用戶也
可以創建 HiveContext。當没有通過 `hive-site.xml` 配置，上下文將會在當前目錄自動地創建 `metastore_db` 和 `warehouse`。

```scala
// sc is an existing SparkContext.
val sqlContext = new org.apache.spark.sql.hive.HiveContext(sc)

sqlContext.sql("CREATE TABLE IF NOT EXISTS src (key INT, value STRING)")
sqlContext.sql("LOAD DATA LOCAL INPATH 'examples/src/main/resources/kv1.txt' INTO TABLE src")

// Queries are expressed in HiveQL
sqlContext.sql("FROM src SELECT key, value").collect().foreach(println)
```
