---
layout: post
title: Using CRF in Python
tags: [NLP, data_processing, algorithm]
---

CRF (Conditional Random Fields) has been a popular supervised learning method before deep learning occurred, and still, it is a easy-to-use and robust machine learning algorithm. We recently used this algorithm to do NER (name entity   recognition), and here is a brief summary of using CRF in Python.

## Introduction of CRF
Edwin Chen has written a concise but very helpful introduction of CRF: [Introduction to Conditional Random Fields][1], so we are not going to repeat this topic.

## Popular CRF libraries
Among CRF toolkits, [CRF++][2] and [CRFsuite][3] are the most popular choices. However, CRFsuite is more robust and faster-to-train. We were told that CRF++ needs the features to be set in files, but CRFsuite can calculate features in the training. Therefore, we chose CRFsuite as the framework.

Several Python libraries provide support to CRFsuite, including [python-crfsuite][4] and [sklearn-crfsuite][5]. We chose the later one due to its comprehensive tutorial.

## Using sklearn-crfsuite
The `sklearn-crfsuite`’s tutorial can be found at [github][6]. It is easy to follow; nevertheless, the code quality cannot match production code quality, so we made a number of modifications.

### Feature format
Based on [python-crfsuite][7], [sklearn-crfsuite][8] also uses dictionary as the default feature format. 

{% highlight python %}

 {'+1:postag': 'Fpa',
'+1:postag[:2]': 'Fp',
'+1:word.istitle()': False,
 '+1:word.isupper()': False,
 '+1:word.lower()': '(',
 'BOS': True,
 'bias': 1.0,
 'postag': 'NP',
 'postag[:2]': 'NP',
 'word.isdigit()': False,
 'word.istitle()': True,
 'word.isupper()': False,
 'word.lower()': 'melbourne',
 ‘word[-2:]': 'ne'}

{% endhighlight %}

Note, it does not support `pandas` DataFrame format as feature format.

In the sklearn-crfsuite, it puts all features in a function, which is very difficult to config in the test environment. 

{% highlight python %}
def word2features(sent, i):
    word = sent[i][0]
    postag = sent[i][1]

    features = {
    'bias': 1.0,
    'word.lower()': word.lower(),
    'word[-3:]': word[-3:],
    'word[-2:]': word[-2:],
    'word.isupper()': word.isupper(),
    'word.istitle()': word.istitle(),
    'word.isdigit()': word.isdigit(),
    'postag': postag,
    'postag[:2]': postag[:2],  
    }
    
    if i > 0:
    word1 = sent[i-1][0]
    postag1 = sent[i-1][1]
    features.update({
    '-1:word.lower()': word1.lower(),
    '-1:word.istitle()': word1.istitle(),
    '-1:word.isupper()': word1.isupper(),
    '-1:postag': postag1,
    '-1:postag[:2]': postag1[:2],
    })
    else:
    features['BOS'] = True

    if i < len(sent)-1:
    word1 = sent[i+1][0]
    postag1 = sent[i+1][1]
    features.update({
    '+1:word.lower()': word1.lower(),
    '+1:word.istitle()': word1.istitle(),
    '+1:word.isupper()': word1.isupper(),
    '+1:postag': postag1,
    '+1:postag[:2]': postag1[:2],
    })
    else:
    features['EOS'] = True

return features
{% endhighlight %}

To fix this problem, we split it into several individual functions. 
1. We use function `load_yaml_conf` to read the feature configuration from a yaml file;
2. we use function `feature_selector` to convert the configuration to the feature dictionary. 

{% highlight python %}
def load_yaml_conf(conf_f):  
    with open(conf_f, 'r') as f:  
        result = load(f)  
    return result

def feature_selector(word, feature_conf, conf_switch, postag):  
    feature_dict = {  
        'bias': 1.0,  
        conf_switch + '_word.lower()': word.lower(),  
        conf_switch + '_word[-3]': word[-3:],  
        conf_switch + '_word[-2]': word[-2:],  
        conf_switch + '_word.isupper()': word.isupper(),  
        conf_switch + '_word.istitle()': word.istitle(),  
        conf_switch + '_word.isdigit()': word.isdigit(),  
        conf_switch + '_word.islower()': word.islower(),  
        conf_switch + '_postag': postag,  
    }  
    return {i: feature_dict.get(i) for i in feature_conf[conf_switch] if i in feature_dict.keys()}
{% endhighlight %}

Here is a sample yaml configuration file. ‘current’ and ‘previous’ are conf_switches.
{% highlight yaml %}
current:  
  - bias  
  - current_word.lower()  
  - current_word[-3]  
  - current_word[-2]  
  - current_word.isupper()  
  - current_word.istitle()  
  - current_word.isdigit()  
  - current_word.islower()  
  - current_postag  
previous:  
  - previous_word.lower()  
  - previous_word.istitle()  
  - previous_word.isupper()  
  - previous_postag
{% endhighlight %}

Then we use another function to calculate the current token its neighbour’s features, which are the most important parts of CRF:

{% highlight python %}
def word2features(sent, i, feature_conf):  
    word, postag, _, = sent[i]  
    features = feature_selector(word, feature_conf, 'current', postag)  
    if i > 0:  
        word1, postag1, _,  = sent[i - 1]  
        features.update(  
            feature_selector(word1, feature_conf, 'previous', postag1))  
    else:  
        features['BOS'] = True  
  
    if i < len(sent) - 1:  
        word1, postag1, _, = sent[i + 1]  
        features.update(  
            feature_selector(word1, feature_conf, 'next', postag1))  
    else:  
        features['EOS'] = True  
    return features
{% endhighlight %}

In this way, users can easily change the feature set in the configuration without changing the script.

### Adding extra features

One trick to boost the performance of CRF is to add extra feature dictionaries. 
Say, if we want to label POS tags by using CRF, we can add an noun suffix dictionary, for example, ‘tion’ is a typical noun suffix. Therefore, we use several  functions to add features from external dictionaries. In this way, we can add features in the calculation instead of inputing all features from files. This is the most noticeable difference between CRFsuite and CRF++.

Function `add_one_features_list` adds features from a list file, and function `add_one_features_dict` adds features from a key-value-pair file.

 {% highlight python %}

def add_one_features_list(sent, feature_set):  
    feature_list = ['1' if line[0] in feature_set else '0' for line in sent]  
    return [(sent[i] + (feature_list[i],)) for i in range(len(list(sent)))]  
  
  
def add_one_feature_dict(sent, feature_dic):  
    feature_list = [str(feature_dic.get(line[0])) if line[0] in feature_dic.keys() else '0' for line in sent]  
    return [(sent[i] + (feature_list[i],)) for i in range(len(list(sent)))]  
{% endhighlight %}

Please notice, both above functions use a special case of list/dict comprehension. Usually, we can only put `if` in a list/dict comprehension, but here, we add `if..., else...` condition in them. The order of this syntax is different from a comprehension only with an `if` condiction.

 {% highlight python %}
['1' if line[0] in feature_set else '0' for line in sent]
{% endhighlight %}

With these two function, we can easily add multiple external features at once.

### Feeding data to CRF trainer
The next step is to feed text data with added features to the CRF trainer. Because each token is converted to dictionary, and each sentence is converted to a list, so a piece of text is therefore converted to a nested list with nested lists of dictionaries. In this case, we need to convert them respectively.

The first function below extracts features of each token, while the second one extracts labels of each token.

 {% highlight python %}
def sent2features(line, feature_conf):  
    return [word2features(line, i, feature]_conf) for i in range(len(line))]  
  
