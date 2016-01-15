---
layout: post
title: Magic commands in IPython
tags: [python, ipython]
---

[IPython](https://ipython.org/) is a integrated Python interface, and it introduces a set of magic commands which start with `%`. These magic commands are very handy when we use IPython. This post discusses several helpful magic commands.

`%logstart` and `%logstop`: These two commands can be used to switch on and off the logging mode in IPython. With `%logon` and `logoff` commands, the logging mode can be switched on and off when logging.

`%paste`: We often use IPython as a testing environment, so we want to paste a function written in an IDE to IPython. However, if we just use copy + paste, the format might become messy. Here comes `%paste`, which will perfectly reserve the indentations and other formats.

`%run`: If we write a python script in an IDE, and would like to test the whole script or just execute it, we can do this inside of IPython with the `%run` command.

`%save`: This command can save logged IPython history to a file. The default destination is to the working directory.

`%timeit`: Sometime, we would like to compare the efficiency of different functions. Although Python's `time` library might help, IPython provides a built-in solution for this problem. We can use `%timeit` to carry out the comparison. Suppose we have a function called `my_function`, we can test it as:

{% highlight python %}

%timeit my_function

{% endhighlight %}

No parenthesis behind the function name.
