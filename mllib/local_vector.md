# 本地向量

一個本地向量有intereage, 0-based indices及double類型，貯存在單一機器上。MLlib 支持兩種類型的本地向量: dense及sparse。

dense向量：透過輸入double陣列回傳。

sparse向量：透過兩個平行陣列：indices及values回傳

舉個例子，向量(1.0, 0.0, 3.0)可以被表示成:

dense格式－[1.0, 0.0, 3.0]

sparse格式－(3, [0, 2], [1.0, 3.0])，3表示此vector的大小。


本地向量的基類是Vector，而且提供DenseVector及SparseVector兩個實作。以下介紹使用工廠方法在Vectors中去創建本地向量。
```scala
import org.apache.spark.mllib.linalg.{Vector, Vectors}

// Create a dense vector (1.0, 0.0, 3.0).
val dv: Vector = Vectors.dense(1.0, 0.0, 3.0)
// Create a sparse vector (1.0, 0.0, 3.0) by specifying its indices and values corresponding to nonzero entries.
val sv1: Vector = Vectors.sparse(3, Array(0, 2), Array(1.0, 3.0))
// Create a sparse vector (1.0, 0.0, 3.0) by specifying its nonzero entries.
val sv2: Vector = Vectors.sparse(3, Seq((0, 1.0), (2, 3.0)))
```
Note: Scala會預設匯入scala.collection.immutable.Vector，所以你必須自行匯入org.apache.spark.mllib.linalg.Vector，才能使用MLlib的Vector

--------

DenseVector 與 SparseVector源碼


```scala
@SQLUserDefinedType(udt = classOf[VectorUDT])
class DenseVector(val values: Array[Double]) extends Vector {

  override def size: Int = values.length

  override def toString: String = values.mkString("[", ",", "]")

  override def toArray: Array[Double] = values

  private[mllib] override def toBreeze: BV[Double] = new BDV[Double](values)

  override def apply(i: Int) = values(i)

  override def copy: DenseVector = {
    new DenseVector(values.clone())
  }

  private[spark] override def foreachActive(f: (Int, Double) => Unit) = {
    var i = 0
    val localValuesSize = values.size
    val localValues = values

    while (i < localValuesSize) {
      f(i, localValues(i))
      i += 1
    }
  }
}

```

```scala

/**
 * A dense vector represented by a value array.
 */
@SQLUserDefinedType(udt = classOf[VectorUDT])
class DenseVector(val values: Array[Double]) extends Vector {

  override def size: Int = values.length

  override def toString: String = values.mkString("[", ",", "]")

  override def toArray: Array[Double] = values

  private[mllib] override def toBreeze: BV[Double] = new BDV[Double](values)

  override def apply(i: Int) = values(i)

  override def copy: DenseVector = {
    new DenseVector(values.clone())
  }

  private[spark] override def foreachActive(f: (Int, Double) => Unit) = {
    var i = 0
    val localValuesSize = values.size
    val localValues = values

    while (i < localValuesSize) {
      f(i, localValues(i))
      i += 1
    }
  }
}

```
