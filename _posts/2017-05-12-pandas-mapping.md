---
layout: post
title: Mapping Pandas Series with a Dict or a List
tags: [Python, pandas]
---

We often need map a Pandas Series with another dictionary, list or series.

By default, Pandas has a function called [`pandas.Series.map`](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.Series.map.html). With this function, it is very easy to map values of a dictionary to a Pandas Series according to the keys. However, this function does not work with lists as it only accepts function, dict or Series as input.

In this case, we need find another solution. Yes, Pandas provides [`pandas.Series.str.contains`](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.Series.str.contains.html) and [`pandas.Series.str.match`](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.Series.str.match.html#pandas.Series.str.match). Both of them use regular expression to do the matching. It is fine to do the mapping when the list only contains a small number of items. However, it becomes extremely slow to map with a huge list.

To bypass this problem, the best solution would be converting a list to a dict and then using `pandas.Series.map`:

{% highlight python %}

def map_list_to_series(df, col, new_col, lst):
    dic = {i: i for i in lst}
    df[new_col] = df[col].map(dic)
    return df

{% endhighlight %}

This is as fast as mapping a dictionary to a Series, well, apparently.

Here is another usage case. We have a DF Series, we aim to do a conditional replacement. If the the condition is complicated, it is not easy to do such a replacement in pandas directly. Therefore, we can construct a dictionary to map the original and target patterns, then we use `map` to finish such job. This is way faster than using `pandas.Series.str.replace` or `pandas.Series.str.translate`.

{% highlight python %}

def merge_tags(df, col, tag):
    tags = df[col].unique()
    tag_dict = dict([(i, 'O') if not i.endswith(tag) else (i, i) for i in tags])
    df[col] = df[col].map(tag_dict)
    return df

{% endhighlight %}
