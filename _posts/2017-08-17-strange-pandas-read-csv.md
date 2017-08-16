---
layout: post
title: A Strange Problem caused by indexing in pandas.read_csv
tags: [Python, Pandas, Data Structure]
---
Yesterday, we encountered a very strange bug caused by the indexing mechanism in `pandas.read_csv` function

Due to the huge data size, we used `pandas` to process data, but a very strange phenomenon occurred. The pseudo code looks like this:

{% highlight python %}

reader = pd.read_csv(IN_FILE, chunksize = 1000, engine='c')
for chunk in reader:
    result = []
    for line in chunk.tolist():
         temp = complicated_process(chunk)  # this involves a very complicated processing, so here is just a simplified version
         result.append(temp)
    chunk['new_series'] = pd.series(result)
    chunk.to_csv(OUT_TILE, index=False, mode='a')

{% endhighlight %}

We can confirm each loop of result is not empty. But only in the first time of the loop, line `chunk['new_series'] = pd.series(result)` has result, and the rest are empty. Therefore, only the first chunk of the output contains new_series, and the rest are empty.

Three other colleagues helped me to debug the code, but none of them thought there was any problem in this logic.

We posted this problem on [StackOvervlow](https://stackoverflow.com/questions/45706833/strange-indexing-mechanism-of-pandas-read-csv-function-with-chunksize-option), and half hour later, we received an answer. The helper told us that we need move the initialization of `result` outside the chunk. Hence, the logic would be:

{% highlight python %}

reader = pd.read_csv(IN_FILE, chunksize = 1000, engine='c')
result = []
for chunk in reader:
    for line in chunk.tolist():
         temp = complicated_process(chunk)  # this involves a very complicated processing, so here is just a simplified version
         result.append(temp)
    chunk['new_series'] = pd.series(result)
    chunk.to_csv(OUT_TILE, index=False, mode='a')

{% endhighlight %}

Strangely, this strategy worked, but this was very awkward to us. In this way, the `result` will hold every single line of the whole file, instead of every line in each chunk, which means that the size of `result` is not equivalent to `chunk`, so how can we concatenate them together? We were really confused.

When we tracked the index of each chunk, we found that they are not individual. We supposed that each chunk would start the index from 0, but in reality, it is NOT. The index of each chunk is a subset of the whole CSV in this situation, so their index derives from the CSV. This is what caused the problem. In the above example, the `pandas.to_csv` writes only the result of the first chunk, instead of the last chunk.

Therefore, a better solution would be rebuild index for each chunk, and concatenating it with result.

{% highlight python %}

reader = pd.read_csv(IN_FILE, chunksize = 1000, engine='c')
for chunk in reader:
    result = []
    for line in chunk.tolist():
        temp = complicated_process(chunk)
        result.append(temp)
    new_chunk = chunk.reindex()
    new_chunk = new_chunk.assign(new_series=result)
    new_chunk.to_csv(OUT_TILE, index=False, mode='a')

{% endhighlight %}

Line `new_chunk = chunk.reindex()` is the key to solve this problem.

Lesson learned: do not trust your intuition when you try a new function.
