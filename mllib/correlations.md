# 相關性(Correlations)
在统计分析中,計算兩系列數據之間的相關性很常见。在 MLlib 中,我们提供了用於計算多系列數據之间兩兩關系的靈活性。目前支持的相關性算法是 Perarson 和 Spearsman 相關。

[Statistics](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.mllib.stat.Statistics$)提供了計算系列間相關性的方法。根據輸入類型，两个RDD[Double] 或 ,輸出相應的一个Double或相關矩陣。
```scala
import org.apache.spark.SparkContext
import org.apache.spark.mllib.linalg._
import org.apache.spark.mllib.stat.Statistics

val sc: SparkContext = ...

val seriesX: RDD[Double] = ... // a series
val seriesY: RDD[Double] = ... // must have the same number of partitions and cardinality as seriesX

// compute the correlation using Pearson's method. Enter "spearman" for Spearman's method. If a
// method is not specified, Pearson's method will be used by default.
val correlation: Double = Statistics.corr(seriesX, seriesY, "pearson")

val data: RDD[Vector] = ... // note that each Vector is a row and not a column

// calculate the correlation matrix using Pearson's method. Use "spearman" for Spearman's method.
// If a method is not specified, Pearson's method will be used by default.
val correlMatrix: Matrix = Statistics.corr(data, "pearson")
```
