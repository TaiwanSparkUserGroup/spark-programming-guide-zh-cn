# Checkpointing

一個串流應用程式必須全天候運行，所有必須能夠解决應用程式邏輯無關的故障（如系统錯誤，JVM崩溃等）。為了使這成為可能，Spark Streaming需要checkpoint足夠的訊息到容錯儲存系统中，
以使系统從故障中恢復。

- Metadata checkpointing：保存流計算的定義訊息到容錯儲存系统如HDFS中。這用來恢復應用程式中運行worker的節點的故障。元資料包括
    - Configuration ：創建Spark Streaming應用程式的配置訊息
    - DStream operations ：定義Streaming應用程式的操作集合
    - Incomplete batches：操作存在隊列中的未完成的批次
- Data checkpointing ：保存生成的RDD到可靠的儲存系统中，這在有狀態transformation（如结合跨多個批次的資料）中是必須的。在這樣一個transformation中，生成的RDDDependencies於之前
批的RDD，隨着時間的推移，這個依賴鏈的長度會持續增長。在恢復的過程中，為了避免這種無限增長。有狀態的transformation的中間RDD將會定時地儲存到可靠儲存系统中，以截斷這個依賴鏈。

元資料checkpoint主要是為了從driver故障中恢復資料。如果transformation操作被用到了，資料checkpoint即使在簡單的操作中都是必須的。

## 何時checkpoint

應用程式在下面兩種情況下必須開啟checkpoint

- 使用有狀態的transformation。如果在應用程式中用到了`updateStateByKey`或者`reduceByKeyAndWindow`，checkpoint目錄必需提供用以定期checkpoint RDD。
- 從運行應用程式的driver的故障中恢復過來。使用元資料checkpoint恢復處理訊息。

注意，没有前述的有狀態的transformation的簡單串流應用程式在運行時可以不開啟checkpoint。在這種情況下，從driver故障的恢復將是部分恢復（接收到了但是還没有處理的資料將會丢失）。
這通常是可以接受的，許多運行的Spark Streaming應用程式都是這種方式。

## 怎樣配置Checkpointing

在容錯、可靠的檔案系統（HDFS、s3等）中設定一個目錄用於保存checkpoint訊息。可以藉由`streamingContext.checkpoint(checkpointDirectory)`函數來做。這運行之前介紹的
有狀態transformation。另外，如果你想從driver故障中恢復，你應該以下面的方式重寫你的Streaming應用程式。

- 當應用程式是第一次啟動，新建一個StreamingContext，啟動所有Stream，然後調用`start()`函數
- 當應用程式因為故障重新啟動，它將會從checkpoint目錄checkpoint資料重新創建StreamingContext

```scala
// Function to create and setup a new StreamingContext
def functionToCreateContext(): StreamingContext = {
    val ssc = new StreamingContext(...)   // new context
    val lines = ssc.socketTextStream(...) // create DStreams
    ...
    ssc.checkpoint(checkpointDirectory)   // set checkpoint directory
    ssc
}

// Get StreamingContext from checkpoint data or create a new one
val context = StreamingContext.getOrCreate(checkpointDirectory, functionToCreateContext _)

// Do additional setup on context that needs to be done,
// irrespective of whether it is being started or restarted
context. ...

// Start the context
context.start()
context.awaitTermination()
```

如果`checkpointDirectory`存在，StreamingContext將會利用checkpoint資料重新創建。如果這個目錄不存在，將會調用`functionToCreateContext`函數創建一個新的StreamingContext，建立DStreams。
請看[RecoverableNetworkWordCount](https://github.com/apache/spark/tree/master/examples/src/main/scala/org/apache/spark/examples/streaming/RecoverableNetworkWordCount.scala)例子。

除了使用`getOrCreate`，開發者必須保證在故障發生時，driver處理自動重啟。只能藉由部署運行應用程式的基礎設施來達到該目的。在部署章節將有更進一步的討論。

注意，RDD的checkpointing有儲存成本。這會導致批次資料（包含的RDD被checkpoint）的處理時間增加。因此，需要小心的設定批次處理的時間間隔。在最小的批次大小(包含1秒的資料)情況下，checkpoint每批次資料會顯著的減少
操作的吞吐量。相反，checkpointing太少會導致lineage以及任務大小增大，這會產生有害的影響。因為有狀態的transformation需要RDD checkpoint。預設的間隔時間是批次間隔時間的倍數，最少10秒。它可以藉由`dstream.checkpoint`
來設定。典型的情況下，設定checkpoint間隔是DStream的滑動間隔的5-10大小是一個好的嘗試。


