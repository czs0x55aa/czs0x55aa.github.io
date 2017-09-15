---
title: '[译]Keras Tutorial - Spoken Language Understanding'
date: 2017-06-24 14:50:22
tags: "自然语言处理"
---
[原文地址](https://chsasank.github.io/spoken-language-understanding.html)
该文章主要关于RNN来实现槽填充，比较基础的读物，这里只做粗略的翻译，一些细节还请看原文。  
原作者提供了Keras实现，近期自己打算用Pytorch重写一个版本。
按照原文重写的[Pytorch版本](https://github.com/czs0x55aa/pytorch-slot-filling)，代码写的并不好，主要是当做练习来写的，实验结果基本和原文的一致。
<!-- more -->
## Problem and the Dataset
我们要解决的问题是自然语言理解(NLU)。NLU的目标是提取语句的含义，然而这仍是个未解决的难题。  
因此，我们把这个问题简化为一个可解决的实际问题——理解说话者的一部分内容。  
具体来说，说话者要了解航班信息，那么我们期望去识别出这个意图。  

我们将使用Airline Travel Information System(ATIS)数据集，该数据集由DARPA在上世纪90年代收集的。  
ATIS数据集包含了航班相关的咨询语句。  
例如"I want to go from Boston to Atlanta on Monday."  
理解这句话，将其简化为可识别的参数，比如目的地和出发日，这个任务被成为槽填充(slot-filling)。  

下面是来自数据集的一个句子样例和对应的标签：  
![](译-Keras-Tutorial-Spoken-Language-Understanding/data_example.png)

ATIS官方数据分为4978条训练集和893条测试集，平均一句话长度为15个词，一共有128个类别(slots)标签。  

我们的方法将使用到如下技术：  
- Word embeddings
- Recurrent Neural Networks
下面几节将介绍这些信息。  

## Word Embeddings
word embeddings 将一个词映射到高维空间。  
理想情况下，embedding可以学习到语义信息，近义词的向量夹角会比较小，而无关词的向量夹角会比较大。  
embedding可以通过大量文本语料或放置于特定问题中学习，本文将采用第二种方法。
> Word Embeddings 中比较流行的是word2vecter和 glove，不过这里用的是与模型一起训练，这在神经网络模型中也很常见  

## Recurrent Neural Networks
卷积层可以很好的汇聚局部信息，但是他们无法捕获数据的顺序信息。循环神经网络可以帮助我们捕获序列信息，比如自然语言中的词序。  
如果我们要预测当前词所属的类别，我们最好去记住它之前的词。RNN单元中有包含了内部记忆单元，能够存储之前的序列信息。因此我们可以使用RNN来出来这类槽填充问题。  
对于这类问题，我们将使用词的embedding作为RNN的输入。  

## Putting it all together
### Loading Data
我们使用data.load.atisfull()来加载数据。第一次运行时会自动下载数据。  
将词和标签用词汇表的索引来代替，词表分别存储于w2idx和lables2idx。  
```python
import numpy as np
import data.load

train_set, valid_set, dicts = data.load.atisfull()
w2idx, labels2idx = dicts['words2idx'], dicts['labels2idx']

train_x, _, train_label = train_set
val_x, _, val_label = valid_set

# Create index to word/label dicts
idx2w = {w2idx[k]:k for k in w2idx}
idx2la = {labels2idx[k]:k for k in labels2idx}

# For conlleval script
words_train = [ list(map(lambda x: idx2w[x], w)) for w in train_x]
labels_train = [ list(map(lambda x: idx2la[x], y)) for y in train_label]
words_val = [ list(map(lambda x: idx2w[x], w)) for w in val_x]
labels_val = [ list(map(lambda x: idx2la[x], y)) for y in val_label]

n_classes = len(idx2la)
n_vocab = len(idx2w)
```
### Keras Model
下面来定义模型，Keras有内置的Embedding层可用于word embeddings。
```python
from keras.models import Sequential
from keras.layers.embeddings import Embedding
from keras.layers.recurrent import SimpleRNN
from keras.layers.core import Dense, Dropout
from keras.layers.wrappers import TimeDistributed
from keras.layers import Convolution1D

model = Sequential()
model.add(Embedding(n_vocab,100))
model.add(Dropout(0.25))
model.add(SimpleRNN(100,return_sequences=True))
model.add(TimeDistributed(Dense(n_classes, activation='softmax')))
model.compile('rmsprop', 'categorical_crossentropy')
```

### training
下面开始训练模型，每批只使用一个句子。
由于每个句子的长度不同，这里不能使用model.fit()，我们使用model.train_on_batch()。
```python
import progressbar
n_epochs = 30

for i in range(n_epochs):
	print("Training epoch {}".format(i))

	bar = progressbar.ProgressBar(max_value=len(train_x))
	for n_batch, sent in bar(enumerate(train_x)):
	label = train_label[n_batch]
	# Make labels one hot
	label = np.eye(n_classes)[label][np.newaxis,:]
	# View each sentence as a batch
	sent = sent[np.newaxis,:]

	if sent.shape[1] > 1: #ignore 1 word sentences
	model.train_on_batch(sent, label)
```

### Evaluation
为了估量模型的准确率，我们使用model.predict_on_batch()和metrics.accuracy.conlleval()。
```python
from metrics.accuracy import conlleval

labels_pred_val = []

bar = progressbar.ProgressBar(max_value=len(val_x))
for n_batch, sent in bar(enumerate(val_x)):
	label = val_label[n_batch]
	label = np.eye(n_classes)[label][np.newaxis,:]
	sent = sent[np.newaxis,:]

	pred = model.predict_on_batch(sent)
	pred = np.argmax(pred,-1)[0]
	labels_pred_val.append(pred)

labels_pred_val = [ list(map(lambda x: idx2la[x], y)) \
	for y in labels_pred_val]
con_dict = conlleval(labels_pred_val, labels_val,
	words_val, 'measure.txt')

print('Precision = {}, Recall = {}, F1 = {}'.format(
	con_dict['r'], con_dict['p'], con_dict['f1']))
```
 该模型可以得到92.36的F1。

### Improvements
该模型的不足在于，预测时没有利用后续的词。  
可以想象，当前词的属性和下一个词也可能会有关联。  
为了捕获后续词的信息，可以在RNN和Embedding层之间加入卷基层。  
```python
model = Sequential()
model.add(Embedding(n_vocab,100))
model.add(Convolution1D(128, 5, border_mode='same', activation='relu'))
model.add(Dropout(0.25))
model.add(GRU(100,return_sequences=True))
model.add(TimeDistributed(Dense(n_classes, activation='softmax')))
model.compile('rmsprop', 'categorical_crossentropy')
```
使用该方法可以达到94.90的F1得分。  

## References
- Grégoire Mesnil, Xiaodong He, Li Deng and Yoshua Bengio. Investigation of Recurrent-Neural-Network Architectures and Learning Methods for Spoken Language Understanding. Interspeech, 2013. pdf
- Recurrent Neural Networks with Word Embeddings, [theano tutorial](http://deeplearning.net/tutorial/rnnslu.html)
