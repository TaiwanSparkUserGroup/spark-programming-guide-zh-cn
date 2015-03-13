# [資料來源](https://spark.apache.org/docs/latest/sql-programming-guide.html#data-sources)

Spark SQL 支持通過 SchemaRDD 接口操作各種資料來源。一個 SchemaRDD 能夠作為一個一般的 RDD 被操作，也可以被註冊為一個臨時的表。註冊一個 SchemaRDD 為一個表就可以允許你在其數據上運行 SQL 查詢。這節描述了將資料讀取為 SchemaRDD 的多種方法。

* [RDDs](rdds.md)
* [parquet文件](parquet-files.md)
* [JSON資料集](jSON-datasets.md)
* [Hive表](hive-tables.md)
