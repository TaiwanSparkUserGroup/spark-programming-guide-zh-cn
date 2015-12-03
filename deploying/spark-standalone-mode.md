# Spark獨立佈署模式

## 安裝Spark獨立模式集群

安装Spark獨立模式，你只需要將Spark的編譯版本簡單的放到集群的每個節點。你可以獲得每個穩定版本的預編譯版本，也可以自己編譯。

## 手動啟動集群

你能夠通過下面的方式啟動獨立的master服務器。

```shell
./sbin/start-master.sh
```

一旦啟動，master將會為自己打印出`spark://HOST:PORT` URL，你能夠用它連接到workers或者作為"master"参數傳遞给`SparkContext`。你也可以在master web UI上發現這個URL，
master web UI默認的地址是`http://localhost:8080`。

相同的，你也可以啟動一個或者多個workers或者将它们連接到master。

```shell
./bin/spark-class org.apache.spark.deploy.worker.Worker spark://IP:PORT
```

一旦你啟動了一個worker，查看master web UI。你可以看到新的節點列表以及節點的CPU數以及内存。

下面的配置参數可以傳遞给master和worker。

Argument | Meaning
--- | ---
-h HOST, --host HOST | 監聽的主機名
-i HOST, --ip HOST | 同上，已經被淘汰
-p PORT, --port PORT | 監聽的服務的端口（master默認是7077，worker隨機）
--webui-port PORT | web UI的端口(master默認是8080，worker默認是8081)
-c CORES, --cores CORES | Spark應用程序可以使用的CPU核數（默認是所有可用）；這個選項僅在worker上可用
-m MEM, --memory MEM | Spark應用程序可以使用的内存數（默認情况是你的機器内存數減去1g）；這個選項僅在worker上可用
-d DIR, --work-dir DIR | 用於暫存空間和工作輸出日誌的目錄（默認是SPARK_HOME/work）；這個選項僅在worker上可用
--properties-file FILE | 自定義的Spark配置文件的加载目录（默認是conf/spark-defaults.conf）

## 集群啟動腳本

為了用啟動腳本啟動Spark獨立集群，你應該在你的Spark目錄下建立一個名為`conf/slaves`的文件，這個文件必須包含所有你要啟動的Spark worker所在機器的主機名，一行一個。如果
`conf/slaves`不存在，啟動脚本默認为單個機器（localhost），這台機器對於測試是有用的。注意，master機器通過ssh訪問所有的worker。在默認情况下，SSH是並行運行，需要設置無密碼（採用私有密鑰）的訪問。
如果你沒有設置為無密碼訪問，你可以設置環境變量`SPARK_SSH_FOREGROUND`，為每個worker提供密碼。

一旦你設置了這個文件，你就可以通過下面的shell腳本啟動或者停止你的集群。

- sbin/start-master.sh：在機器上啟動一個master實例
- sbin/start-slaves.sh：在每台機器上啟動一個slave實例
- sbin/start-all.sh：同時啟動一個master实例和所有slave實例
- sbin/stop-master.sh：停止master實例
- sbin/stop-slaves.sh：停止所有slave實例
- sbin/stop-all.sh：停止master实例和所有slave實例

注意，這些脚本必須在你的Spark master運行的機器上執行，而不是在你的本地機器上面。

你可以在`conf/spark-env.sh`中設置環境變量進一步配置集群。利用`conf/spark-env.sh.template`創建這个文件，然後將它複製到所有的worker機器上使設置有效。下面的設置可以起作用：

Environment Variable | Meaning
--- | ---
SPARK_MASTER_IP | 绑定master到一個指定的ip地址
SPARK_MASTER_PORT | 在不同的端口上啟動master（默認是7077）
SPARK_MASTER_WEBUI_PORT | master web UI的端口（默認是8080）
SPARK_MASTER_OPTS | 應用到master的配置屬性，格式是 "-Dx=y"（默认是none），查看下面的表格的選項以组成一個可能的列表
SPARK_LOCAL_DIRS | Spark中暂存空間的目錄。包括map的輸出文件和存儲在磁盤上的RDDs(including map output files and RDDs that get stored on disk)。这必须在一个快速的、你的系统的本地磁盘上。它可以是一个逗号分隔的列表，代表不同磁盘的多个目录
SPARK_WORKER_CORES | Spark应用程序可以用到的核心数（默認是所有可用）
SPARK_WORKER_MEMORY | Spark应用程序用到的内存总数（默認是内存總數減去1G）。注意，每個應用程序個體的内存通過`spark.executor.memory`設置
SPARK_WORKER_PORT | 在指定的端口上啟動Spark worker(默認是随機)
SPARK_WORKER_WEBUI_PORT | worker UI的端口（默认是8081）
SPARK_WORKER_INSTANCES | 每台機器運行的worker實例数，默認是1。如果你有一台非常大的機器並且希望運行多個worker，你可以設置這個數大於1。如果你設置了這個環境變量，確保你也設置了`SPARK_WORKER_CORES`環境變量用於限制每個worker的核數或者每個worker嘗試使用所有的核。
SPARK_WORKER_DIR | Spark worker運行目錄，該目錄包括日誌和暫存空間（默認是SPARK_HOME/work）
SPARK_WORKER_OPTS | 應用到worker的配置屬性，格式是 "-Dx=y"（默認是none），查看下面表格的選項以组成一個可能的列表
SPARK_DAEMON_MEMORY | 分配给Spark master和worker守護進程的内存（默認是512m）
SPARK_DAEMON_JAVA_OPTS | Spark master和worker守護進程的JVM選項，格式是"-Dx=y"（默認為none）
SPARK_PUBLIC_DNS | Spark master和worker公共的DNS名（默認是none）

