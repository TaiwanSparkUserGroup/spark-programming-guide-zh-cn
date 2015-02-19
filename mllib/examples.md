# 示例
下面代碼片段演示了如何加載數據集，運用算法對象的靜態方法執行訓練算法，以及運用模型預測來計算訓練誤差。
```scala
import org.apache.spark.SparkContext
import org.apache.spark.mllib.classification.SVMWithSGD
import org.apache.spark.mllib.evaluation.BinaryClassificationMetrics
import org.apache.spark.mllib.regression.LabeledPoint
import org.apache.spark.mllib.linalg.Vectors
import org.apache.spark.mllib.util.MLUtils

// Load training data in LIBSVM format.
val data = MLUtils.loadLibSVMFile(sc, "data/mllib/sample_libsvm_data.txt")

// Split data into training (60%) and test (40%).
val splits = data.randomSplit(Array(0.6, 0.4), seed = 11L)
val training = splits(0).cache()
val test = splits(1)

// Run training algorithm to build the model
val numIterations = 100
val model = SVMWithSGD.train(training, numIterations)

// Clear the default threshold.
model.clearThreshold()

// Compute raw scores on the test set.
val scoreAndLabels = test.map { point =>
  val score = model.predict(point.features)
  (score, point.label)
}

// Get evaluation metrics.
val metrics = new BinaryClassificationMetrics(scoreAndLabels)
val auROC = metrics.areaUnderROC()

println("Area under ROC = " + auROC)
```
預設配置下，SVMWithSGD.train()將正則化參數設置為1.0來進行L2正則化。如果我們想配置算法參數，我們可以直接生一個SVMWithSGD對象，然後調用setter方法。所有其他的MLlib算法都支持這 種自定義方法。舉例來說，下面代碼生了一個用於SVM的L1正則化變量，其正則化參數為0.1，且迭代次數為200。
```scala
import org.apache.spark.mllib.optimization.L1Updater

val svmAlg = new SVMWithSGD()
svmAlg.optimizer.
  setNumIterations(200).
  setRegParam(0.1).
  setUpdater(new L1Updater)
val modelL1 = svmAlg.run(training)
```
[LogisticRegressionWithSGD](https://spark.apache.org/docs/latest/api/scala/index.html#package)的使用方法與SVMWithSGD相似。
