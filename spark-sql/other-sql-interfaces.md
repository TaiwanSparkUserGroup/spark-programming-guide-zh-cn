# [其它SQL接口](https://spark.apache.org/docs/latest/sql-programming-guide.html#other-sql-interfaces)

Spark SQL 也支持直接運行 SQL 查詢的接口，不用寫任何代碼。

## 運行 Thrift JDBC/ODBC 伺服器

這裡實現的 Thrift JDBC/ODBC 伺服器與 Hive 0.12中的[HiveServer2](https://cwiki.apache.org/confluence/display/Hive/Setting+Up+HiveServer2)一致。你可以用在 Spark 
或者 Hive 0.12 附帶的 beeline 腳本測試 JDBC 伺服器。

在 Spark 目錄中，運行下面的命令啟動 JDBC/ODBC 伺服器。

```shell
./sbin/start-thriftserver.sh
```

這個腳本接受任何的 `bin/spark-submit` 命令行參數，加上一個 `--hiveconf` 參數用來指明 Hive 屬性。你可以運行 `./sbin/start-thriftserver.sh --help` 來獲得所有可用選項的完整列表。預設情况下，伺服器監聽 `localhost:10000` 。你可以用環境變數附蓋這些變數。

```shell
export HIVE_SERVER2_THRIFT_PORT=<listening-port>
export HIVE_SERVER2_THRIFT_BIND_HOST=<listening-host>
./sbin/start-thriftserver.sh \
  --master <master-uri> \
  ...
```
或者透過系統變數覆蓋。

```shell
./sbin/start-thriftserver.sh \
  --hiveconf hive.server2.thrift.port=<listening-port> \
  --hiveconf hive.server2.thrift.bind.host=<listening-host> \
  --master <master-uri>
  ...
```
現在你可以用 `beeline` 測試 Thrift JDBC/ODBC 伺服器。

```shell
./bin/beeline
```

連接到 Thrift JDBC/ODBC 伺服器的方式如下：

```shell
beeline> !connect jdbc:hive2://localhost:10000
```

Beeline 將會詢問你的用戶名稱和密碼。在非安全的模式，簡單地輸入你機器的用戶名稱和空密碼就行了。對於安全模式，你可以按照[Beeline文檔](https://cwiki.apache.org/confluence/display/Hive/HiveServer2+Clients)的說明來執行。

## 運行 Spark SQL CLI

Spark SQL CLI 是一個便利的工具，它可以在本地運行 Hive 元儲存(metastore)服務、執行命令行輸入的查詢。注意，Spark SQL CLI不能與 Thrift JDBC 伺服器通信。

在 Spark 目錄運行下面的命令可以啟動 Spark SQL CLI。

```shell
./bin/spark-sql
```

