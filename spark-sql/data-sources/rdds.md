# [RDDs](https://spark.apache.org/docs/latest/sql-programming-guide.html#rdds)

Spark 支持兩種方法將存在的 RDDs 轉換為 SchemaRDDs 。第一種方法使用映射來推斷包含特定對象類型的RDD模式(schema)。在你寫 spark 程序的同時，當你已經知道了模式，這種基於映射的方法可以使代碼更簡潔並且程序工作得更好。

創建 SchemaRDDs 的第二種方法是通過一個編程接口來實現，這個接口允许你建構一個模式，然後在存在的 RDDs 上使用它。雖然這種方法更冗長，但是它允許你在運行期之前不知道列以及列的類型的情况下構造 SchemaRDDs。

## 利用映射推断(inffering)模式(scheme)

Spark SQL的Scala接口支持將包含樣本類(case class)的 RDDs 自動轉換為 SchemaRDD。這個樣本類(case class)定義了表的模式。

給樣本類的參數名字通過映射來讀取，然後作為列的名字。樣本類可以嵌套或者包含複雜的類型如序列或者數組。這個 RDD 可以隱式轉化為一個 SchemaRDD ，然後註冊為一個表。表可以在後續的 sql 語句中使用。

```scala
// sc is an existing SparkContext.
val sqlContext = new org.apache.spark.sql.SQLContext(sc)
// createSchemaRDD is used to implicitly convert an RDD to a SchemaRDD.
import sqlContext.createSchemaRDD

// Define the schema using a case class.
// Note: Case classes in Scala 2.10 can support only up to 22 fields. To work around this limit,
// you can use custom classes that implement the Product interface.
case class Person(name: String, age: Int)

// Create an RDD of Person objects and register it as a table.
val people = sc.textFile("examples/src/main/resources/people.txt").map(_.split(",")).map(p => Person(p(0), p(1).trim.toInt))
people.registerTempTable("people")

// SQL statements can be run by using the sql methods provided by sqlContext.
val teenagers = sqlContext.sql("SELECT name FROM people WHERE age >= 13 AND age <= 19")

// The results of SQL queries are SchemaRDDs and support all the normal RDD operations.
// The columns of a row in the result can be accessed by ordinal.
teenagers.map(t => "Name: " + t(0)).collect().foreach(println)
```

## [编程指定模式](https://spark.apache.org/docs/latest/sql-programming-guide.html#programmatically-specifying-the-schema)

當樣本類不能提前確定（例如，記錄的結構是經過編碼的字串，或者一個文本集合將會被解析，不同的字段投射给不同的用戶），一个 SchemaRDD 可以通过三步来创建。

- 從原來的 RDD 創建一個行的 RDD
- 創建由一个 `StructType` 表示的模式與第一步創建的 RDD 的行結構相匹配
- 在行 RDD 上通過 `applySchema` 方法應用模式

```scala
// sc is an existing SparkContext.
val sqlContext = new org.apache.spark.sql.SQLContext(sc)

// Create an RDD
val people = sc.textFile("examples/src/main/resources/people.txt")

// The schema is encoded in a string
val schemaString = "name age"

// Import Spark SQL data types and Row.
import org.apache.spark.sql._

// Generate the schema based on the string of schema
val schema =
  StructType(
    schemaString.split(" ").map(fieldName => StructField(fieldName, StringType, true)))

// Convert records of the RDD (people) to Rows.
val rowRDD = people.map(_.split(",")).map(p => Row(p(0), p(1).trim))

// Apply the schema to the RDD.
val peopleSchemaRDD = sqlContext.applySchema(rowRDD, schema)

// Register the SchemaRDD as a table.
peopleSchemaRDD.registerTempTable("people")

// SQL statements can be run by using the sql methods provided by sqlContext.
val results = sqlContext.sql("SELECT name FROM people")

// The results of SQL queries are SchemaRDDs and support all the normal RDD operations.
// The columns of a row in the result can be accessed by ordinal.
results.map(t => "Name: " + t(0)).collect().foreach(println)
```
