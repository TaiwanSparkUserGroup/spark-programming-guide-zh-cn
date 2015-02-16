# 標記點(Labeled point)

標記點是一個本地向量，無論密集(dense)或稀疏(sparse)均會與一一個標記/響應相關。在MLlib中，標記點被使用在監督式學習算法中。我們使用一個double去儲存一個標記(label)，那麼我們將可以在迴歸(regression)及分類(classification)中使用標記點。在二元分類中，標記應該是0或1；而在多類分類中，標記應為從0開始的的索引:0, 1, 2, ...


一個標記點用 [LabeledPoint](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.mllib.regression.LabeledPoint)來表示（Scala中它屬於一個case class）

```scala
import org.apache.spark.mllib.linalg.Vectors
import org.apache.spark.mllib.regression.LabeledPoint

// Create a labeled point with a positive label and a dense feature vector.
val pos = LabeledPoint(1.0, Vectors.dense(1.0, 0.0, 3.0))

// Create a labeled point with a negative label and a sparse feature vector.
val neg = LabeledPoint(0.0, Vectors.sparse(3, Array(0, 2), Array(1.0, 3.0)))
```

* Sparse data

