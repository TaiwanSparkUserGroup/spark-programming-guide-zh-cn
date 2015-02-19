# 線性支持向量機(SVMs)
對於大規模的分類任務來說，[線性支持向量機](http://en.wikipedia.org/wiki/Support_vector_machine#Linear_SVM)是標準的方法。它是之前"數學公式"一節中所描述的線性方法，廿金金損失函數是hinge loss:
$$
L(w;x,y)=\max\{0,1-yw^Tx\}.
$$
預設配置下，線性SVM使用L2正則化訓練。我們支持L1正則化。通過這種方式，問題變為線性規劃問題。

線性SVM算法是產出一個SVM模型。給定新數據點$$x$$，該模型基於$$w^Tx$$的值來預測。默認情形下，$$w^Tx\geq0$$時為正例，否則為反例。
