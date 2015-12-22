---
layout: post
title: Converting Two formats of Chinese Texts with OpenCC
---

Often, we have a dataset with mixed formats of Chinese characters: the simplified Chinese used in mainland China, and the traditional Chinese used in other areas. It is not a good idea to ignore the mixed usage of these two forms, because it will bring further problems in the later processing. To overcome this, we use [OpenCC](https://github.com/BYVoid/OpenCC) by [BYVoid](opencc.byvoid.com/).

OpenCC claims to be [a precise conversion](https://github.com/BYVoid/OpenCC/wiki/%E7%B7%A3%E7%94%B1) solution between the traditional and simplified Chinese. Hence, the included traditional Chinese corpus has two different sub-divisions: Hong Kong standard (香港小學學習字詞表標準) and Taiwan Standard (臺灣正體標準).

OpenCC is relatively easy to use. The default setting is to convert texts to traditional Chinese form regardless its original format.

{% highlight bash %}

opencc -i INPUT_FILE -o OUTPUT_FILE

{% endhighlight %}

If we would like to convert traditional Chinese to simplified Chinese, we need use the `-c` flag to claim the configuration:

In addition, OpenCC provides 10 configuration settings [here](https://github.com/BYVoid/OpenCC/tree/master/data/config).

1. s2t.json Simplified Chinese to Traditional Chinese 簡體到繁體
2. t2s.json Traditional Chinese to Simplified Chinese 繁體到簡體
3. s2tw.json Simplified Chinese to Traditional Chinese (Taiwan Standard) 簡體到臺灣正體
4. tw2s.json Traditional Chinese (Taiwan Standard) to Simplified Chinese 臺灣正體到簡體
5. s2hk.json Simplified Chinese to Traditional Chinese (Hong Kong Standard) 簡體到香港繁體（香港小學學習字詞表標準）
6. hk2s.json Traditional Chinese (Hong Kong Standard) to Simplified Chinese 香港繁體（香港小學學習字詞表標準）到簡體
7. s2twp.json Simplified Chinese to Traditional Chinese (Taiwan Standard) with Taiwanese idiom 簡體到繁體（臺灣正體標準）並轉換爲臺灣常用詞彙
8. tw2sp.json Traditional Chinese (Taiwan Standard) to Simplified Chinese with Mainland Chinese idiom 繁體（臺灣正體標準）到簡體並轉換爲中國大陸常用詞彙
9. t2tw.json Traditional Chinese (OpenCC Standard) to Taiwan Standard 繁體（OpenCC 標準）到臺灣正體
10. t2hk.json Traditional Chinese (OpenCC Standard) to Hong Kong Standard 繁體（OpenCC 標準）到香港繁體（香港小學學習字詞表標準）

Therefore, if we would like to convert simplified Chinese to Traditional Chinese (Taiwan Standard), we could use:

{% highlight bash %}

opencc -i INPUT_FILE -o OUTPUT_FILE -c s2tw.json

{% endhighlight %}

Just as simple as this. OpenCC provides a convenient solution to tackle the Chinese conversion problem.

One more thing, if you want to see the result of OpenCC, there is an online demo [here](http://opencc.byvoid.com/).
