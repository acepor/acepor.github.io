---
layout: page
title: NER Resources
permalink: /ner/
---

NER is short for Name Entity Recognition, which is one of fundamental tasks in NLP and critical to other NLP tasks.

As machine learning develops, more and more new methods have been applied in this area. This resource book attempts to give a glance of these methods.

## Vanilla machine learning methods

### CRF

CRF is short for Conditional Random Fields.

__Toolkits__

[CRF++](https://taku910.github.io/crfpp/) is a simple, customizable, and open source implementation of Conditional Random Fields (CRFs) for segmenting/labeling sequential data.

[CRFsuite](http://www.chokkan.org/software/crfsuite/): A fast implementation of Conditional Random Fields (CRFs).

[python-crfsuite](https://github.com/scrapinghub/python-crfsuite) is a python binding to CRFsuite.

[sklearn-crfsuite](https://github.com/TeamHG-Memex/sklearn-crfsuite) is a thin CRFsuite (python-crfsuite) wrapper which provides interface similar to scikit-learn.

[CRF in Tensorflow](https://github.com/tensorflow/tensorflow/tree/master/tensorflow/contrib/crf) Linear-chain CRF layer.

__Papers__

Lafferty, J., McCallum, A., & Pereira, F. (2001). Conditional random fields: Probabilistic models for segmenting and labeling sequence data. [ACM](https://dl.acm.org/citation.cfm?id=655813).

Sutton, C., & McCallum, A. (2012). An Introduction to Conditional Random Fields. [Now Pub](http://doi.org/10.1561/2200000013).

Sha, F., & Pereira, F. (2003). Shallow parsing with conditional random fields. [ACLWeb](http://aclweb.org/anthology/N/N03/N03-1028.pdf).

__Other readings__

[Tutorial of using sklearn-crfsuite for NER task](https://sklearn-crfsuite.readthedocs.io/en/latest/tutorial.html)

### Learning2Search

Learning to Search is a nickname for Vowpal Wabbit.

__Toolkits__

Vowpal Wabbit on [Github](https://github.com/JohnLangford/vowpal_wabbit)

__Papers__

Chang, K.-W., He, H., Daumé, H., III, & Langford, J. (2015, March 19). Learning to Search for Dependencies. [arXiv.org](http://doi.org/10.1086/520098?ref=search-gateway:1fb8dfd0029c10fc8bd70957d58bb05e)

__Other readings__

[Named Entity Classification](https://blog.booking.com/named-entity-classification.html) by Themis Mavridis from booking.com


## Deep learning methods

### LSTM

__Toolkits__

[NeuroNER](https://github.com/Franck-Dernoncourt/NeuroNER#adding-a-new-dataset) is a program that performs named-entity recognition (NER).

__Papers__

Dernoncourt, F., Lee, J. Y., & Szolovits, P. (2017, May 16). NeuroNER: an easy-to-use program for named-entity recognition based on neural networks. [arXiv.org](https://arxiv.org/pdf/1705.05487.pdf)

__Other readings__


### LSTM with CRF

__Toolkits__

[Sequence Tagging with Tensorflow](https://github.com/guillaumegenthial/sequence_tagging)

__Papers__

Ma, X., & Hovy, E. (2016, March 4). End-to-end Sequence Labeling via Bi-directional LSTM-CNNs-CRF. [arXiv.org](https://arxiv.org/pdf/1603.01354.pdf).

__Other readings__

[Sequence Tagging with Tensorflow](https://guillaumegenthial.github.io/sequence-tagging-with-tensorflow.html)
