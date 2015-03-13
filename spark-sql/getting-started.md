# [開始](https://spark.apache.org/docs/latest/sql-programming-guide.html#getting-started)

Spark中所有相關功能的入口點是[SQLContext](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.SQLContext)物件或者它的子物件，創建一個SQLContext僅僅需要一個SparkContext。

```scala
val sc: SparkContext // An existing SparkContext.
val sqlContext = new org.apache.spark.sql.SQLContext(sc)

// createSchemaRDD is used to implicitly convert an RDD to a SchemaRDD.
import sqlContext.createSchemaRDD
```
除了一個基本的 SQLContext，你也能夠創建一個 HiveContext，它支持基本 SQLContext 所支持功能的一個超集( superset)。它額外的功能包括用更完整的 HiveQL 分析器寫查詢去訪問 HiveUDFs 的能力、
從 Hive Table 讀取數據的能力。用 HiveContext 你不需要開啟一個已經存在的 Hive， SQLContext 可用的數據源對 HiveContext 也可用。HiveContext 分開打包是為了避免在 Spark 構建時包含了所有的 Hive 依賴。如果對你的應用程序來說，這些依賴不存在问题， Spark 1.2推薦使用 HiveContext。以後的穩定版本將專注於為 SQLContext 提供與 HiveContext 等價的功能。

用來解析查詢語句的特定SQL變種語言可以通過 `spark.sql.dialect` 選項來選擇。這個參數可以通過兩種方式改變，一種方式是通過 `setConf` 方法設定，另一種方式是在SQL命令中通過 `SET key=value` 
來設定。對於 SQLContext ，唯一可用的方言是 “sql” ，它是 Spark SQL 提供的一个简单的SQL解析器。在 HiveContext 中，虽然也支持 "sql" ，但默认的方言是 “hiveql”。這是因為 HiveQL 解析器更
完整。在很多用例中推薦使用 “hiveql”。
