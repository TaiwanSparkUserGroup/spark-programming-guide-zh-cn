# RDD 操作

RDDs 支持兩種類型的操作：_轉換(transformations)_ 從已經存在的資料集裡面建立一個新的資料集：_動作(actions)_ 在資料集上做運算之後返回一個值到驅動程式。例如，`map` 是一個轉換操作，它會將每一個資料集內的元素傳遞給函數並且返回新的 RDD。另一方面，`reduce` 是一個動作，它會使用相同的函數來對 RDD 內所有的元素做整合，並且將最後的結果送回驅動程式(不過也有一個函數 `reduceByKey` 功能是返回一個分布式資料集)。

在 Spark 中，所有的转换(transformations)都是惰性(lazy)的，它们不会马上计算它们的结果。相反的，它们仅仅记录转换操作是应用到哪些基础数据集(例如一个文件)上的。转换仅仅在这个时候计算：当动作(action) 需要一个结果返回给驱动程序的时候。这个设计能够让 Spark 运行得更加高效。例如，我们可以实现：通过 `map` 创建一个新数据集在 `reduce` 中使用，并且仅仅返回 `reduce` 的结果给 driver，而不是整个大的映射过的数据集。

預設情況，每一個轉換過後的 RDD 會在每次執行動作(action) 的時候重新計算。當然，你也可以使用 `persist` (或 `cache`) 方式做持久化(`persist`) 一個 RDD 到緩存裡。在這情況之下， Spark 會在集群上保存相關的訊息，在你下次查詢時，縮短運行時間，同時也支援 RDD 持久化到硬碟，或是在其他節點上做複製動作。

## 基礎

為了理解 RDD 基本知識，可以先了解下面的簡單程式：

```scala
val lines = sc.textFile("data.txt")
val lineLengths = lines.map(s => s.length)
val totalLength = lineLengths.reduce((a, b) => a + b)
```

第一行是定義外部文件的 RDD。這個資料集並沒有收到緩存或做其他的操作：`lines` 單單只是一個指向文件位置的動作。第二行是定義 `lineLengths`，它是 `map` 轉換(transformation) 的結果。同樣，因為 Spark 的 lazy 模式，`lineLengths` _不會_被立刻計算。最後，當我們執行 `reduce`，因為這是一個動作(action)， Spark 會把計算分成多個任務(task)，並且讓他們運作在多個機器上。每台機器都會執行自己的 map 或是本地 reduce ，最後將結果送回給驅動程式。

如果想再次使用 `lineLengths`，我們可以這樣做：

```scala
lineLengths.persist()
```

在 `reduce` 之前，它會把 `lineLengths` 在第一次運算結束後將內容保存到緩存中，以利後續再次使用。
