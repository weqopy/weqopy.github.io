---
layout: post
title: Daru::DateTimeIndex-2
date: 2019-07-04 07:16:42 +0800
category: Note
tag: Ruby & Rails
---

* content
{: toc}
---

## Init Data

```ruby
ids = Daru::DateTimeIndex.new(nis(&:net_date))
df = Daru::DataFrame.from_activerecord(nis, :net_date, :val)
```

