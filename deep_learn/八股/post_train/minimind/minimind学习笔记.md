
# 先对minimind进行一些复现



大模型训练为什么使用交叉熵，$H(P, Q) = -\sum_{x} P(x) \log Q(x)$，LLM的本质就是一个超大规模的分类问题：从词表中预测下一个词。对于分类问题，交叉熵是绝对的统治者，

norm，将值映射为，均值为0，方差为1中的值，防止梯度爆炸或者梯度消失。

RMSnorm 是 mata在llama模型中使用的。
传统的transformer中使用的是layernorm




[避开复数推导，我们还可以怎么理解RoPE？ - 知乎](https://zhuanlan.zhihu.com/p/863378538)