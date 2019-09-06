---
layout: post
title: Skiprow in Pandas read_csv
tags: [Python, pandas]
---

`pandas` has a powerful tool `read_csv` for reading csv files, for example, we introduced that we can use this tool to read a large csv file [chunk by chunk](http://acepor.github.io/2017/08/03/using-chunksize/). 

There is an option in `read_csv` callsed `skiprows`, which can skip certain rows. In practice, if our dataset has some dirty records, we may use this option to abandon the dirty data, which seems useful. However, this option does not work as I espected.

According to the documentary, `read_csv` can accept the following input: list-like, int or callable, optional. These three inputs will have three completely different outputs:

> Line numbers to skip (0-indexed) or number of lines to skip (int) at the start of the file.
If callable, the callable function will be evaluated against the row indices, returning True if the row should be skipped and False otherwise. An example of a valid callable argument would be lambda x: x in [0, 2].

Yes, if you input a list of int, it will skip these certain rows.

If you give an int, it will skip from the start to that line.

If you give a lambda function, it will evaluate each row according to the function.

OK, if you don't think it's complicated, then there is another trap.

When you use `read_csv` to read a dataset withe dirty records, you will encounter an error like this:

{% highlight python %}

ParserError: Error tokenizing data. C error: Expected 18 fields in line 2443440, saw 19

{% endhighlight %}

If you put this number as an input to `skiprows`, congratulations, you will get the same error again.

{% highlight python %}

ParserError: Error tokenizing data. C error: Expected 18 fields in line 2443440, saw 19

{% endhighlight %}

Actully, `skiprows` is indexed from 0, so you need put the row number n-1 as the input:

{% highlight python %}

> df = pd.read_csv(data_file, sep='\t', skiprows=[2443439]

{% endhighlight %}

Only in this way will `pandas` skips the exact row as you expected.

