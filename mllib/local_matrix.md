# 本地矩陣

本地矩陣儲存在單一台機器上，它有整數類型行列索引，以及浮點類型的值。MLlib支持密度矩陣(dense matrices)，其輸入值是被儲存在單一個以列為主的浮點陣列。舉例來說，以下這個矩陣
$$\begin{pmatrix}
    1.0 & 2.0\\
    3.0 & 4.0\\
    5.0 & 6.0\\
    \end{pmatrix}$$

被儲存在一個一維陣列[1.0, 3.0, 5.0, 2.0, 4.0, 6.0]以及大小為(3, 2)的矩陣中。


本地矩陣的基類為Matrix，並且提供一個實作類：DenseMatrix。我㫓建議使用Matrices中的工廠方法去創建本地矩陣。
```scala
import org.apache.spark.mllib.linalg.{Matrix, Matrices}

// Create a dense matrix ((1.0, 2.0), (3.0, 4.0), (5.0, 6.0))
val dm: Matrix = Matrices.dense(3, 2, Array(1.0, 3.0, 5.0, 2.0, 4.0, 6.0))
```
