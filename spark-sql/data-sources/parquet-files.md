# [Parquet檔案](https://spark.apache.org/docs/latest/sql-programming-guide.html#parquet-files)

Parquet是一欄位(columnar)格式，可以被許多其它的資料處理系统支援。 Spark SQL 提供支援讀和寫 Parquet 檔案的功能，這些檔案可以自動地保留原始資料的模式。

## 讀取資料

```scala
// sqlContext from the previous example is used in this example.
// createSchemaRDD is used to implicitly convert an RDD to a SchemaRDD.
import sqlContext.createSchemaRDD

val people: RDD[Person] = ... // An RDD of case class objects, from the previous example.

// The RDD is implicitly converted to a SchemaRDD by createSchemaRDD, allowing it to be stored using Parquet.
people.saveAsParquetFile("people.parquet")

// Read in the parquet file created above.  Parquet files are self-describing so the schema is preserved.
// The result of loading a Parquet file is also a SchemaRDD.
val parquetFile = sqlContext.parquetFile("people.parquet")

//Parquet files can also be registered as tables and then used in SQL statements.
parquetFile.registerTempTable("parquetFile")
val teenagers = sqlContext.sql("SELECT name FROM parquetFile WHERE age >= 13 AND age <= 19")
teenagers.map(t => "Name: " + t(0)).collect().foreach(println)
```

## 配置

可以在 SQLContext 上使用 setConf 方法配置 Parquet ．或者在用 SQL 時使用 `SET key=value` 指令來配置 Parquet。

Property Name | Default | Meaning
--- | --- | ---
spark.sql.parquet.binaryAsString | false | 一些其它的Parquet-producing系统，特别是Impala和其它版本的Spark SQL，當寫出 Parquet 模式的时候，二進位資料和字串之間無法分區分。這個標記告诉Spark SQL 將二進位資料解釋為字串來提供這些系统的相容性。
spark.sql.parquet.cacheMetadata | true | 打開 parquet 中介資料的暫存，可以提高靜態數據的查詢速度
spark.sql.parquet.compression.codec | gzip | 設置寫 parquet 文件時的壓縮算法，可以接受的值包括：uncompressed, snappy, gzip, lzo
spark.sql.parquet.filterPushdown | false | 打開 Parquet 過濾器的 pushdown 優化。因為已知的 Paruet 錯誤，這個選項預設是關閉的。如果你的表不包含任何空的字串或者二進位的列，開啟這個選項仍是安全的
spark.sql.hive.convertMetastoreParquet | true | 當設置為 false 時，Spark SQL 將使用 Hive SerDe 代替内建的支援
