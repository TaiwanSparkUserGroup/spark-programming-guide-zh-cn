#Optimized Representation

一些較高層次的理解可能對於演算法設計和是用API有幫助，然而在這個章節並不會介紹Optimization太多的細節。GraphX在分散式Graph的分割是採用的Vertex-cut方法：
![Partitioning Approach](https://spark.apache.org/docs/latest/img/edge_cut_vs_vertex_cut.png)
而不是沿著Graph的邊來切，GraphX透過頂點來切能夠減少訊息交換和儲存上的負擔。邏輯上，這就像分配邊給機器且允許頂點橫跨多個機器。分配邊的方法是仰賴於[PartitionStrategy]()和多個不同的heuristics的權衡。使用者可以藉由[Graph.partitionBy]()運算子重心切割Graph，來選擇不同的策略。預設的切割策略是使用最初分割的邊當做Graph建立。然而，使用者能夠方便的轉換成2D-partitioning或式其他heuristics。
![SomeThing](https://spark.apache.org/docs/latest/img/vertex_routing_edge_tables.png)
一旦邊已經分割了，如何有效率作graph-parallel運算的主要關鍵是有效率的將頂點屬性和邊join起來。因為在現實的圖中，邊的數量多於頂點，我們將頂點的屬性移至邊上。因為並非所有分區都會包含所有頂點相鄰的邊，當要實作join所需的運算子像triplets和aggregateMessages時，我們內部會有一個routing table來識別這些頂點的位置。
