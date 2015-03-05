# Spark獨立布署模式

## 安裝Spark獨立模式集群

欲安裝 Spark獨立模式，只需要將Spark的編譯版本放到集群的每個節點。你可以取得穩定版本的編譯本，也可以自己編譯。

## 手動啟動集群

你可以透過下列方式啟動獨立的master服務器。

```shell
./sbin/start-master.sh
```

啟動以後，master會印出`spark://HOST:PORT` URL，就可以用它連接到workers或是作為"master"參數傳遞到`SparkContext`。當然，你也可以在master web UI上找到這個URL。master web UI預設的位址是`http://localhost:8080`。

同樣的，你也可以啟動一個或是多個workers，或將他們連接到master。

```shell
./bin/spark-class org.apache.spark.deploy.worker.Worker spark://IP:PORT
```

當你啟動一個worker，就可查看master web UI。就可以看到新節點的增加、各節點的CPU數量以及記憶體大小。

下列的參數設定可以傳送給master與worker。

Argument | Meaning
--- | ---
-h HOST, --host HOST | 監聽的主機名稱
-i HOST, --ip HOST | 同上，已被淘汰
-p PORT, --port PORT | 監聽的服務接口（master預設是7077，worker隨機）
--webui-port PORT | web UI的接口(master預設是8080，worker預設是8081)
-c CORES, --cores CORES | Spark應用程式可用的CPU數量（預設是所有可用）；這個選項只能在在worker上用
-m MEM, --memory MEM | Spark應用程式可以使用的記憶體數量（預設是你的機器記憶體數量減1g）；這個選項只能在在worker上用
-d DIR, --work-dir DIR | 用于暫存空間和工作輸出日誌的目錄（預設是SPARK_HOME/work）；這個選項只能在在worker上用
--properties-file FILE | 自定的Spark配置文件的加掛目錄（預設是conf/spark-defaults.conf）

## 集群啟動程式

為了用啟動程式來啟動Spark獨立集群，應該要在你的Spark目錄下建立一個名為`conf/slaves`的文件，這個文件必須包含所有你要啟動的Spark worker所在機器的主機名稱，一行一個。如果`conf/slaves`不存在，啟動程式會將之預設為單台機器（localhost），這樣適用測試環境。注意，master機器通過ssh連結所有的worker。在預設情況下，ssh是併行運作，所以需要設定無密碼遠端登入。

如果你沒有設定無密碼遠端登入，可以設定環境變數`SPARK_SSH_FOREGROUND`，讓每個worker取得密碼。

當你設定了這個文件，就可以通過下列的shell程式啟動或是停止你的集群，

- sbin/start-master.sh：在機器上啟動master實體
- sbin/start-slaves.sh：在每台機器上啟動一個slave實體
- sbin/start-all.sh：同時啟動一個master實體和所有slave實體
- sbin/stop-master.sh：停止master實體
- sbin/stop-slaves.sh：停止所有slave實體
- sbin/stop-all.sh：停止master實體和所有slave實體

注意，這些程式必須在你的Spark master執行的該台機器上執行，而不是在你的本機上頭。

你可以在`conf/spark-env.sh`中設定環境變數以利進一步配置集群。利用`conf/spark-env.sh.template`建立這個文件以後，將他複製到所有workers機器上讓設定生效。

所以下列的設定可以生效：

Environment Variable | Meaning
--- | ---
SPARK_MASTER_IP | 綁定master到一個指定的ip位址
SPARK_MASTER_PORT | 在不同的接口上啟動master（預設是7077）
SPARK_MASTER_WEBUI_PORT | master web UI的端口（預設是8080）
SPARK_MASTER_OPTS | 應用到master的配置屬性，格式是 "-Dx=y"（預設是none），查看下面的表格的選項以組成一個可能的列表
SPARK_LOCAL_DIRS | Spark中暫存的目錄。包括map的輸出文件和儲存在硬碟上的RDDs(including map output files and RDDs that get stored on disk)。這必須在一個快速的，你的系統的本地硬碟上。他可以是一個逗號分隔的列表，代表不同硬碟的多個目錄
SPARK_WORKER_CORES | Spark应用程序可以用到的核心数（預設是所有可用）
SPARK_WORKER_MEMORY | Spark應用程式用到的記憶體總數（預設是記憶體數量減1G）。注意，每個應用程式的記憶體是透過`spark.executor.memory`設定
SPARK_WORKER_PORT | 在指定接口上啟動Spark worker(預設是隨機)
SPARK_WORKER_WEBUI_PORT | worker UI的接口（預設是8081）
SPARK_WORKER_INSTANCES | 每台機器上執行的worker實體數，預設是1。如果你有一台非常大的機器並且希望運行多個worker，你可以設定這個數字大於1。如果你設定這個環境變數，也要確保你已設定了`SPARK_WORKER_CORES`環境變數以利限制每個 worker的CPU數量或是每個worker嘗試使用最多的CPU。
SPARK_WORKER_DIR | Spark worker運行目錄，該目錄包括日誌以及暫存空間（預設是SPARK_HOME/work）
SPARK_WORKER_OPTS | 應用到worker的配置屬性，格式是 "-Dx=y"（預設是none），查看下面表格的選項以組成一個可能的列表
SPARK_DAEMON_MEMORY | 分配給Spark master和worker保護進程的記憶體（預設是512m）
SPARK_DAEMON_JAVA_OPTS | Spark master和worker保護進程的JVM選項，格式是"-Dx=y"（預設是none）
SPARK_PUBLIC_DNS | Spark master和worker公用的DNS名稱（預設是none）

