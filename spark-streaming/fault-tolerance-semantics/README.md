# 容錯語意

這一節，我們將討論在節點錯誤事件時Spark Streaming的行為。為了理解這些，讓我們先記住一些Spark RDD的基本容錯語意。

- 一個RDD是不可變的、確定可重複計算的、分散式資料集。每個RDD記住一個確定性操作的lineage(lineage)，這個lineage用在容錯的輸入資料集上來創建該RDD。
- 如果任何一個RDD的分區因為節點故障而丢失，這個分區可以藉由操作lineage從來源容錯的資料集中重新計算得到。
- 假定所有的RDD transformations是確定的，那麼最终轉換的資料是一樣的，不論Spark機器中發生何種錯誤。

Spark運行在像HDFS或S3等容錯系统的資料上。因此，任何從容錯資料而來的RDD都是容錯的。然而，這不是在Spark Streaming的情況下，因為Spark Streaming的資料大部分情況下是從
網路中得到的。為了獲得生成的RDD相同的容錯属性，接收的資料需要重複保存在worker node的多個Spark executor上（預設的複製因子是2），這導致了當出現錯誤事件時，有兩類資料需要被恢復

- Data received and replicated ：在單個worker節點的故障中，這個資料會幸存下來，因為有另外一個節點保存有這個資料的副本。
- Data received but buffered for replication：因為没有重複保存，所以為了恢復資料，唯一的办法是從來源中重新讀取資料。

有兩種錯誤我們需要關心

- worker節點故障：任何運行executor的worker節點都有可能出故障，那樣在這個節點中的所有記憶體資料都會丢失。如果有任何receiver運行在錯誤節點，它們的暫存資料將會丢失
- Driver節點故障：如果運行Spark Streaming應用程式的Driver節點出現故障，很明顯SparkContext將會丢失，所有執行在其上的executors也會丢失。

## 作為輸入來源的文件語意（Semantics with files as input source）

如果所有的輸入資料都存在於一個容錯的檔案系統如HDFS，Spark Streaming總可以從任何錯誤中恢復並且執行所有資料。這给出了一個恰好一次(exactly-once)語意，即無論發生什麼故障，
所有的資料都將會恰好處理一次。

## 基於receiver的輸入來源語意

對於基於receiver的輸入來源，容錯的語意既Dependencies於故障的情形也Dependencies於receiver的類型。正如之前討論的，有兩種類型的receiver

- Reliable Receiver：這些receivers只有在確保資料複製之後才會告知可靠來源。如果這樣一個receiver失敗了，緩衝區（非複製）資料不會被來源所承認。如果receiver重啟，來源會重發數
据，因此不會丢失資料。
- Unreliable Receiver：當worker或者driver節點故障，這種receiver會丢失資料

選擇哪種類型的receiverDependencies於這些語意。如果一個worker節點出現故障，Reliable Receiver不會丢失資料，Unreliable Receiver會丢失接收了但是没有複製的資料。如果driver節點
出現故障，除了以上情況下的資料丢失，所有過去接收並複製到記憶體中的資料都會丢失，這會影響有狀態transformation的结果。

為了避免丢失過去接收的資料，Spark 1.2引入了一個實驗性的特徵`write ahead logs`，它保存接收的資料到容錯儲存系统中。有了`write ahead logs`和Reliable Receiver，我們可以
做到零資料丢失以及exactly-once語意。

下面的表格總结了錯誤語意：

Deployment Scenario | Worker Failure | Driver Failure
--- | --- | ---
Spark 1.1 或者更早, 没有write ahead log的Spark 1.2 | 在Unreliable Receiver情況下緩衝區資料丢失；在Reliable Receiver和文件的情況下，零資料丢失 | 在Unreliable Receiver情況下緩衝區資料丢失；在所有receiver情況下，過去的資料丢失；在文件的情況下，零資料丢失
带有write ahead log的Spark 1.2 | 在Reliable Receiver和文件的情況下，零資料丢失 | 在Reliable Receiver和文件的情況下，零資料丢失

## 輸出操作的語意

根據其確定操作的lineage，所有資料都被建模成了RDD，所有的重新計算都會產生同樣的结果。所有的DStream transformation都有exactly-once語意。那就是說，即使某個worker節點出現
故障，最终的轉換结果都是一樣。然而，輸出操作（如`foreachRDD`）具有`at-least once`語意，那就是說，在有worker事件故障的情況下，變换後的資料可能被寫入到一個外部實體不止一次。
利用`saveAs***Files`將資料保存到HDFS中的情況下，以上寫多次是能夠被接受的（因為文件會被相同的資料覆盖）。