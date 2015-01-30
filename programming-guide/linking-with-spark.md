# 引入 Spark

Spark 1.2.0 使用 Scala 2.10 撰寫應用程式，因此你的Scala 版本必須相容(例如：2.10.X)。

撰寫 Spark 應用程式時，你需要添加 Spark 的 Maven 依賴，Spark 可以透過 Maven 中心庫來取得：

```
groupId = org.apache.spark
artifactId = spark-core_2.10
version = 1.2.0
```

如果你希望連結 HDFS 集群，需要根據你的 HDFS 版本設定 `hadoop-client`的相依性。你可以透過[第三方發行頁面](https://spark.apache.org/docs/latest/hadoop-third-party-distributions.html)找到相對應的版本

```
groupId = org.apache.hadoop
artifactId = hadoop-client
version = <your-hdfs-version>
```

最後，你需要匯入一些 Spark 的類別(class) 和隱式轉換 (implicit conversions) 到你的程式，增加下面幾行即可：

```scala
import org.apache.spark.SparkContext
import org.apache.spark.SparkContext._
import org.apache.spark.SparkConf
```
