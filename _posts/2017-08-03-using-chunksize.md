---
layout: post
title: Using Chunksize in Pandas
tags: [Python, pandas]
---

`pandas` is an efficient tool to process data, but when the dataset cannot be fit in memory, using `pandas` could be a little bit tricky.

Recently, we received a 10G+ dataset, and tried to use `pandas` to preprocess it and save it to a smaller CSV file. When we attempted to put all data into memory on our server (with 64G memory, but other colleagues were using more than half it), the memory was fully occupied by `pandas`, and the task was stuck there.

{% highlight python %}

def preprocess_patetnt(in_f, out_f):
    df = pd.read_table(in_f, sep='##')
    df.columns = ['id0', 'id1', 'ref']
    result = chunk[(df.ref.str.contains('^[a-zA-Z]+')) & (df.ref.str.len() > 80)]
    result.to_csv(out_f, index=False, header=False, mode='w')

{% endhighlight %}

Then, I remembered that `pandas` offers `chunksize` option in related functions, so we took another try, and succeeded.

{% highlight python %}

def preprocess_patetnt(in_f, out_f, size):
    reader = pd.read_table(in_f, sep='##', chunksize=size)
    for chunk in reader:
        chunk.columns = ['id0', 'id1', 'ref']
        result = chunk[(chunk.ref.str.contains('^[a-zA-Z]+')) & (chunk.ref.str.len() > 80)]
        result.to_csv(out_f, index=False, header=False, mode='a')

{% endhighlight %}

Some aspects are worth paying attetion to:

1. The `chunksize` should not be too small. If it is too small, the IO cost will be high to overcome the benefit. For example, if we have a file with one million lines, we did a little experiment:

| Chunksize | Memory (MiB) | Time (s) |
|-----------|--------------|----------|
| 100       | 142.13       | 36.9     |
| 1,000     | 141.38       | 13.8     |
| 10,000    | 141.38       | 12.1     |
| 100,000   | 209.88       | 12.7     |
| 200,000   | 312.15       | 12.5     |

In our main task, we set `chunksize` as 200,000, and it used 211.22MiB memory to process the 10G+ dataset with 9min 54s.

2. the `pandas.DataFrame.to_csv()` mode should be set as 'a' to append chunk results to a single file; otherwise, only the last chunk will be saved.
