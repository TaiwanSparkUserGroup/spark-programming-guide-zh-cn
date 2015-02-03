# 離散化串流（DStreams）

離散化串流（Discretized Streams）或者DStreams是Spark Streaming提供的基本的抽象類別，它代表一個連續的資料串流。它要麼是從來源中獲取的輸入串流，要麼是輸入串流藉由轉換操作生成的處理後的資料串流。在内部，DStreams由一系列連續的
RDD組成。DStreams中的每個RDD都包含特定時間間隔内的資料，如下圖所示：

![DStreams](../../img/streaming-dstream.png)

任何對DStreams的操作都轉換成了對DStreams隱含的RDD的操作。在前面的[例子](../a-quick-example.md)中，`flatMap`操作應用於`lines`這個DStreams的每個RDD，生成`words`這個DStreams的
RDD。過程如下圖所示：

![DStreams](../../img/streaming-dstream-ops.png)

藉由Spark引擎計算這些隱含RDD的轉換操作。DStreams操作隱藏了大部分的細節，並且為了更便捷，為開發者提供了更高層的API。下面幾節將具體討論這些操作的細節。