# RowMatrix

一个 RowMatrix 是一个面向行的分布式矩阵,其行索引是没有具体的含義。例如：一系列特征向量的一个集合。通過一个 RDD 来代表所有的行,每一行就是一个本地向量。既然 每一行由一个本地向量表示,所以其列数就被整型数据大小所限制,其實實作中列数是一個很小的数值。

RowMatrix可以由RDD[Vector] 實例被創建。接著我們可以計算列的摘要統計量。

```scala
import org.apache.spark.mllib.linalg.Vector
import org.apache.spark.mllib.linalg.distributed.RowMatrix


val rows: RDD[Vector] = ... // an RDD of local vectors
// Create a RowMatrix from an RDD[Vector].
val mat: RowMatrix = new RowMatrix(rows)

// Get its size.
val m = mat.numRows()
val n = mat.numCols()
```
