---
layout: post
title: General Pipelines for Chinese NLP Engineering with Stanford NLP software
---

With a focus on processing Chinese textual data, we have been using different tools extensively, namely [Jieba](https://github.com/fxsjy/jieba) and [Stanford NLP software](http://nlp.stanford.edu/software/). Either of them has its own advantages and drawbacks. To balance efficiency and accuracy, we eventually chose Stanford NLP software as our default toolset. Therefore, this post introduces some major procedures of processing Chinese textual data with the help of Stanford NLP software.

## A Brief Introduction of NLP on Chinese data

Different from Latin languages, Chinese does not use any space as a delimiter or word boundary. For example, when we say 'We love co ding.' in English, there are three words with two spaces as delimiters, but in Chinese, we express the same idea as '我爱写代码。': four words with no space. In this sense, if we want to process Chinese data, we have to parse it first, which appears to be the major obstacle in NLP on Chinese data.

The following introduces the general procedures in Chinese processing:

1. Converting traditional Chinese data to simplified Chinese, or vice versa.

2. Parsing Chinese data into words

3. POS-tagging segmented Chinese data

4. Generating word-embedding by GLOVE

5. Generating dependency syntax trees

## A Brief Introduction of Stanford NLP software

Stanford NLP software is a huge toolset: it has a number of individual components, such as [Stanford POS tagger](nlp.stanford.edu/software/tagger.shtml), and a combination of all these components - [Stanford CoreNLP](stanfordnlp.github.io/CoreNLP/). Due to its size, each tool has its own command. Hence, each document should be carefully read and studied. We have encountered numerous problems, so this note attempts to jot down the key steps in dealing those problems.

Stanford NLP groups offers a brief FAQ for this toolset -- [Stanford Parser FAQ](http://nlp.stanford.edu/software/parser-faq.shtml), and a detailed Java document [Stanford JavaNLP API Documentation](http://nlp.stanford.edu/nlp/javadoc/javanlp/overview-summary.html).

As stated on the above pages, especially these three points should be borne in mind:

> The PCFG parsers are smaller and faster. But the Factored parser is significantly better for Chinese, and we would generally recommend its use.

> By default, the Chinese parser uses GB18030 (the native character encoding of the Penn Chinese Treebank) for input and output.

> With the flag -encoding encoding to the parser, where encoding is a character set encoding name recognised within Java, such as: UTF-8, Big5-HKSCS, or GB18030, the parser can process data in the needed format.

## Major procedures of Chinese NLP engineering

### 1. Converting traditional Chinese data to simplified Chinese

Often, we have a dataset with mixed Chinese characters: the simplified Chinese used in mainland China, and the traditional Chin used in other areas. It is not a good idea to ignore the mixed usage, because it will bring further problems in the later processing. To overcome this, we use [opencc](https://github.com/BYVoid/OpenCC) by [BYVoid](opencc.byvoid.com/). It is relatively easy to use as it automatically detects the type of Chinese used in the data.

{% highlight bash %}

opencc -i INPUT_FILE -o OUTPUT_FILE

{% endhighlight %}

### 2. Parsing Chinese data into words

As mentioned above, parsing Chinese is a major obstacle in processing. Stanford NLP software provides [Stanford Word Segmenter](http://nlp.stanford.edu/software/segmenter.shtml) to tackle this problem. See [Stanford Segmenter FAQ](http://nlp.stanford.edu/software/segmenter-faq.shtml).

{% highlight bash %}

bash -x segment.sh ctb INPUT_FILE UTF-8 0

{% endhighlight %}

### 3. POS-tagging segmented Chinese data

This is another major difficulty in processing. We used to deploy Jieba to deal with this problem. However, Jieba uses a dictioanry-based method to annotate POS tags, and each word has only one POS tag, which is less reasonable to a linguist. Imagine that 'book' in English has multiple senses: as a noun when referring an object to be read, or as a verb when referring the action of making an appointment. Nevertheless, if a POS parser assigns only one tag to the word 'book' regardless its sense, it does not make any sense. Luckily, [Stanford POS Tagger](http://nlp.stanford.edu/software/tagger.shtml) applies a different approach to handle this problem.

{% highlight bash %}

java -mx300m -cp "./*" edu.stanford.nlp.tagger.maxent.MaxentTagger -model models/chinese-distsim.tagger -textFile INPUT_FILE

{% endhighlight %}

### 4. Generating word-embedding by GLOVE

Deep learning is currently the hottest topic in NLP and machine learning. The key step is to generate a good word embedding. Stanford NLP group also provides a solution [GLOVE](nlp.stanford.edu/projects/glove), apart from Google's [Word2vec](https://code.google.com/p/word2vec/).

The demo.sh in the GLVOE directory is a great example, so we can modify its parameters according to our needs.

{% highlight bash %}

CORPUS=text8                                    # set input file path
VOCAB_FILE=vocab.txt                            # set output vocabulary file path
COOCCURRENCE_FILE=cooccurrence.bin              
COOCCURRENCE_SHUF_FILE=cooccurrence.shuf.bin
BUILDDIR=build
SAVE_FILE=vectors                               # set output vector file path
VERBOSE=2           
MEMORY=4.0                                      # set the minimum memory size
VOCAB_MIN_COUNT=5                               # set the minimum word count
VECTOR_SIZE=50                                  # set the column size of the vector
MAX_ITER=15                                     # set the iteration time
WINDOW_SIZE=15                                  # set the window size of embedding
BINARY=2
NUM_THREADS=8
X_MAX=10

{% endhighlight %}

### 5. Generating dependency syntax trees

To understand sentences, POS-tags are far from enough, so we need more information, such as relationships between words. Stanford NLP group provides two supposed-to-be-good solutions: [The Stanford Parser: A statistical parser](http://nlp.stanford.edu/software/lex-parser.shtml), and [Neural Network Dependency Parser](http://nlp.stanford.edu/software/nndep.shtml). The first one provides the right dependency format, but the speed is terribly slow (one sentence per second), while the second one is speedy, but the dependency format is totally different from what they described in their paper [Discriminative reordering with Chinese grammatical relations features – acepor](http://www.aclweb.org/anthology/W09-2307). We emailed main authors of two parsers and several papers, and posed a request on [Stackoverflow](https://stackoverflow.com/questions/33294148/how-to-use-nndep-parser-in-stanford-parser-to-process-chinese-data), but it seems that no one can provide a workable solution to solve this 'tricky' dilemma.

Anyway, we show two commands here.

Option 1: right format, but slower speed

{% highlight bash %}

java -Xmx12g -cp "*:." -Xmx4g edu.stanford.nlp.pipeline.StanfordCoreNLP -file INPUT_FILE -props StanfordCoreNLP-chinese.properties -outputFormat text -parse.originalDependencies

{% endhighlight %}

Option2: wrong format, but faster speed

{% highlight bash %}

java -cp "./*" edu.stanford.nlp.parser.nndep.DependencyParser -model edu/stanford/nlp/models/parser/nndep/CTB_CoNLL_params.txt.gz -tagger.model edu/stanford/nlp/models/pos-tagger/chinese-distsim/chinese-distsim.tagger -sentenceDelimiter "。" -escaper edu.stanford.nlp.trees.international.pennchinese.ChineseEscaper -textFile INPUT_FILE -outFile OUTPUT_FILE

java -cp "./*" edu.stanford.nlp.parser.nndep.DependencyParser -props nndep.props -textFile $i -outFile output/`basename $i`.parsed

{% endhighlight %}


## Conclusion

Processing Chinese textual data is not an easy task, and Stanford NLP group has done a good job on this area. We have benefited tremendously from their work, and we really appreciate that. We hope the quality of these tools can be improved constantly in the future.
