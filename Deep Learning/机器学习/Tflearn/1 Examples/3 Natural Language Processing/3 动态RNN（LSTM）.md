应用动态LSTM从IMDB数据集中分类变长文本。
[toc]

# 1、TFlearn
https://github.com/tflearn/tflearn/blob/master/examples/nlp/dynamic_lstm.py
```python
# -*- coding: utf-8 -*-
"""
Simple example using a Dynamic RNN (LSTM) to classify IMDB sentiment dataset.
Dynamic computation are performed over sequences with variable length.
References:
    - Long Short Term Memory, Sepp Hochreiter & Jurgen Schmidhuber, Neural
    Computation 9(8): 1735-1780, 1997.
    - Andrew L. Maas, Raymond E. Daly, Peter T. Pham, Dan Huang, Andrew Y. Ng,
    and Christopher Potts. (2011). Learning Word Vectors for Sentiment
    Analysis. The 49th Annual Meeting of the Association for Computational
    Linguistics (ACL 2011).
Links:
    - http://deeplearning.cs.cmu.edu/pdfs/Hochreiter97_lstm.pdf
    - http://ai.stanford.edu/~amaas/data/sentiment/
"""
from __future__ import division, print_function, absolute_import

import tflearn
from tflearn.data_utils import to_categorical, pad_sequences
from tflearn.datasets import imdb

n_class=2
hiddle_layes=128
hiddle_layes_2=128
time_seqs=1
input_length_each_seq=100

n_datas=10000

# IMDB Dataset loading
train, test, _ = imdb.load_data(path='imdb.pkl', n_words=n_datas,
                                valid_portion=0.1)
trainX, trainY = train
testX, testY = test

# Data preprocessing
# NOTE: Padding is required for dimension consistency. This will pad sequences
# with 0 at the end, until it reaches the max sequence length. 0 is used as a
# masking value by dynamic RNNs in TFLearn; a sequence length will be
# retrieved by counting non zero elements in a sequence. Then dynamic RNN step
# computation is performed according to that length.
trainX = pad_sequences(trainX, maxlen=input_length_each_seq, value=0.)
testX = pad_sequences(testX, maxlen=input_length_each_seq, value=0.)
# Converting labels to binary vectors
trainY = to_categorical(trainY,n_class)
testY = to_categorical(testY,n_class)

# Network building
net = tflearn.input_data([None, input_length_each_seq])
# Masking is not required for embedding, sequence length is computed prior to
# the embedding op and assigned as 'seq_length' attribute to the returned Tensor.
net = tflearn.embedding(net, input_dim=n_datas, output_dim=hiddle_layes)
net = tflearn.lstm(net, hiddle_layes_2, dropout=0.8, dynamic=True)
net = tflearn.fully_connected(net, n_class, activation='softmax')
net = tflearn.regression(net, optimizer='adam', learning_rate=0.001,
                         loss='categorical_crossentropy')

# Training
model = tflearn.DNN(net, tensorboard_verbose=0)
model.fit(trainX, trainY,n_epoch=1, validation_set=(testX, testY), show_metric=True,
          batch_size=32)
```
# 2、tensorflow
https://github.com/aymericdamien/TensorFlow-Examples/blob/master/examples/3_NeuralNetworks/dynamic_rnn.py