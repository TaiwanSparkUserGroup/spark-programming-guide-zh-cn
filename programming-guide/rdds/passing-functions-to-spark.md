# 傳遞函數到 Spark

Spark 的 API 大多數是依靠在驅動程式裡傳遞函數到集群上運作，目前有兩種推薦方式：

<<<<<<< HEAD
- [ 匿名函數  (Anonymous function syntax)](http://docs.scala-lang.org/tutorials/tour/anonymous-function-syntax.html)，可在較短的程式碼中使用。
=======
- [ 匿名函數  (Anonymous function syntax)](http://docs.scala-lang.org/tutorials/tour/anonymous-function-syntax.html)，可在較短的程式碼中使用。
>>>>>>> 6671e10e0d8770aa9448556ebe999929fbfc6321
- 全局單例物件裡的靜態方法。例如，定義 `object MyFunctions` 然後傳遞 `MyFounctions.func1`，例如：

```scala
object MyFunctions {
  def func1(s: String): String = { ... }
}

myRdd.map(MyFunctions.func1)
```

注意，它可能傳遞的是一個類別實例裡面的一個方法(非一個單例物件)，在這裡，必須傳送包含方法的整個物件。例如：

```scala
class MyClass {
  def func1(s: String): String = { ... }
  def doStuff(rdd: RDD[String]): RDD[String] = { rdd.map(func1) }
}
```

如果我們建立一個 `new MyClass` 物件，並且使用它的 `doStuff`，`map` 裡面引用了這個 `MyClass` 中的 `func1` 方法，所以這個物件必須也傳送到集群上。類似寫成 `rdd.map(x => this.func1(x))`。

以同樣的方式，存取外部物件的變數將會引用整個物件：

```scala
class MyClass {
  val field = "Hello"
  def doStuff(rdd: RDD[String]): RDD[String] = { rdd.map(x => field + x) }
}
```

等於寫成 `rdd.map(x => this.field + x)`，引用整個 `this` 物件。為了避免這個問題，最簡單的方式是複製 `field` 到一個本地變數而不是從外部取用：

```scala
def doStuff(rdd: RDD[String]): RDD[String] = {
  val field_ = this.field
  rdd.map(x => field_ + x)
}
```
