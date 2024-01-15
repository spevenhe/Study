# Data Augmentation	
## Simple Label-Preserving Transformations	
![image](https://github.com/spevenhe/Study/assets/42630862/dce8ecc6-9c48-4b6c-a848-3c3aafdf0c5b)
![image](https://github.com/spevenhe/Study/assets/42630862/f8f87709-8a88-4dde-82c6-ad8d03458bc0)


## Perturbation	扰动
Perturbation is also a label-preserving operation, but because sometimes, it’s used to trick models into making wrong predictions
![image](https://github.com/spevenhe/Study/assets/42630862/ad31c229-a56e-4093-a7c0-5dd6fd8c6720)
![image](https://github.com/spevenhe/Study/assets/42630862/9cbc1927-8cd0-410d-8019-a1458744a4ea)


## Data Synthesis	
![image](https://github.com/spevenhe/Study/assets/42630862/cb9baa50-2fc6-45ab-92a8-292adfb0fd93)
![image](https://github.com/spevenhe/Study/assets/42630862/23ad7fc0-2088-46b3-b783-14a395b5cdd9)



# Learned Features vs. Engineered Features	
对于深度学习：不需要，因为 deep learning 也被叫做 feature engineering
对于传统机器学习： 需要专家研究feature

![image](https://github.com/spevenhe/Study/assets/42630862/860927d0-b7cf-40e7-8651-eeae60f93eea)


# Common Feature Engineering Operations	

## Handling Missing Values

### Deletion

1. column deletion

2. row deletion
这两种方式都有风险，可能会遗失关键信息，因为数据之间可能据有关连，比如一个女性不愿意透露年龄，一个流浪汉不愿的房产可能是空

### Imputation
![image](https://github.com/spevenhe/Study/assets/42630862/befdcb83-67a3-4f91-a5a6-7daa6fdc627d)


## Scaling
![image](https://github.com/spevenhe/Study/assets/42630862/04e55276-8fcd-4d14-b5b6-61508452a21f)


## Discretization 离散化
![image](https://github.com/spevenhe/Study/assets/42630862/bfe150e0-cbbf-43a4-8d69-f219ca8527d6)



## Encoding Categorical Features
categories 是动态的，会一直变化
把新的feature都标记为unknow 会导致无法正确推荐

两种解决方法
1. Represent each category with its attribute
   
E.g. to represent a brand, use features: yearly revenue, company size, etc..

2. Hashing trick
预留组够大的空间避免碰撞。这个学术界不是很支持这种做法，但是工业界是这样用的，


2.1 Choose a hash space large enough to reduce collisions

2.2 Choose functions with properties beneficial to your use case

Locality-sensitive hashing



## Feature Crossing
![image](https://github.com/spevenhe/Study/assets/42630862/240657d2-156f-403c-8bf4-8e76d17ac157)
![image](https://github.com/spevenhe/Study/assets/42630862/b7d1a82d-9279-4963-a3ea-e1ea27bd20e0)


## Discrete and Continuous Positional Embeddings（位置嵌入）
Positional Embeddings 来自于 论文 Attention Is All You Need https://arxiv.org/abs/1706.03762
介绍：
文本是时序型数据，词与词之间的顺序关系往往影响整个句子的含义。这里我整理了一些顺序不同，含义不同的例子。

传统的RNN模型在处理句子时，以序列的模式逐个处理句子中的词语，这使得词语的顺序信息在处理过程中被天然的保存下来了，并不需要额外的处理。

而对于Transformer来说，由于句子中的词语都是同时进入网络进行处理，顺序信息在输入网络时就已丢失。因此，Transformer是需要额外的处理来告知每个词语的相对位置的。论文《Attention is all you need》https://arxiv.org/abs/1706.03762 中提到的一个Positional Encoding（位置编码）公式如下，它能将表示位置信息的编码添加到输入中，让网络知道每个词的位置和顺序。


**positional embedding has become a standard data engineering technique for many applications in both computer vision and natural language processing**

![image](https://github.com/spevenhe/Study/assets/42630862/07e4f640-71ca-438e-94cc-39a354ae8dec)




