注意，啟動腳本還不支持windows。為了在windows上啟動Spark集群，需要手動啟動master和workers。

`SPARK_MASTER_OPTS`支持以下的系统屬性：

Property Name | Default | Meaning
--- | --- | ---
spark.deploy.retainedApplications | 200 | 展示完成的應用程序的最大数目。老的應用程序會被删除以滿足該限制
spark.deploy.retainedDrivers | 200 | 展示完成的drivers的最大數目。老的應用程序會被删除以滿足該限制
spark.deploy.spreadOut | true | 這個選項控制獨立的集群管理器是應該跨節點傳遞應用程序還是應努力將程序整合到儘可能少的節點上。在HDFS中，傳遞程序是數據本地化更好的選擇，但是，對於計算密集型的負載，整合會更有效率。
spark.deploy.defaultCores | (infinite) | 在Spark獨立模式下，給應用程序的默認核数（如果没有設置`spark.cores.max`）。如果没有設置，應用程序總數獲得所有可用的核，除非設置了`spark.cores.max`。在共享集群上設置較低的核數，可用防止用戶默認抓住整個集群。
spark.worker.timeout | 60 | 獨立佈署的master認為worker失敗（没有收到心跳信息）的間隔時間。

`SPARK_WORKER_OPTS`支持的系统屬性：

Property Name | Default | Meaning
--- | --- | ---
spark.worker.cleanup.enabled | false | 周期性的清空worker/應用程序目錄。注意，這僅僅影響獨立佈署模式。不管應用程序是否還在執行，用於程序目錄都會被清空
spark.worker.cleanup.interval | 1800 (30分) | 在本地機器上，worker清空老的應用程序工作目錄的時間間隔
spark.worker.cleanup.appDataTtl | 7 * 24 * 3600 (7天) | 每個worker中應用程序工作目錄的保留時間。這個時間依賴於你可用磁盤空間的大小。應用程序日誌和jar包上傳到每個應用程序的工作目錄。隨著時間的推移，工作目錄會很快的填滿磁盤空間，特別是如果你運行的作業很頻繁。

## 連接一個應用程序到集群中

