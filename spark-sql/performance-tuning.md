# [性能優化](https://spark.apache.org/docs/latest/sql-programming-guide.html#performance-tuning)

對於某些工作負載，可以在通過在記憶體中緩存數據或者打開一些實驗選項來提高性能。

## 在記憶體中緩存數據

Spark SQL可以通過調用 `sqlContext.cacheTable("tableName")` 方法來緩存使用欄位格式的表。然後， Spark 將會僅僅瀏覽需要的欄位並且自動壓縮數據以減少記憶體的使用以及垃圾回收的
壓力。你可以通過調用 `sqlContext.uncacheTable("tableName")` 方法在記憶體中刪除表。

注意，如果你調用 `schemaRDD.cache()` 而不是 `sqlContext.cacheTable(...)` ,表將不會用欄位格式來緩存。在這種情况下， `sqlContext.cacheTable(...)` 是强烈推薦的用法。

可以在 SQLContext 上使用 setConf 方法或者在用SQL時運行 `SET key=value` 命令來配置記憶體緩存。

Property Name | Default | Meaning
--- | --- | ---
spark.sql.inMemoryColumnarStorage.compressed | true | 當設置為 true 時， Spark SQL 將為基於數據統計信息的每列自動選擇一個壓縮算法。
spark.sql.inMemoryColumnarStorage.batchSize | 10000 | 控制每一批(batches)資料給欄位緩存的大小。更大的資料可以提高記憶體的利用率以及壓縮效率，但有OOMs的風險

## 其它的配置選項

以下的選項也可以用來條整查詢執行的性能。有可能這些選項會在以後的版本中放棄使用，這是因為更多的最佳化會自動執行。

Property Name | Default | Meaning
--- | --- | ---
spark.sql.autoBroadcastJoinThreshold | 10485760(10m) | 配置一個表的最大大小(byte)。當執行 join 操作時，這個表將會廣播到所有的 worker 節點。可以將值設置為 -1 來禁用廣播。注意，目前的統計數據只支持 Hive Metastore 表，命令 `ANALYZE TABLE <tableName> COMPUTE STATISTICS noscan` 已經在這個表中運行。
spark.sql.codegen | false | 當為 true 時，特定查詢中的表達式求值的代碼將會在運行時動態生成。對於一些用有複雜表達式的查詢，此選項可導致顯著速度提升。然而，對於簡單的查詢，這各選項會減慢查詢的執行
spark.sql.shuffle.partitions | 200 | 設定當操作 join 或者 aggregation 而需要 shuffle 資料時分區的数量
