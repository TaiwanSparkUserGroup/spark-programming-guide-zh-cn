# IndexedRowMatrix

IndexedRowMatrix與RowMatrix是相似的，但其行索引具有特定定含義，本質上是一個含有索引訊息的行數據集合(an RDD of indexed rows)。
每一行由long類型索引和一個本地向量組成。



[IndexedRowMatrix](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.mllib.linalg.distributed.IndexedRowMatrix)可以由一個RDD[IndexedRow]實例被創建，其中[IndexedRow](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.mllib.linalg.distributed.IndexedRow)是一個被包裝過的(Long, Vcetor)。IndexedRowMatrix可以透過刪除行索引被轉換成RowMatrix。
```scala
import org.apache.spark.mllib.linalg.distributed.{IndexedRow, IndexedRowMatrix, RowMatrix}
import util.Random

val rows: RDD[IndexedRow] = ... // an RDD of indexed rows
// Create an IndexedRowMatrix from an RDD[IndexedRow].
val mat: IndexedRowMatrix = new IndexedRowMatrix(rows)

// Get its size.
val m = mat.numRows()
val n = mat.numCols()

// Drop its row indices.
val rowMat: RowMatrix = mat.toRowMatrix()
```