注意，啟動程式尚未支持windows。如要在windows上啟動Spark集群，需要手動啟動master和workers。

`SPARK_MASTER_OPTS`支持一下的系統屬性：

Property Name | Default | Meaning
--- | --- | ---
spark.deploy.retainedApplications | 200 | 展示完成的应用程序的最大数目。老的应用程序会被删除以满足该限制
spark.deploy.retainedDrivers | 200 | 展示完成的drivers的最大数目。老的应用程序会被删除以满足该限制
spark.deploy.spreadOut | true | 这个选项控制独立的集群管理器是应该跨节点传递应用程序还是应努力将程序整合到尽可能少的节点上。在HDFS中，传递程序是数据本地化更好的选择，但是，对于计算密集型的负载，整合会更有效率。
spark.deploy.defaultCores | (infinite) | 在Spark独立模式下，给应用程序的默认核数（如果没有设置`spark.cores.max`）。如果没有设置，应用程序总数获得所有可用的核，除非设置了`spark.cores.max`。在共享集群上设置较低的核数，可用防止用户默认抓住整个集群。
spark.worker.timeout | 60 | 独立部署的master认为worker失败（没有收到心跳信息）的间隔时间。

`SPARK_WORKER_OPTS`支持的系统属性：

Property Name | Default | Meaning
--- | --- | ---
spark.worker.cleanup.enabled | false | 周期性的清空worker/应用程序目录。注意，这仅仅影响独立部署模式。不管应用程序是否还在执行，用于程序目录都会被清空
spark.worker.cleanup.interval | 1800 (30分) | 在本地机器上，worker清空老的应用程序工作目录的时间间隔
spark.worker.cleanup.appDataTtl | 7 * 24 * 3600 (7天) | 每个worker中应用程序工作目录的保留时间。这个时间依赖于你可用磁盘空间的大小。应用程序日志和jar包上传到每个应用程序的工作目录。随着时间的推移，工作目录会很快的填满磁盘空间，特别是如果你运行的作业很频繁。

## 连接一个应用程序到集群中

