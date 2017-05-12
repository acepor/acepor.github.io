---
layout: post
title: Mapping Pandas Series with a Dict or a List
tags: [Python, logging, hack]
---

It is often that we need map a Pandas Series with another dictionary, list or series.

By default, Pandas has a function called [`pandas.Series.map`](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.Series.map.html). With this function, it is very easy to map values of a dictionary to a Pandas Series according to the keys. However, this function does not work with lists as it only accepts function, dict or Series as input.

In this case, we need find another solution. Yes, Pandas provides [`pandas.Series.str.contains`](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.Series.str.contains.html) and [`pandas.Series.str.match`] (http://pandas.pydata.org/pandas-docs/stable/generated/pandas.Series.str.match.html#pandas.Series.str.match). Both of them use regular expression to do the matching. It is fine to do the mapping when the list only contains a small number of items. However, it becomes extremely slow to map with a huge list.

To bypass this problem, the best solution would be converting a list to a dict and then using `pandas.Series.map`:

{% highlight python %}

def map_list_to_series(df, col, new_col, lst):
    dic = {i: i for i in lst}
    df[new_col] = df[col].map(lst)
    return df

{% endhighlight %}

This is as fast as mapping a dictionary to a Series, well, apparently.
