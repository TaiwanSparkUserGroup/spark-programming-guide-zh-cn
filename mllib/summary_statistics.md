# 概述統計量(Summary Statistics)
对 RDD[Vector]格式數據的概述統計量,我们提供 Statistics 中的 colStats 方法来實现。

colStats()方法返回一個MultivariateStatisticalSummary實例，其中包括面向列的最大值，最小值，平均，變異數，非零值個數以及總數量。
```scala
import org.apache.spark.mllib.linalg.Vector
import org.apache.spark.mllib.stat.{MultivariateStatisticalSummary, Statistics}

val observations: RDD[Vector] = sc.textFile("data/mllib/sample_lda_data.txt").map(s=>Vectors.dense(s.split(" ").map(_.toDouble)))
... // an RDD of Vectors

// Compute column summary statistics.
val summary: MultivariateStatisticalSummary = Statistics.colStats(observations)
println(summary.mean) // a dense vector containing the mean value for each column
println(summary.variance) // column-wise variance
println(summary.numNonzeros) // number of nonzeros in each column
```