def sent2labels(line):  
    return [i[2] for i in line]  # Use the right column  
{% endhighlight %}

### Setting parameters of CRF algorithm
CRF is an umbrella term for a family of algorithms. For the NER task, which is basically a sequence prediction task, the chain CRF is more suitable. Therefore we need set the specific CRF algorithm in CRFsuite. Here, we choose lbfgs CRF (Limited-memory Broyden-Fletcher-Goldfarb-Shanno), and `sklearn-crfsuite` will take care of the rest.

 {% highlight python %}
def train_crf(X_train, y_train):  
    crf = sklearn_crfsuite.CRF(  
        algorithm='lbfgs',  
        c1=0.1,  
        c2=0.1,  
        max_iterations=100,  
        all_possible_transitions=True  
    )  
    return crf.fit(X_train, y_train)
{% endhighlight %}

### Testing CRF result
To calculate F1 score of the CRF training, we can use function `metrics.flat_f1_score` from `sklearn`.
 {% highlight python %}
def test_crf_prediction(crf, X_test, y_test):  
    labels = show_crf_label(crf)  
    y_pred = crf.predict(X_test)  
    result = metrics.flat_f1_score(y_test, y_pred, average='weighted', labels=labels)  
    details = metrics.flat_classification_report(y_test, y_pred, digits=3, labels=labels)  
    result = metrics.flat_f1_score(y_test_converted, y_pred_converted, average='weighted', labels=['1'])  
  
    details = [i for i in [findall(RE_WORDS, i) for i in details.split('\n')] if i != []][1:-1]  
    details = pd.DataFrame(details, columns=HEADER_CRF)  
    return result, details
{% endhighlight %}

## Final thought
`sklearn-crfsuite` is a very easy-to-use package for applying CRF algorithms, and this post summarizes some key steps of using it in a NER task.

[1]:	http://blog.echen.me/2012/01/03/introduction-to-conditional-random-fields/
[2]:	https://taku910.github.io/crfpp/
[3]:	http://www.chokkan.org/software/crfsuite/
[4]:	https://github.com/scrapinghub/python-crfsuite
[5]:	https://github.com/TeamHG-Memex/sklearn-crfsuite
[6]:	https://github.com/TeamHG-Memex/sklearn-crfsuite/blob/master/docs/CoNLL2002.ipynb
[7]:	https://github.com/scrapinghub/python-crfsuite
[8]:	https://github.com/TeamHG-Memex/sklearn-crfsuite
