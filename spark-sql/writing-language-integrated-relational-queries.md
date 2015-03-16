# [編寫語言整合(Language-Integrated)的關聯性查詢](https://spark.apache.org/docs/latest/sql-programming-guide.html#writing-language-integrated-relational-queries)

**語言整合的關聯性查詢是實驗性的，現在暂时只支援scala。**

Spark SQL也支持用特定領域的語法編寫查詢語句，請參考使用下列範例的資料：

```scala
// sc is an existing SparkContext.
val sqlContext = new org.apache.spark.sql.SQLContext(sc)
// Importing the SQL context gives access to all the public SQL functions and implicit conversions.
import sqlContext._
val people: RDD[Person] = ... // An RDD of case class objects, from the first example.

// 下述等同於 'SELECT name FROM people WHERE age >= 10 AND age <= 19'
val teenagers = people.where('age >= 10).where('age <= 19).select('name)
teenagers.map(t => "Name: " + t(0)).collect().foreach(println)
```

DSL使用Scala的符號來表示在潜在表(underlying table)中的列，這些列以前缀(')標示。將這些符號透過隱式轉成由SQL執行引擎計算的表達式。你可以在[ScalaDoc](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.SchemaRDD)
中了解詳情。