为了在Spark集群中运行一个应用程序，简单地传递`spark://IP:PORT` URL到[SparkContext](http://spark.apache.org/docs/latest/programming-guide.html#initializing-spark)

为了在集群上运行一个交互式的Spark shell，运行一下命令：

```shell
./bin/spark-shell --master spark://IP:PORT
```
你也可以传递一个选项`--total-executor-cores <numCores>`去控制spark-shell的核数。

## 启动Spark应用程序

[spark-submit脚本](submitting-applications.md)支持最直接的提交一个Spark应用程序到集群。对于独立部署的集群，Spark目前支持两种部署模式。在`client`模式中，driver启动进程与
客户端提交应用程序所在的进程是同一个进程。然而，在`cluster`模式中，driver在集群的某个worker进程中启动，只有客户端进程完成了提交任务，它不会等到应用程序完成就会退出。

如果你的应用程序通过Spark submit启动，你的应用程序jar包将会自动分发到所有的worker节点。对于你的应用程序依赖的其它jar包，你应该用`--jars`符号指定（如` --jars jar1,jar2`）。

另外，`cluster`模式支持自动的重启你的应用程序（如果程序一非零的退出码退出）。为了用这个特征，当启动应用程序时，你可以传递`--supervise`符号到`spark-submit`。如果你想杀死反复失败的应用，
你可以通过如下的方式：

```shell
./bin/spark-class org.apache.spark.deploy.Client kill <master url> <driver ID>
```

你可以在独立部署的Master web UI（http://<master url>:8080）中找到driver ID。

## 资源调度

独立部署的集群模式仅仅支持简单的FIFO调度器。然而，为了允许多个并行的用户，你能够控制每个应用程序能用的最大资源数。在默认情况下，它将获得集群的所有核，这只有在某一时刻只
允许一个应用程序才有意义。你可以通过`spark.cores.max`在[SparkConf](http://spark.apache.org/docs/latest/configuration.html#spark-properties)中设置核数。

```scala
val conf = new SparkConf()
             .setMaster(...)
             .setAppName(...)
             .set("spark.cores.max", "10")
val sc = new SparkContext(conf)
```
另外，你可以在集群的master进程中配置`spark.deploy.defaultCores`来改变默认的值。在`conf/spark-env.sh`添加下面的行：

```properties
export SPARK_MASTER_OPTS="-Dspark.deploy.defaultCores=<value>"
```

這在使用者沒設定最大CPU數量的共享集群中是有用的。

## 高可用

預設情況，獨立的調度集群對workers失敗是有彈性的處理（在Spark本身的範圍內有彈性，對漏失的工作任務會被轉移到另外的worker來解決）。然而，調度器通過master去執行調度再決定，這會造成單點故障：如果master掛了，新的應用程式就無法被創見。為了避免這種情況，我們有兩個高可用模式：

### 用ZooKeeper的備用master

利用ZooKeeper去支持領導候選以及一些狀態儲存，你可以在你的集群中啟動多個master，這些master會連接到同一個ZooKeeper上。其中一個被選為“主控者”，其他的為備用模式。如果當前主控者掛了，會推出另一個master做新的主控者，恢復原本master的狀態，然後繼續任務調度。整個復原過程約需1到2分鐘。注意，這個復原時間只會影響新應用程式，運作在掛點master中的應用程式不被影響。

#### 設定

為了開啟這個復原模式，你可以用下列的屬性在`spark-env`中設定`SPARK_DAEMON_JAVA_OPTS`參數。

System property | Meaning
--- | ---
spark.deploy.recoveryMode | 設定ZOOKEEPER去啟動備用master模式（預設none）
spark.deploy.zookeeper.url | zookeeper集群url(如192.168.1.100:2181,192.168.1.101:2181)
spark.deploy.zookeeper.dir | zookeeper保存復原狀態的目錄（預設是/spark）

你可能遇到的陷阱：如果在集群中有多個masters，但是沒有用ZooKeeper設定正確的參數，這些master不會知道彼此存在，反而認為他們都是主控者。這會造成集群不健康的狀態，因為所有的master都會獨立調度任務。

#### 細節

ZooKeeper集群被啟動以後，要開啟高啟用是很方便的。在相同的ZooKeeper設定下，在不同的節點上簡單的啟動多個master程式。master可以隨時作增加刪減動作。

為了調度新的應用程式或是增加worker到集群，他需要知道當前主控者的IP位址。這可以通過簡單的傳遞一個master列表完成。例如你可能啟動你的SparkContext指向`spark://host1:port1,host2:port2`。
這會造成你的SparkContext同時註冊兩個master-如果`host1`掛了，這個設定文件將一直是正確的，因為我們會選出新的leader-`host2`。

"registering with a Master"和正常操作之間有重要的區別。當啟動時，一個應用程式或是worker需要能夠發現和註冊當前的leader master。一旦它成功註冊，它就被放在系統裡。如果發生錯誤，新的leader會接觸所有之前註冊過的應用程式和worker，通知他們leader掛了，所以他們甚至不用事先知道新啟動的主控者存在。

由於存在這種屬性，新的master可以在任何時候被建立。你唯一要注意的是新的應用程式和workers能夠找到它並且把它註冊以防止它變成leader master。

### 用本地文件系統做單點復原

zookeeper是生产环境下最好的选择，但是如果你想在master死掉后重启它，`FILESYSTEM`模式可以解决。当应用程序和worker注册，它们拥有足够的状态写入提供的目录，以至于在重启master
进程时它们能够恢复。

ZooKeeper是生產環境下最好的選擇，但如果你想在master掛掉後重啟，`FILESYSTEM`模式可以幫你解決。當應用程式和worker註冊，他們擁有足夠的狀態寫入目錄，以致於在重啟master時一起恢復。

#### 配置

為了啟動這個復原模式，你可以用下列的屬性在`spark-env`中設定`SPARK_DAEMON_JAVA_OPTS`。

System property | Meaning
--- | ---
spark.deploy.recoveryMode | 設定為FILESYSTEM開啟單點復原模式（預設none）
spark.deploy.recoveryDirectory | 用來恢復狀態的目錄

#### 細節

- 這個解決方案可以和監控器/管理器（如[monit](http://mmonit.com/monit/)）互相配合，或者單單通過重開手動恢復。
- 雖然文件系統的復原比什麼都沒了好，但對於特定開發目的，這種模式可能不是最好的。特別是，通過`stop-master.sh`殺掉master不會清除它的復原狀態，所以，不管你何時啟動一個新的master，它都會進入復原模式，可能讓啟動時間增加1分鐘。
- 雖然他不是官方建議的模式，你還是可以建立一個NFS目錄作為復原目錄。如果最原本的master掛了，你可以在不同節點上啟動master，它還是可以正確復原之前註冊的所有應用程式和workers。下一個應用程式近來就會發現這個新的master。
