---
layout: post
title: ruby symbol string
date: 2019-05-01 02:06:15 +0800
category: Note
tag: Ruby & Rails
---

* content
{: toc}
---

### symbol

对象，一般作为名称标签使用，表示方法名称。
与字符串可相互转换。

相同内容的 symbol 相同，类似 Python 中 id 相同。而内容相同的字符串并不是同一个对象。

### symbol 与 string

都是 Ruby 中表示文本的方式，可以作为 hash 的键。

symbol 比 string 更加节省内容，提高运行效率。

### 使用选择

使用字符串的内容，这个内容可能会变化，使用 String

使用固定的名字或者说是标识符，使用 Symbol，枚举值、关键字（哈希表关键字、方法的参数）等。