為了在Spark集群中運行一個應用程序，簡單地傳遞`spark://IP:PORT` URL到[SparkContext](http://spark.apache.org/docs/latest/programming-guide.html#initializing-spark)

為了在集群上運行一個交互式的Spark shell，運行一下命令：

```shell
./bin/spark-shell --master spark://IP:PORT
```
你也可以傳遞一個選項`--total-executor-cores <numCores>`去控制spark-shell的核數。

## 啟動Spark應用程序

[spark-submit脚本](submitting-applications.md)支持最直接的提交一個Spark應用程序到集群。對於獨立部署的集群，Spark目前支持兩種佈署模式。在`client`模式中，driver啟動进程與
客户端提交應用程序所在的进程是同一个进程。然而，在`cluster`模式中，driver在集群的某個worker进程中啟動，只有客户端進程完成了提交任务，它不會等到应用程序完成就會退出。

如果你的應用程序通過Spark submit啟動，你的應用程序jar包將會自動分發到所有的worker節點。對於你的應用程序依賴的其它jar包，你應該用`--jars`符號指定（如` --jars jar1,jar2`）。

另外，`cluster`模式支持自動的重啟你的應用程序（如果程序一非零的退出碼退出）。為了用這個特徵，當啟動應用程序時，你可以傳遞`--supervise`符号到`spark-submit`。如果你想殺死反覆失敗的應用，
你可以通過如下的方式：

```shell
./bin/spark-class org.apache.spark.deploy.Client kill <master url> <driver ID>
```

你可以在獨立部署的Master web UI（http://<master url>:8080）中找到driver ID。

## 資源調度

獨立佈署的集群模式僅僅支持簡單的FIFO調度器。然而，為了允許多個並行的用户，你能夠控制每個應用程序能用的最大資源數。在默認情况下，它將獲得集群的所有核，這只有在某一時刻只
允許一个應用程序才有意義。你可以通過`spark.cores.max`在[SparkConf](http://spark.apache.org/docs/latest/configuration.html#spark-properties)中設置核數。

```scala
val conf = new SparkConf()
             .setMaster(...)
             .setAppName(...)
             .set("spark.cores.max", "10")
val sc = new SparkContext(conf)
```
另外，你可以在集群的master进程中配置`spark.deploy.defaultCores`來改變默認的值。在`conf/spark-env.sh`添加下面的行：

```properties
export SPARK_MASTER_OPTS="-Dspark.deploy.defaultCores=<value>"
```

這在用户沒有配置最大核數的共享集群中是有用的。

## 高可用

默認情况下，獨立的調度集群對worker失敗是有彈性的（在Spark本身的範圍内是有彈性的，對丢失的工作通過轉移它到另外的worker來解決）。然而，調度器通過master去執行調度决定，
這會造成單點故障：如果master死了，新的應用程序就無法創建。為了避免這個，我们有兩個高可用的模式。

### 用ZooKeeper的備用master

利用ZooKeeper去支持领导选举以及一些狀態存储，你能夠在你的集群中啟動多個master，這些master連接到同一個ZooKeeper实例上。一個被選为“领导”，其它的保持備用模式。如果當前
的領導死了，另一个master將會被選中，恢復老master的狀態，然後恢復調度。整個的恢復進程大概需要1到2分鐘。注意，這個恢復時間僅僅會影響調度新的應用程序-運行在失敗master中的
應用程序不受影響。

#### 配置

為了開啟這個恢復模式，你可以用下面的屬性在`spark-env`中設置`SPARK_DAEMON_JAVA_OPTS`。

System property | Meaning
--- | ---
spark.deploy.recoveryMode | 設置ZOOKEEPER去啟動備用master模式（默認為none）
spark.deploy.zookeeper.url | zookeeper集群url(如192.168.1.100:2181,192.168.1.101:2181)
spark.deploy.zookeeper.dir | zookeeper保存恢復狀態的目錄（默認是/spark）

可能的陷阱：如果你在集群中有多个masters，但是沒有用zookeeper正确的配置這些masters，這些masters不會發現彼此，會認為它們都是leaders。這將會造成一個不健康的集群狀態（因为所有的master都會獨立的調度）。

#### 細節

zookeeper集群啟動之後，開啟高可用是簡單的。在相同的zookeeper配置（zookeeper URL和目錄）下，在不同的節點上簡單地啟動多個master进程。master可以随時添加和删除。

為了調度新的應用程序或者添加worker到集群，它需要知道當前leader的IP地址。這可以通過簡單的傳遞一个master列表來完成。例如，你可能啟動你的SparkContext指向`spark://host1:port1,host2:port2`。
這將造成你的SparkContext同时注册这两个master-如果`host1`死了，這個配置文件將一直是正確的，因为我們將找到新的leader-`host2`。

"registering with a Master"和正常操作之間有重要的區别。當啟動时，一個應用程序或者worker需要能夠發現和註册當前的leader master。一旦它成功註册，它就在系统中了。如果
錯誤發生，新的leader將會接觸所有之前註册的應用程序和worker，通知他們领导關係的變化，所以它們甚至不需要事先知道新啟動的leader的存在。

由於這個屬性的存在，新的master可以在任何時候創建。你唯一需要擔心的問題是新的應用程序和workers能夠發現它並將它註册進来以防它成為leader master。

### 用本地文件系统做單節點恢復

zookeeper是生產環境下最好的選擇，但是如果你想在master死掉後重啟它，`FILESYSTEM`模式可以解決。當應用程序和worker註册，它们擁有足夠的狀態寫入提供的目錄，以至於在重啟master
進程時它们能夠恢復。

#### 配置

為了開啟這個恢復模式，你可以用下面的屬性在`spark-env`中設置`SPARK_DAEMON_JAVA_OPTS`。

System property | Meaning
--- | ---
spark.deploy.recoveryMode | 設置为FILESYSTEM開啟單節點恢復模式（默認为none）
spark.deploy.recoveryDirectory | 用來恢復狀態的目錄

#### 細節

- 這個解决方案可以和監控器/管理器（如[monit](http://mmonit.com/monit/)）相配合，或者僅僅通過重啟開啟手動恢復。
- 雖然文件系统的恢復似乎比沒有做任何恢復要好，但對於特定的開發或實驗目的，這種模式可能是次優的。特别是，通過`stop-master.sh`殺掉master不會清除它的恢復狀態，所以，不管你何時啟動一個新的master，它都將進入恢復模式。這可能使啟動時間增加到1分鐘。
- 雖然它不是官方支持的方式，你也可以創建一個NFS目錄作为恢復目錄。如果原始的master節點完全死掉，你可以在不同的節點啟動master，它可以正確的恢復之前註册的所有應用程序和workers。未來的應用程序會發現這個新的master。