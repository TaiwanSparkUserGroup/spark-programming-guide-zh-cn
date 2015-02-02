# 部署應用程式

## Requirements

運行一個Spark Streaming應用程式，有下面一些步骤

- 有管理器的集群-這是任何Spark應用程式都需要的需求，詳見[部署指南](../../deploying/README.md)
- 將應用程式包裝為jar檔-你必須編譯你的應用程式為jar檔。如果你用[spark-submit](../../deploying/submitting-applications.md)啟動應用程式，你不需要將Spark和Spark Streaming包裝進這個jar檔。
如果你的應用程式用到了高級來源（如kafka，flume），你需要將它們連接的外部artifact以及它們的Dependencies打包進需要部署的應用程式jar檔中。例如，一個應用程式用到了`TwitterUtils`，那麼就需要將`spark-streaming-twitter_2.10`
以及它的所有Dependencies打包到應用程式jar中。
- 為executors配置足夠的記憶體-因為接收的資料必須儲存在記憶體中，executors必須配置足夠的記憶體用來保存接收的資料。注意，如果你正在做10分鐘的窗口操作，系统的記憶體要至少能保存10分鐘的資料。所以，應用程式的記憶體需求Dependencies於使用
它的操作。
- 配置checkpointing-如果stream應用程式需要checkpointing，然後一個與Hadoop API兼容的容錯儲存目錄必須配置為檢查點的目錄，串流應用程式將checkpoint訊息寫入該目錄用於錯誤恢復。
- 配置應用程式driver的自動重啟-為了自動從driver故障中恢復，運行串流應用程式的部署設施必須能監控driver進程，如果失敗了能夠重啟它。不同的集群管理器，有不同的工具得到該功能
    - Spark Standalone：一個Spark應用程式driver可以提交到Spark獨立集群運行，也就是說driver運行在一個worker節點上。進一步來看，獨立的集群管理器能夠被指示用來監控driver，並且在driver失敗（或者是由於非零的退出程式碼如exit(1)，
    或者由於運行driver的節點的故障）的情況下重啟driver。
    - YARN：YARN為自動重啟應用程式提供了類似的機制。
    - Mesos： Mesos可以用[Marathon](https://github.com/mesosphere/marathon)提供該功能
- 配置write ahead logs-在Spark 1.2中，為了獲得极强的容錯保證，我們引入了一個新的實驗性的特性-預寫日誌（write ahead logs）。如果該特性開啟，從receiver獲取的所有資料會將預寫日誌寫入配置的checkpoint目錄。
這可以防止driver故障丢失資料，從而保證零資料丢失。這個功能可以藉由設定配置參數`spark.streaming.receiver.writeAheadLogs.enable`為true來開啟。然而，這些較强的語意可能以receiver的接收吞吐量為代价。這可以藉由
平行運行多個receiver增加吞吐量來解决。另外，當預寫日誌開啟時，Spark中的複製資料的功能推薦不用，因為該日誌已經儲存在了一個副本在儲存系统中。可以藉由設定輸入DStream的儲存級别為`StorageLevel.MEMORY_AND_DISK_SER`獲得該功能。


## 升級應用程式程式碼

如果運行的Spark Streaming應用程式需要升級，有兩種可能的函數

- 啟動升級的應用程式，使其與未升級的應用程式平行運行。一旦新的程式（與就程式接收相同的資料）已經準備就緒，舊的應用程式就可以關閉。這種函數支援將資料發送到兩個不同的目的地（新程式一個，舊程式一個）
- 首先，平滑的關閉（`StreamingContext.stop(...)`或`JavaStreamingContext.stop(...)`）现有的應用程式。在關閉之前，要保證已經接收的資料完全處理完。然後，就可以啟動升級的應用程式，升級
的應用程式會接着舊應用程式的點開始處理。這種函數僅支援具有來源端暫存功能的輸入來源（如flume，kafka），這是因為當舊的應用程式已經關閉，升級的應用程式還没有啟動的時候，資料需要被暫存。

