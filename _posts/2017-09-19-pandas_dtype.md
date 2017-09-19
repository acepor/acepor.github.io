---
layout: post
title: Why shall we always set dtype when we use Pandas?
tags: [Python, Pandas, Data Structure]
---

Recently, we were planning to use a more compact and convenient file format to store our data on cloud, and when we tested `pandas` with parquet, we encountered a strange and difficult bug.

We tried to use `fastparquet` lib to output a `pandas.DataFrame` as a parquet file to disk.

{% highlight python %}

import fastparquet as fp
fp.write(OUTPUT_DIR, df)

{% endhighlight %}

However, the error always occurred as

> TypeError: bad argument type for built-in operation

We tried to set different options in this `write` function; however, none of them fixed the error.

{% highlight python %}

fp.write(OUTPUT_DIR, df, object_encoding='utf8')

# or

fp.write(OUTPUT_DIR, df, has_nulls=True)

{% endhighlight %}

Then we realised a warning message when we use `pandas.read_csv` to load data into memory:

> /usr/local/lib/python3.6/site-packages/memory_profiler.py:996: DtypeWarning: Columns (1) have mixed types. Specify dtype option on import or set low_memory=False.
  include_children=include_children)

OK, here is our code for loading data with `pandas.read_csv`, and we haven't specified other options explicitly.

{% highlight python %}

df = pd.read_csv(INPUT_DIR, engine='c')

{% endhighlight %}

By default, `low_memory` is `True`, so `pandas` will follow the following procedure:

> Internally process the file in chunks, resulting in lower memory use
    while parsing, but possibly mixed type inference.

Aha, `pandas` will guess the data type chunk by chunk if `low_memory` is `True`, so each item in a single column might have different types. However, the fix is pretty easy: stating `dtype` explicitly.

{% highlight python %}

%time %memit df = pd.read_csv(INPUT_DIR, engine='c', dtype={'FULL': 'str', 'COUNT': 'int'}, header=1)

fp.write(OUTPUT_DIR, df)

{% endhighlight %}

Everything can be perfectly outputted. Therefore, to prevent such annoying bugs, we shall always state `dtype` explicitly.
