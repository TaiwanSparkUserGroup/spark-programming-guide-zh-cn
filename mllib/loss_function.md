# 損失函數
下表概述了MLlib支持的損失函數及其梯度和子梯度：


|   	|  損失函數$$L(w;x,y)$$	|  梯度或子梯度 	|
|---	|---	|---	|
| hinge loss  	|$$max\{0, 1-yw^Tx\} $$   	|  $$ \left\{ \begin{array}{ll}  -yx ~~~ 若~yw^Tx \lt1,   \\  0 ~~~~~~~~~otherwise \end{array} \right.$$|
|  logistic loss 	|$$log(1+exp(-yw^Tx)),~y \in \{-1,+1\}$$   	|$$-y(1-\frac{1}{1+exp(-yw^Tx)})x$$   	|
|   squared loss	| $$\frac{1}{2}(x^Tx-y)^2,~y \in R$$  	|  $$(w^Tx-y)x$$ 	|
