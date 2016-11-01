---
layout: post
title: Tips of Pandas
tags: [python, pandas]
---

### View the header of a pd table

{% highlight python %}

data = pd.read_csv(FILE)
list(data.head())

{% endhighlight %}


### Convert a pd Series to df

{% highlight python %}

pd.Series.to_frame()

{% endhighlight %}

### Extract data

{% highlight python %}

df[df[column].isin()]

{% endhighlight %}

### Replace strings in a given column

{% highlight python %}

df.loc[:, column].str.replace()

{% endhighlight %}


### Merge df

{% highlight python %}

pd.concat([df1, df2], axis=1)

{% endhighlight %}


### Read CSV into pandas

Although Pandas provides a convenient way for users to import CSV data, sometimes it could be tricky to use this method.

Bt default, this method use Python code to interpret CSV data; however, this solution is at least 50% slower than C implementation. The difference is that the Python code is more robust than C code in dealing with the separator. If the separator is a comma, then both methods can be used, otherwise, only the python code is applicable. Therefore, for the matter of speed, please use comma as the separator whenever possible.

{% highlight python %}

def csv2pd(in_file, column_constant, column_names, sep=",", quote=3, engine='c'):
    """
    use Function get_column_index to get a list of column indices
    :param in_file: a csv_file
    :param column_names: [column_names]
    :param column_constant: a predefined column header
    :param sep , by default
    :param quote: exlcude quotation marks
    :param engine: set import engine here
    :return: a trimmed pandas table
    """
    column_indices = get_column_index(column=column_constant, col_list=column_names)
    return pd.read_csv(in_file, usecols=column_indices, sep=sep, quoting=quote, engine=engine)

{% endhighlight %}

###

{% highlight python %}


{% endhighlight %}

###

{% highlight python %}


{% endhighlight %}

###

{% highlight python %}


{% endhighlight %}

###

{% highlight python %}


{% endhighlight %}

###

{% highlight python %}


{% endhighlight %}

###

{% highlight python %}


{% endhighlight %}

###

{% highlight python %}


{% endhighlight %}
