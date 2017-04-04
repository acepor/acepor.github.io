---
layout: post
title: An Elegant Way to Replace Items in Pandas
tags: [Python, Pandas]
---

When we deal with a `Pandas DataFrame`, it is often we need to convert values of cells according specific conditions.

Suppose that we have a DataFrame that contains city info, and we need may specific cities into different tiers. One of my former colleagues provided such a solution:

{% highlight python %}
import numpy as np

def map_city_tier(df):
    """
    :param df: a pd df containing City
    :return: a pd df containing CityTier
    """
    df.CityTier = np.where(df.City.isin(CITY0), '0',
                           np.where(df.City.isin(CITY1), '1',
                                    np.where(df.City.isin(CITY2), '2',
                                             np.where(df.City.isin(CITY3), '3', '4'))))
    return df

{% endhighlight %}

In this case, `np.where` is used to map items. Nice try, but if we have much more tiers, this way will be redundant. Yes, we can use a recursion, but we have a much simpler solution to solve this problem.

In a previous [post](http://localhost:4000/2017/03/14/Intro-spaCy/), I mentioned using a dictionary to replace the `if...else` logic, and here we go, we will use `dict` here as well.

First, we can create a mapping dict.

{% highlight python %}

MAPPING = {'city1': 1, 'city2': 2, 'city3': 1, 'city4': 3, ...}

{% endhighlight %}

The we can use a simple `dict.get()` to replace the items in a DataFrame. Here is the magic:

{% highlight python %}

def map_city_tier2(df):
    df.CityTier = df.City.apply(MAPPING.get)
    return df

{% endhighlight %}

Yes, just one `dict` and one line of code, and the problem has been easily solved. Super easy, right?
