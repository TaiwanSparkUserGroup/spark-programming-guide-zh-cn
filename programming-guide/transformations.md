# Transformations

下面的表格列了 Sparkk 支援且常用 transformations。細節請參考 RDD API 手冊([Scala](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.rdd.RDD), [Java](https://spark.apache.org/docs/latest/api/java/index.html?org/apache/spark/api/java/JavaRDD.html), [Python](https://spark.apache.org/docs/latest/api/python/pyspark.rdd.RDD-class.html)) 和 PairRDDFunctions 文档([Scala](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.rdd.PairRDDFunctions), [Java](https://spark.apache.org/docs/latest/api/java/index.html?org/apache/spark/api/java/JavaPairRDD.html))。 

Transformation | Meaning
--- | ---
map(_func_) | 回傳一個新的分布式資料集，將資料來源的每一個元素傳遞給函數 _func_ 映射組成。
filter(_func_) | 回傳一個新的資料集，從資料來源中篩選元素通過函數 _func_ 回傳 true。
flatMap(_func_) | 類似 map，但是每個輸入元素能被映射成多個輸出選項(所以 _func_ 必須回傳一個 Seq，而非單一 item)。
mapPartitions(_func_) | 類似 map，但分別運作在 RDD 的每個分區上，火以 _func_ 的類型必須為 `Iterator<T> => Iterator<U>` 當執行在類型是 T 的 RDD 上。
mapPartitionsWithIndex(_func_) | 類似 mapPartitions，但 _func_ 需要提供一個 integer 值描述索引(index)，所以 _func_ 的類型必須是 (Int, Iterator<T>) => Iterator<U> 當執行在類型為 T 的 RDD 上。
sample(withReplacement, fraction, seed) | 對資料做抽樣。
union(otherDataset) | Return a new dataset that contains the union of the elements in the source dataset and the argument.
intersection(otherDataset) | Return a new RDD that contains the intersection of elements in the source dataset and the argument.
distinct([numTasks])) | Return a new dataset that contains the distinct elements of the source dataset.
groupByKey([numTasks]) | When called on a dataset of (K, V) pairs, returns a dataset of (K, Iterable<V>) pairs. Note: If you are grouping in order to perform an aggregation (such as a sum or average) over each key, using reduceByKey or combineByKey will yield much better performance. Note: By default, the level of parallelism in the output depends on the number of partitions of the parent RDD. You can pass an optional numTasks argument to set a different number of tasks.
reduceByKey(func, [numTasks]) | When called on a dataset of (K, V) pairs, returns a dataset of (K, V) pairs where the values for each key are aggregated using the given reduce function func, which must be of type (V,V) => V. Like in groupByKey, the number of reduce tasks is configurable through an optional second argument.
aggregateByKey(zeroValue)(seqOp, combOp, [numTasks]) | When called on a dataset of (K, V) pairs, returns a dataset of (K, U) pairs where the values for each key are aggregated using the given combine functions and a neutral "zero" value. Allows an aggregated value type that is different than the input value type, while avoiding unnecessary allocations. Like in groupByKey, the number of reduce tasks is configurable through an optional second argument.
sortByKey([ascending], [numTasks]) | When called on a dataset of (K, V) pairs where K implements Ordered, returns a dataset of (K, V) pairs sorted by keys in ascending or descending order, as specified in the boolean ascending argument.
join(otherDataset, [numTasks]) | When called on datasets of type (K, V) and (K, W), returns a dataset of (K, (V, W)) pairs with all pairs of elements for each key. Outer joins are also supported through leftOuterJoin and rightOuterJoin.
cogroup(otherDataset, [numTasks]) | When called on datasets of type (K, V) and (K, W), returns a dataset of (K, Iterable<V>, Iterable<W>) tuples. This operation is also called groupWith.
cartesian(otherDataset) | When called on datasets of types T and U, returns a dataset of (T, U) pairs (all pairs of elements).
pipe(command, [envVars]) | Pipe each partition of the RDD through a shell command, e.g. a Perl or bash script. RDD elements are written to the process's stdin and lines output to its stdout are returned as an RDD of strings.
coalesce(numPartitions) | Decrease the number of partitions in the RDD to numPartitions. Useful for running operations more efficiently after filtering down a large dataset.
repartition(numPartitions) | Reshuffle the data in the RDD randomly to create either more or fewer partitions and balance it across them. This always shuffles all data over the network.
