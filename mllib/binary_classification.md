# 二元分類
[二元分類](http://en.wikipedia.org/wiki/Logistic_regression)將數據項分成兩類： 正例及負例。MLlib支持兩種二元分類的線性方法：線性支持向量機與邏輯斯迴歸。對這兩種方法來說，MLlib都支持L1, L2正則化。在MLlib中，訓練數据集用一個LabeledPoint格式的RDD來表示。需要注意，本指南中的數學公式裡，約定訓練標簽$$y$$為+1(正例)或-1(反例)，但在MLlib中，為了與多煩標簽保持一致，反例標簽是0, 而不是-1。
