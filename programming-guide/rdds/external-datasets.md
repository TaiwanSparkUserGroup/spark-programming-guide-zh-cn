## 外部資料集

Spark 支援任何一個 Hadoop 的文件系統建立分布式資料集，例如，HDFS，Cassandra，HBase，[Amazon S3](http://wiki.apache.org/hadoop/AmazonS3)等。此外， Spark 也支援文字文件(text files)，[SequenceFiles](http://hadoop.apache.org/docs/current/api/org/apache/hadoop/mapred/SequenceFileInputFormat.html) 和其他 Hadoop [InputFormat](http://hadoop.apache.org/docs/stable/api/org/apache/hadoop/mapred/InputFormat.html)。

文字文件 RDDs 可以由 SparkContext 的 `textFile` 函數建立。只需要在這函數中標示文件的 URI (機器上的本地路徑或是 `hdfs://`，`s3n://` 等)， Spark 會將文件讀取寫入成一個集合。以下是一個範例：

```scala
scala> val distFile = sc.textFile("data.txt")
distFile: RDD[String] = MappedRDD@1d4cee08
```

當上述步驟建立完成後，`distFiile` 就可以針對資料集做操作。例如，使用下面的方法使用 `map` 和 `reduce` 加總所有行的長度：`distFile.map(s => s.length).reduce((a, b) => a + b)`。

注意，Spark 讀取資料時：

- 如果使用本地文件系統路徑，文件必須能在 work 節點上同一個路徑中找到，所以你需要複製文件到所有的 workers，或者使用網路的方法分享文件。
- 所有 Spark 內關於連結文件的方法，除了 `textFile`，還有針對壓縮過的檔案，例如 `textFile("/my/文件目錄")`，`textFile("/my/文件目錄/*.txt")` 和`textFile("/my/文件目錄/*.gz")`。
- `textFile` 函數中，有第二個選擇參數來設定切片(_slices_) 的數目。預設情況下，Spark 為每一塊文件(HDFS 預設文件大小是 64M) 建立一個切片(_slice_)。但是你也可以修改成一個更大的值来設定一個更高的切片數目。值得注意，你不能設定一個小於文件數目的切片值。

除了文本文件，Spark 的 Scala API 支持其他幾種資料格式：

- `SparkContext.wholeTextFiles` 可以讀取一個包含多個檔案的文件目錄，並且返回每一個(filename, content)。和 `textFile` 的差異是：它記錄的是每一個檔案中的每一行。
- 關於 [SequenceFiles](http://hadoop.apache.org/docs/current/api/org/apache/hadoop/mapred/SequenceFileInputFormat.html)，你可以使用 SparkContext 的 `sequenceFile[K, V]` 方法產生，K 和 V 分別就是 key 和 values 。像 [IntWritable](http://hadoop.apache.org/docs/current/api/org/apache/hadoop/io/IntWritable.html) 與 [Text](http://hadoop.apache.org/docs/current/api/org/apache/hadoop/io/Text.html)，他們必須是Hadoop 的[Writable](http://hadoop.apache.org/docs/current/api/org/apache/hadoop/io/Writable.html) 的子類。另外對於幾種通用的 Writables，Spark 允許你指定原本類型來替代。例如： `sequenceFile[Int, String]` 會自動讀取 IntWritables 和 Text。

- 至於其他的 Hadoop InputFormats，可以使用 `SparkContext.hadoopRDD` 方法，它可以指定任意的 `JobConf`，輸入格式(InputFormat)，key 類型，values 類型。你可以跟設定 Hadoop job 一樣的方法設定輸入來源。你還可以在新的的 MapReduce 接口(org.apache.hadoop.mapreduce) 基礎上使用 `SparkContext.newAPIHadoopRDD` (譯者提醒：原生的接口是 `SparkContext.newHadoopRDD`)。
- `RDD.saveAsObjectFile` 和 `SparkContext.objectFile` 可以保存一個RDD，保存格式是一個簡單的 Java 物件序列化格式。這是一種效率不高的特有格式，如 Avro，它提供簡單的方法來保存任何一個 RDD。
