---
layout: post
title: Backreference in Python's re.sub function
---

While regular expression is a widely-used technique in text processing, it seems a not-easy-to-learn technique. Often, an experienced programmer might find an unfamiliar point. I have been using regular expression in bash for several years, but when I switched to Python, I found it a totally new world. I finished reading the regular expression chapters in [_Beginning Python_](https://www.apress.com/9781590599822) by Magnus Lie and [_Core Python Programming 2nd ed_](http://corepython.com/) by Wesley Chun, but neither of them has mentioned the backreference mechanism in Python. In this post, we will discuss about backreference in Python, and will use the case in sed as illustration.

Recently, we had a case that we need using `re.sub` in Python, in addition, we need backreference to help us. Simply put, backreference is a mechanism to reuse the matched patterns in the later task: often, when we substitute one part of a matched pattern, we want to keep the rest part unchanged. This can be a bit tricky. For example, in a mockup string 'zootoozootoo', we want to substitute the two 'o's between 'z' and 't' with two zeros, but we we want to keep the 'o's between 'z' and 't' unchanged. We cannot do a global substitution to replace all 'o's with zeros as

{% highlight bash %}

echo 'zootoozootoo' | sed 's/oo/00/g' | less

{% endhighlight %}

Instead, we need a captured group to keep the unchanged part, and use backreference to refer the captured group. In sed, this is very straightforward: we use \\(\\) to construct a captured group, and use \num to refer the captured group.

{% highlight bash %}

echo 'zootoozootoo' | sed 's/\(z\)oo\(t\)/\100\2/g' | less

{% endhighlight %}

The backreference mechanism also exists in Python, but it is not as straightforward as that in sed. There are two ways to construct a captured group: either with or without a name. The first one is to use a pair of parentheses, as `(...)`, and the second one has the form as `(?P<name>...)`, where users can define the name of the captured group. However, backreferening the captured group is a little bit complicated. In the Python doc [7.2.1. Regular Expression Syntax](https://docs.python.org/2/library/re.html#regular-expression-syntax), the specific guide is given as:

If the captured groups is constructed as `(?P<quote>['"]).*?(?P=quote)`, then the backreference may have the following formats.

|        Context of reference to group “quote”         | Ways to reference it      |
|: ---------------------------------------------------:|:-------------------------:|
|              in the same pattern itself              | (?P=quote) (as shown), \1 |
| when processing match object m | m.group('quote'), m.end('quote') (etc.)         |
|  in a string passed to the repl argument of re.sub() | \g\<quote\>, \g<1>, \1    |

In short, there are three contexts of using the backreference mechanism, and each of them has its own grammar. If we want to use it in the `re.sub` function, we will apply the last context, and we discuss it in detail here.

If we use the unnamed captured group, the grammar is as following:

{% highlight python %}

from re import compile, search

tt = 'zootoozootoo'

O_ZERO = compile('(z)oo(t)')

O_ZERO.sub('\g<1>00\g<2>', tt)

{% endhighlight %}

First, we use `re.compile` to construct two unnamed captured groups as `(z)oo(t)`, and then we use `\g<1>00\g<2>` in `re.sub` to backreference them. The output is as same as in sed: `zOOtoozOOtoo`.

The second option goes with the named captured group:

{% highlight python %}

O_ZERO = compile('(?P<ZZ>z)oo(?P<TT>t)')

O_ZERO.sub('\g<ZZ>OO\g<TT>', tt)

{% endhighlight %}

We first use `(?P<ZZ>z)oo(?P<TT>t)` to construct two named captured groups: ZZ and TT, and then we use `\g<ZZ>OO\g<TT>` in `re.sub` to backreference them. The output is as same as in the above example: `zOOtoozOOtoo`. Both solutions worked quite well.

Although the official guide provides the third option to backreference in `re.sub` function, which looks like the simplest form, it definitely is the most confusing form. If we construct the unnamed group as in the first form, the output is out of our expectation.

{% highlight python %}

O_ZERO = compile('(z)oo(t)')

O_ZERO.sub('\100\2', tt)

{% endhighlight %}

The output is `\x01OO\x02oo\x01OO\x02oo`.

Similarly, even if we use the named captured group as in the second form, the result still keeps unexpected:

{% highlight python %}

O_ZERO = compile('(?P<ZZ>z)oo(?P<TT>t)')

O_ZERO.sub('\100\2', tt)

{% endhighlight %}

The output is `\x01OO\x02oo\x01OO\x02oo`.

Under [`re.sub` function](https://docs.python.org/2/library/re.html#re.sub) in the official guide, the reason of this unexpected behavior is explained as 'ambitious'.

> \g<number> uses the corresponding group number; \g<2> is therefore equivalent to \2, but isn’t ambiguous in a replacement such as \g<2>0. \20 would be interpreted as a reference to group 20, not a reference to group 2 followed by the literal character '0'.

Therefore, we strongly suggest not using the third form in the official guide.

Last but not least, we have another solution to bypass the backreference in `re.sub` function. The idea is to use the non-capturing group to find the pattern, and substitute the whole pattern instead of using the backreference.

{% highlight python %}

O_ZERO = compile('(?:z)oo(?:t)')

O_ZERO.sub('z00t', tt)

{% endhighlight %}

`(?:)` is the non-capturing group used in `re`, which means that the pattern is used to capture the whole pattern, but not reserved in the output. In the above example, `(?:z)` and `(?:z)` were used to capture the whole pattern, but the captured result does not include these two characters. Instead, in the `re.sub`, we add 'z' and 't' to the beginning and the end of the intended replacement 'OO'. The final output is still `z00tooz00too`. Moreover, this is theoretically memory-efficiant than using backreference as the size of the captured group is smaller than above examples using backreference.

The post gave a detailed introduction of the backreference mechanism in Python. Hope it helps.
