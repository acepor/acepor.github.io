---
layout: post
title: Tips of Pandas
tags: [python, numpy]
---

### View the header of a pandas table

{% highlight python %}

import pandas as pd

data = pd.read_vcsv(FILE)
list(data.head())

{% endhighlight %}
