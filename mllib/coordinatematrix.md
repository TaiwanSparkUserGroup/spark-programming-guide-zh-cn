# CoordinateMatrix
CoordinateMatrix是一个分布式矩陣,其實體集合是一個 RDD。每一个實體是一个(i:Long, j:Long, value:Double)座標 ￼ ,其中i代表行索引，j代表列索引,value代 表實体的值。只有當矩陣的行和列都很巨大,並且矩陣很稀疏时才使用 CoordinateMatrix。

一个CoordinateMatrix可從一個RDD[MatrixEntry]實例創建,这里的 MatrixEntry是的(Long, Long, Double)的封裝尸類。通過調用toIndexedRowMatrix可以將一个CoordinateMatrix 轉變为一個IndexedRowMatrix(但其行是稀疏的)。目前暂不支持其他計算操作。

```scala
import org.apache.spark.mllib.linalg.distributed.{CoordinateMatrix, MatrixEntry}

val entries: RDD[MatrixEntry] = ... // an RDD of matrix entries
// Create a CoordinateMatrix from an RDD[MatrixEntry].
val mat: CoordinateMatrix = new CoordinateMatrix(entries)

// Get its size.
val m = mat.numRows()
val n = mat.numCols()

// Convert it to an IndexRowMatrix whose rows are sparse vectors.
val indexedRowMatrix = mat.toIndexedRowMatrix()
```
