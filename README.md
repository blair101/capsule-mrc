# capsule-mrc
基于capsule的观点型阅读理解模型,应用于[观点型阅读理解](https://challenger.ai/competition/oqmrc2018)。单模不加trick、不用外部词向量，直接使用本模型的得分约74.2。
本模型认为观点型阅读理解的任务目标是聚类，即从passage和query中提取信息，以候选答案为中心进行capsule动态路由聚类；若某个候选答案得到的信息越多，则该答案模长越长，作为答案的概率越高。

# 数据样例
{
“query_id”:1,
“query”:“维生c可以长期吃吗”,
“url”: “xxx”,
“passage”: “每天吃的维生素的量没有超过推荐量的话是没有太大问题的。”,
“alternatives”:”可以|不可以|无法确定”,
“answer”:“可以”
}

# 模型图
![pic1](https://github.com/freefuiiismyname/capsule-mrc/blob/master/capsuleNet-mrc/1.png)

# 模型思路
**步骤1、问题编码**
  
使用bi-LSTM对query进行编码，它将作为passage和候选答案的背景信息。（无论是passage还是候选答案，都是基于query出现的，故而query应当作为两者的上下文） 

**步骤2、形成候选答案的各自意义**
  
使用bi-LSTM对三个候选答案进行编码，以步骤1输出的state作为lstm的初始状态，使它们拥有问题的上下文信息。编码后将每个候选答案看作capsule，分别代表了三个不同事件。

**步骤3、形成passage对问题的理解**
  
对passage进行LSTM（使passage每个词语拥有上下文意思，state初始化同上）、match(与问题信息交互)、fuse（信息融合）、cnn（抽取关键信息）之后，形成N个特征capsule，代表了passage根据问题而抽取出的信息。

**步骤4、以候选答案为中心，对passage信息进行聚类**
  
将passage中抽取出的信息，转换为候选答案capsule。当某答案编码与passage信息相近时，信息更容易为它提供支撑；反之，它受到的支撑将减少。经过几轮的动态路由迭代过程后，最终capsule的模长代表了该答案存在的程度。Softmax后，求出每个候选答案作为答案的概率。


# 总结
该模型在本次比赛的成绩，单模为74点几，并不是特别好。归其原因，主要是本次比赛更偏向于分类任务。我们根据候选答案来将本次比赛分为了两类数据，一类是【A，不A，无法确定】，另一类是【A，B，无法确定】。很显然前者类似于【正面|负面|无法确定】的分类任务，而且它的占比达到了90%。我们试验时将该比赛完全当作分类模型来做，效果并没有太差。
我们认为本模型应该能在更复杂的观点型阅读理解（如【A，B，无法确定】或不定量候选答案的【A，B，C……】或）中会有更好的表现，也对此抱有期待。
