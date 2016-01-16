---
layout: post
title: 基于 Stanford NLP software 的中文文本预处理
---

#### 本文已被[cos.name](http://cos.name/)转载：[http://cos.name/2016/01/intro-to-chinese-nlp/](http://cos.name/2016/01/intro-to-chinese-nlp/)

作为一个处理自然语言的团队，我们在日常工作中要用到不同的工具来预处理中文文本，比如[Jieba](https://github.com/fxsjy/jieba)和[Stanford NLP software](http://nlp.stanford.edu/software/)。出于准确性和效率的考虑，我们选择了Stanford NLP software， 所以本文将介绍基于Stanford NLP software的中文文本预处理流程。

## 中文文本处理简要介绍

与拉丁语系的文本不同，中文并不使用空格作为词语间的分隔符。比如当我们说‘We love coding.’，这句英文使用了两个空格来分割三个英文词汇；如果用中文做同样的表述， 就是‘我们爱写代码。’，其中不包含任何空格。因而，处理中文数据时，我们需要进行分词，而这恰恰时中文自然语言处理的一大难点。

下文将介绍中文文本预处理的几个主要步骤：

1. 中文分词

2. 标注词性

3. 生成词向量

4. 生成中文依存语法树

## Stanford NLP software 简要介绍

Stanford NLP software 是一个较大的工具合集：包括 [Stanford POS tagger](nlp.stanford.edu/software/tagger.shtml) 等组件，也有一个包含所有组件的合集 [Stanford CoreNLP](stanfordnlp.github.io/CoreNLP/)。各个组件是由不同的开发者开发的，所以每一个工具都有自己的语法。当我们研究这些组件的文档时，遇到了不少问题。下文记录这些问题和相对应的对策，以免重蹈覆辙。

Stanford NLP 小组提供了一个简明的FAQ -- [Stanford Parser FAQ](http://nlp.stanford.edu/software/parser-faq.shtml)和一份详细的Java文档 -- [Stanford JavaNLP API Documentation](http://nlp.stanford.edu/nlp/javadoc/javanlp/overview-summary.html)。在这两份文档中，有几点格外重要：

> 尽管PSFG分词器小且快，Factored分词器更适用于中文，所以我们推荐使用后者。

> 中文分词器默认使用GB18030编码 (Penn Chinese Treebank的默认编码)。

> 使用 -encoding 选项可以指定编码，比如 UTF-8，Big-5 或者 GB18030。

## 中文预处理的主要步骤。

### 1. 中文分词

诚如上面所言，分词是中文自然语言处理的一大难题。[Stanford Word Segmenter](http://nlp.stanford.edu/software/segmenter.shtml) 是专门用来处理这一问题的工具。FAQ请参见[Stanford Segmenter FAQ](http://nlp.stanford.edu/software/segmenter-faq.shtml)。具体用法如下：

{% highlight bash %}

bash -x segment.sh ctb INPUT_FILE UTF-8 0

{% endhighlight %}

### 2. 中文词性标注

词性标注是中文处理的另一大难题。我们曾经使用过Jieba来解决这个问题，但效果不尽理想。Jieba是基于词典规则来标注词性的，所以任意一个词在Jieba里有且只有一个词性。如果一个词有一个以上的词性，那么它的标签就变成了一个集合。比如‘阅读’既可以表示动词，也可以理解为名词，Jieba就会把它标注成n，而不是根据具体语境来给出合适的v或n的标签。这样一来，标注的效果就大打折扣。幸好[Stanford POS Tagger](http://nlp.stanford.edu/software/tagger.shtml)提供了一个根据语境标注词性的方法。具体用法如下：

{% highlight bash %}

java -mx3000m -cp "./*" edu.stanford.nlp.tagger.maxent.MaxentTagger -model models/chinese-distsim.tagger -textFile INPUT_FILE

{% endhighlight %}

### 3. 生成词向量

深度学习是目前机器学习领域中最热门的一个分支。而生成一个优质的词向量是利用深度学习处理NLP问题的一个先决条件。除了Google的[Word2vec](https://code.google.com/p/word2vec/)， Stanford NLP小组提供了另外一个选项 -- [GLOVE](nlp.stanford.edu/projects/glove)。

使用Glove也比较简单，下载并解压之后，只要对里面的demo.sh脚本进行相应修改，然后执行这个脚本即可。

{% highlight bash %}

CORPUS=text8                                    # 设置输入文件路径
VOCAB_FILE=vocab.txt                            # 设置输入词汇路径
COOCCURRENCE_FILE=cooccurrence.bin              
COOCCURRENCE_SHUF_FILE=cooccurrence.shuf.bin
BUILDDIR=build
SAVE_FILE=vectors                               # 设置输入文件路径
VERBOSE=2           
MEMORY=4.0                                      # 设置内存大小
VOCAB_MIN_COUNT=5                               # 设置词汇的最小频率
VECTOR_SIZE=50                                  # 设置矩阵维度
MAX_ITER=15                                     # 设置迭代次数
WINDOW_SIZE=15                                  # 设置词向量的窗口大小
BINARY=2
NUM_THREADS=8
X_MAX=10

{% endhighlight %}

### 4. 生成中文依存语法树

文本处理有时需要比词性更丰富的信息，比如句法信息，Stanford NLP小组提供了两篇论文： [The Stanford Parser: A statistical parser](http://nlp.stanford.edu/software/lex-parser.shtml) 和 [Neural Network Dependency Parser](http://nlp.stanford.edu/software/nndep.shtml)，并在这两篇论文的基础上开发了两个工具，可惜效果都不太理想。前者的处理格式是正确的中文依存语法格式，但是速度极慢（差不多一秒一句）；而后者虽然处理速度较快，但生成的格式和论文[Discriminative reordering with Chinese grammatical relations features – acepor](http://www.aclweb.org/anthology/W09-2307)中的完全不一样。我们尝试了邮件联系论文作者和工具作者，并且在[Stackoverflow](https://stackoverflow.com/questions/33294148/how-to-use-nndep-parser-in-stanford-parser-to-process-chinese-data)上提问，但这个问题似乎无解。

尽管如此，我们还是把两个方案都记录在此：

{% highlight bash %}

java -cp "*:." -Xmx4g edu.stanford.nlp.pipeline.StanfordCoreNLP -file INPUT_FILE -props StanfordCoreNLP-chinese.properties -outputFormat text -parse.originalDependencies

java -cp "./*" edu.stanford.nlp.parser.nndep.DependencyParser -props nndep.props -textFile INPUT_FILE -outFile OUTPUT_FILE

{% endhighlight %}

## 结论

预处理中文文本并非易事，Stanford NLP小组对此作出了极大的贡献。我们的工作因而受益良多，所以我们非常感谢他们的努力。当然我们也期待 Stanford NLP software 能更上一层楼。
