---
layout: post
title: Reading Files into Pandas
tags: [python, pandas]
---

Pandas is an awesome Python package for manipulating data. It provides various tools to import data. In our daily job, CSV and JSON files are the most common data format to deal with. This post will introduce some tricks of `pandas.read_csv` and `pandas.read_json`.

## `pandas.read_csv`

`pandas.read_csv` is a convenient tool in pandas to handle csv file importing; however, it is not very intuitive as it supposes to be. It can use either Python or C engine to parse data, and the C engine is much more efficient than the default Python engine.

According to the [doc](http://pandas.pydata.org/pandas-docs/stable/io.html#basic), `pandas.read_csv` contains four basic parameters: `filepath_or_buffer`, `sep`, `delimiter` and `delim_whitespace`. The most important parameter here is `sep`: if the `sep` is set to be a comma, pandas can use the C engine to increase the reading speed, otherwise, the `sep` will be regarded as a regular expression string, and pandas can only use the default Python engine to parse.

Reading a 471MB CSV file containing 5,023,382 lines of data into pandas, with the help of `C` engine, the time cost reduces more than 50%.

{% highlight python %}

%time dd = csv2pd(in_file, engine='c', column_names=get_header(in_file), column_constant=get_header(in_file), sep=',', )
CPU times: user 14.6 s, sys: 1.93 s, total: 16.5 s
Wall time: 17.3 s

%time dd = csv2pd(in_file, engine='python', column_names=get_header(in_file), column_constant=get_header(in_file), sep=',')
CPU times: user 32.2 s, sys: 2.77 s, total: 35 s
Wall time: 38.6 s

{% endhighlight %}

Another trick to increase reading speed and reduce memory use  is to read columns only needed. However, this is not very easy to do `pandas.read_csv`. The [doc](http://pandas.pydata.org/pandas-docs/stable/io.html#column-and-index-locations-and-names) `usecols`:

>Return a subset of the columns. All elements in this array must either be positional (i.e. integer indices into the document columns) or strings that correspond to column names provided either by the user in `names` or inferred from the document header row(s).

Put it simple, we need provide the index of the columns to select the needed columns. Suppose the CSV file has many columns, and we only have to pick few of them, counting the index is not a realistic way to do so. Therefore I wrote two functions to solve this problem: using `get_header` to fetch the header of a CSV file, and use `get_column_index` to fetch the index of needed columns.

{% highlight python %}

def get_header(in_file, sep=","):
    in_file = open(in_file)
    result = next(in_file).strip('\n\r').split(sep)
    in_file.close()
    return result

{% endhighlight %}

{% highlight python %}

def get_column_index(col_list, column=get_column_index()):
    """
    :param column: the COLUMN_CHAT constant
    :param col_list: define wanted columns here
    :return: a list of indices
    """
    col_index_dic = {v: k for (k, v) in enumerate(column)}
    return [int(col_index_dic[col]) for col in col_list]

{% endhighlight %}

Combining them together, I can get the index of needed columns, and pass it to `usecols` options in `pandas.read_csv` function. This simplifies the whole process.

{% highlight python %}

def csv2pd(in_file, column_constant, column_names, sep=",", quote=3, engine='python'):
    """
    user Function get_column_index to get a list of column indices
    :param in_file: a csv_file
    :param column_names: the full column names of the csv
    :param column_constant: a predefined column header
    :param sep: ',' by default
    :param quote: exlcude quotation marks
    :param engine: choose engine for reading data
    :return: a trimmed pandas table
    """
    column_indices = get_column_index(column=column_constant, col_list=column_names)
    return pd.read_csv(in_file, usecols=column_indices, sep=sep, quoting=quote, engine=engine)

{% endhighlight %}



{% highlight python %}

def quickest_read_csv(in_file, column_names):
    """
    param: in_file: csv file
    """
    data = csv2pd(column_constant=get_header(in_file), column_names=column_names, engine='c',
                  in_file=in_file, quote=0, sep=',')
    return data

{% endhighlight %}

## `pandas.read_json`

{% highlight python %}

def json2pd(in_file, col_list, lines=True):
    """
    :param in_file: a json file
    :param col_list: set the extracted columns
    :param lines: True if the file is line-based
    :return: a pd df
    """
    data = pd.read_json(in_file, lines=lines)
    result = data[col_list]
    return result

{% endhighlight %}
