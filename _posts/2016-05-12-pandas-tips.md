---
layout: post
title: Tips of Pandas
tags: [python, numpy]
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

###

{% highlight python %}


{% endhighlight %}
