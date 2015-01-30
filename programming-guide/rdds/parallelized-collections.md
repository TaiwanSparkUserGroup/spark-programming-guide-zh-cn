## 併行集合

建立併行集合 (_Parallelized collections_) 的方法是利用一個已存在的序列(Scala `Seq`)上使用 SparkContext 的 `parallelize`。序列中的元素會被複製到一個併行操作的分布式資料集合裡。例如：如何建立一個包含數字1 到5 的併行集合：

```scala
val data = Array(1, 2, 3, 4, 5)
val distData = sc.parallelize(data)
```

當併行集合建立完成，這樣的分布式資料集(`distData`) 就可以被操作。例如，可以使用 `distData.reduce((a, b) => a + b)` 將裡面的元素數字加總。後續會再對這部份進行操作上的介紹。

值得注意，在併行集合裡有一個重要的概念，就是切片數(_slices_)，即一個資料集被切的份數。Spark 會在集群上指派每一個切片執行任務。你也可以在集群上替每個 CPU 設定 2-4 個切片(slices)。一般情況下， Spark 會常識基於你的集群狀態自動設定切片的數量。當然，你也可以利用 `parallelize` 參數的第二個位置做設定，例如`sc.parallelize(data, 10)`。

