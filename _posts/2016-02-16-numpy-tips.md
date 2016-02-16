---
layout: post
title: Tips of Numpy
tags: [python, numpy]
---

### Calculate item frequency in an Numpy array

If we want to calculate the item frequency in a list, it is quite simple:

{% highlight python %}

from collections import Counter
tt = u'教室 少 ， 设施 陈旧 ， 与 其他 早 教 中心 比 ， 硬件 确实 差 了 些 ， 但 收费 挺 高 ， 性价比 低 ， 不 会 选择 。'
t1 = tt.split(' ')
Counter(tt)

{% endhighlight %}

If we want to do the same thing with a Numpy array, it is slightly different:

{% highlight python %}

import numpy as np
tn = np.array(t1)
np.unique(tn, return_counts=True)

{% endhighlight %}

Although Scipy provides a similar solution, if is not as fast as `unique` in Numpy, the comp can be found at this [Stackoverflow post](http://stackoverflow.com/a/25943480).

{% highlight python %}

from scipy.stats import itemfreq
itemfreq(tn)

{% endhighlight %}

Bonus:

`numpy.unique` has two other optional parameters: `return_index` and `return_inverse`:

{% highlight python %}

np.unique(tn, return_index=True)

np.unique(tn, return_inverse=True)

{% endhighlight %}

### Conditional expression (ternary operator)

In python, a simple `if ..., else ...` statement can construct a ternary operator:

{% highlight python %}

if x > 3:
  print x
else:
  print 0

print x if x > 3 else 0

{% endhighlight %}

In Numpy, such a ternary operator can be applied to arrays:

{% highlight python %}

arr = randn(4, 4)
np.where(arr > 0, 2, -2)

{% endhighlight %}

In this example, a random 4 by 4 matrix was constructed: with np.where, if the element in the original matrix is bigger than 0, it will be converted to 2; if smaller than 0, it will be converted to -2.

Another example:

{% highlight python %}

xarr = np.array([1.1, 1.2, 1.3, 1.4, 1.5])
yarr = np.array([2.1, 2.2, 2.3, 2.4, 2.5])
cond = np.array([True, False, True, True, False])
np.where(cond, xarr, yarr)

{% endhighlight %}
