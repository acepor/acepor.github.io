---
layout: post
title: A Brief Introduction to spaCy
tags: [NLP, data\_processing, algorithm]
---
In our recent projects, we gave a try of `spaCy`, an under-developing Python NLP library. According to the author:

> `spaCy` excels at large-scale information extraction tasks. It's written from the ground up in carefully memory-managed `Cython`. Independent research has confirmed that `spaCy` is the fastest in the world. If your application needs to process entire web dumps, spaCy is the library you want to be using.

Considering its speed and performance, we used it in some low-level NLP tasks, such as chunking and POS-tagging. Also, we tried `spaCy` in the NER task, but, because of being a black-box, we did not use it in our production.

>  The exact algorithm is a pastiche of well-known methods, and is not currently described in any single publication.

## Using spaCy for low-level NLP tasks

`spaCy`’s official site provides [a comprehensive guide](https://spacy.io/docs/usage/) to use its workflows, but in reality, we need a more flexible and robust way to use these workflows. Therefore, we can wrap them together for low-level NLP tasks.

{% highlight python %}

def spacy_parser(text, switches, label):  
    """  
    :param text: a sentence or a doc  
    :param switch: a list of switches: chk for sentence chunking, pos for pos tagging, and ner for NER,  
    crf for crf pre-processing  
    :param label: filtering the NER labels  
    :return:  
    """  
    spacy_result = NLP(text)  
    spacy_dic = {'chk': [i.text for i in spacy_result.sents],  
                 'txt': (i.text for i in spacy_result),  
                 'pos': (i.pos_ for i in spacy_result),  
                 'dep': (i.dep_ for i in spacy_result)  
                 }  
    result = {'pos': spacy_dic['pos'],  
              'chk': spacy_dic['chk'],  
              'crf': (i + ('O',) for i in zip(spacy_dic['txt'], spacy_dic['pos']),  
              'dep': (i for i in zip(spacy_dic['txt'], spacy_dic['dep']),  
              }  
    return list(result[switches]) 

{% endhighlight %}
	
In this function, we use two dictionaries to compile all choices together. 

In the first dictionary `spacy_dic`, we read specific result out from the `spaCy` processing result `spacy_result` object. Each result is wrapped by a comprehension, either a list comprehension or a generator comprehension. Respectively, `spacy_dic['chk']` returns the sentence chunks, `spacy_dic['txt']` returns the original text, `spacy_dic['pos']` returns part-of-speech tagging result, and `spacy_dic['dep']` returns dependency parsing result.

[spaCy token API page](https://spacy.io/docs/api/token) provides all avaiable APIs, so we can add any other avaiable APIs to this dictionary.

{% highlight python %}
  
spacy_dic = {'chk': [i.text for i in spacy_result.sents],  
            'txt': (i.text for i in spacy_result),  
            'pos': (i.pos_ for i in spacy_result),  
            'dep': (i.dep_ for i in spacy_result)  
            }

{% endhighlight %}

We then use the second dictionary `result` to assemble the result from the first dictionary `spacy_dic`, in order to avoid unnecessary repeated computing.

{% highlight python %}
  
result = {'pos': spacy_dic['pos'],  
        'chk': spacy_dic['chk'],  
        'crf': (i + ('O',) for i in zip(spacy_dic['txt'], spacy_dic['pos']),  
        'dep': (i for i in zip(spacy_dic['txt'], spacy_dic['dep']),  
        }  

{% endhighlight %}

The last step is to use `dict`'s default key-get method to achieve the result from above two dictionaries.

{% highlight python %}

return list(result[switches]) 

{% endhighlight %}

## Using dictionary to handle multi-choice logic

Using a dictionary structure is an elegent way to handle a complicated `if..elif..else` logic. 

{% highlight python %}

spacy_dic = {'chk': [i.text for i in spacy_result.sents],  
            'txt': (i.text for i in spacy_result),  
            'pos': (i.pos_ for i in spacy_result),  
            'dep': (i.dep_ for i in spacy_result)  
            }
spacy_dic[switches]

{% endhighlight %}

If we do not use a dictionary strucutre here, we may use a tradiaional way to deal with this logic like this:

{% highlight python %}

if switches == 'chk':
    result1 = [i.text for i in spacy_result.sents], 
elif switches == 'txt': 
    result1 = (i.text for i in spacy_result),  
elif switches == 'pos': 
    result1 = (i.pos_ for i in spacy_result),  
elif switches == 'dep': 
    result1 = (i.dep_ for i in spacy_result)  

{% endhighlight %}

Which one do you like? Definitely I would choose the first one.

## Final thought

`spaCy` indeed provides an efficient and extensive library for NLP tasks. If you have not given a try, it is worth taking a look at it.