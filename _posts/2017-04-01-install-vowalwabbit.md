---
layout: post
title: Installing Vowpal Wapbbit with Python3
tags: [algorithm]
---


In a recent blog post [Named Entity Classification](https://blog.booking.com/named-entity-classification.html) by Themis Mavridis from [booking.com](booking.com), Learn2search (aka. Vowpal Wabbit) was stated as 'by far the best model' and 'by far the less demanding' of training resources in terms of doing NER.

As explained by the author, the major reason of these advantages is because of that

> L2S treats the problem as a sequential decision making process.

Therefore, we would like to give a try.

## Installation

Officially, learn2search is called Vowpal Wabbit. [The VW official site](https://github.com/JohnLangford/vowpal_wabbit) provides a detailed installation guidance.

On macOS, we can use `brew` to install VW:

{% highlight bash %}

brew install vowpal-wabbit

{% endhighlight %}

Some dependencies may be required if not installed beforehand:

{% highlight bash %}

brew install libtool
brew install autoconf
brew install automake
brew install boost  # This will take some time to install.
brew install boost-python  # This will take some time to install.

{% endhighlight %}

## Problem

However, this simple guidance did not work with my environment.

{% highlight bash %}

MacOS 10.12.3
Python 3.6.0
Python 3.5.2

{% endhighlight %}

Then, I used the manual installation approach, it still did not work with Python 3.5 or 3.6; however, strangely, when I switched to Python 2.7.13, it can be installed.

Soon, I posted an issue on [the official website](https://github.com/JohnLangford/vowpal_wabbit/issues/1208#issuecomment-290885557), and got the help from the authors.

## Solution

The solution is fairly simple: just changing the Makefile in the python directory of VW:

{% highlight bash %}

ifeq ($(VIRTUAL_ENV), )
  PYTHON_INCLUDE = $(shell python$(PYTHON_VERSION)-config --includes)
  PYTHON_LDFLAGS = $(shell python$(PYTHON_VERSION)-config --ldflags)
else
  PYTHON_INCLUDE = $(shell python-config --includes)
  PYTHON_LDFLAGS = $(shell python-config --ldflags)
endif

{% endhighlight %}

to ->

{% highlight bash %}

PYTHON_INCLUDE = $(shell python$(PYTHON_VERSION)-config --includes)
PYTHON_LDFLAGS = $(shell python$(PYTHON_VERSION)-config --ldflags)

{% endhighlight %}

Finally, VW can run on my machine.

## Final thoughts

We will give a shot of VW on our NER task, so stay tuned.
