---
layout: post
title: daru basic
date: 2019-04-09 12:19:53 +0800
category: Note
tag: Ruby & Rails
---

* content
{: toc}
---

## [DARU](https://github.com/SciRuby/daru)

### 基础
Ruby 数据处理工具，可创建及使用日期索引
主要可用结构：
- Daru::Vector 一维
- Daru::DataFrame 二维
- Daru::DateTimeIndex 时间索引序列

在创建一维或二维结构时，可使用时间索引序列作为索引
```ruby
data = DataTable.select("date,val,total_val").order("date asc")

data_index = Daru::DateTimeIndex.new(data.collect(&:date))
data_dv = Daru::Vector.new(data.collect(&:val), index: data_index)

df = Daru::DataFrame.new(
  {:val=>data.collect(&:val),
  :total_val=>data.collect(&:total_val)
  },
  index:data_index
  )
```

其中`df.val`结构与`data_dv`相同
```ruby
[9] pry(main)> data_dv
=> #<Daru::Vector(19)>
 2018-03-30T00:00:00+                  1.0
 2018-04-02T00:00:00+               0.9999
 2018-04-27T00:00:00+               0.9908
 2018-05-02T00:00:00+                  1.0
 2018-05-31T00:00:00+               1.0885
 2018-06-01T00:00:00+               1.0833
 2018-06-29T00:00:00+               1.0586
                ...                 ...
 ```

可通过时间索引查找数据
```ruby
[14] pry(main)> data_dv['2018-04']
=> #<Daru::Vector(2)>
 2018-04-02T00:00:00+               0.9999
 2018-04-27T00:00:00+               0.9908
[15] pry(main)> data_dv['2018-04-02']
=> 0.9999

[16] pry(main)> data_dv.index
=> #<Daru::DateTimeIndex(19) 2018-03-30T00:00:00+00:00...2018-12-28T00:00:00+00:00>
[17] pry(main)> data_dv.index['2018-04']
=> #<Daru::DateTimeIndex(2) 2018-04-02T00:00:00+00:00...2018-04-27T00:00:00+00:00>
```

### 获取数据的特殊情况

当某月只有一条数据时，按月索引会直接取到该数据
```ruby
[19] pry(main)> data_dv['2018-03']
=> 1.0
```

按月份获取最后日期之后的数据会报错
```ruby
[27] pry(main)> data_dv['2019-01']
ArgumentError: bad value for range
from /usr/local/lib/ruby/gems/2.4.0/gems/daru-0.2.1/lib/daru/date_time/index.rb:547:in `slice_between_dates'
```

获取最早日期前的月份时，获取之前一年内月份都会直接返回全部数据
```ruby
[29] pry(main)> data_dv['2018-02']
=> #<Daru::Vector(19)>
 2018-03-30T00:00:00+                  1.0
 2018-04-02T00:00:00+               0.9999
 2018-04-27T00:00:00+               0.9908
 2018-05-02T00:00:00+                  1.0
 2018-05-31T00:00:00+               1.0885
 2018-06-01T00:00:00+               1.0833
 2018-06-29T00:00:00+               1.0586
                ...                 ...

[29] pry(main)> data_dv['2017-03']
=> #<Daru::Vector(19)>
 2018-03-30T00:00:00+                  1.0
 2018-04-02T00:00:00+               0.9999
 2018-04-27T00:00:00+               0.9908
 2018-05-02T00:00:00+                  1.0
 2018-05-31T00:00:00+               1.0885
 2018-06-01T00:00:00+               1.0833
 2018-06-29T00:00:00+               1.0586
                ...                 ...
```

获取最早日期一年之前月份的数据报错
```ruby
[28] pry(main)> data_dv['2017-02']
ArgumentError: Key 2017-02 is out of bounds
from /usr/local/lib/ruby/gems/2.4.0/gems/daru-0.2.1/lib/daru/date_time/index.rb:362:in `[]'
```

按年获取数据时，如果该年内无数据，会报错
```ruby
[36] pry(main)> data_dv['2019']
ArgumentError: Key 2019 is out of bounds
from /usr/local/lib/ruby/gems/2.4.0/gems/daru-0.2.1/lib/daru/date_time/index.rb:362:in `[]'
[37] pry(main)> data_dv['2017']
ArgumentError: Key 2017 is out of bounds
from /usr/local/lib/ruby/gems/2.4.0/gems/daru-0.2.1/lib/daru/date_time/index.rb:362:in `[]'
```

可以直接取首个数据，但取最后一个数据时需要先转换为 Array 类型
```ruby
[31] pry(main)> data_dv.first
=> 1.0
[33] pry(main)> data_dv.to_a.last
=> 0.8472
[34] pry(main)> data_dv.last
NoMethodError: undefined method `last' for #<Daru::Vector:0x00007fe58a713f28>
from /usr/local/lib/ruby/gems/2.4.0/gems/daru-0.2.1/lib/daru/vector.rb:1420:in `method_missing'
```

### 使用经验

判断是否包含某日数据
```ruby
[48] pry(main)> if data_dv.index.include?('2018-04-02')
[48] pry(main)*   res = data_dv['2018-04-02']
[48] pry(main)* end
=> 0.9999
```

获取某个日期之前或之后的数据
```ruby
[53] pry(main)> dates = Daru::Vector.new(data_dv.index)
=> #<Daru::Vector(19)>
                    0 2018-03-30T00:00:00+
                    1 2018-04-02T00:00:00+
                    2 2018-04-27T00:00:00+
                    3 2018-05-02T00:00:00+
                    4 2018-05-31T00:00:00+
                  ...                  ...
[61] pry(main)> bool = dates < '2018-04-30'
=> #<Daru::Core::Query::BoolArray:70311951424740 bool_arry=[true, true, true, false, false, false, false, false, false, false, false, false, false, false, false, false, false, false, false]>
[62] pry(main)> ids = dates.where(bool)
=> #<Daru::Vector(3)>
                    0 2018-03-30T00:00:00+
                    1 2018-04-02T00:00:00+
                    2 2018-04-27T00:00:00+
[67] pry(main)> ids_index = dates.where(bool).index
=> #<Daru::Index(3): {0, 1, 2}>
[68] pry(main)> new_data = data_dv.at(*ids_index)
=> #<Daru::Vector(3)>
 2018-03-30T00:00:00+                  1.0
 2018-04-02T00:00:00+               0.9999
 2018-04-27T00:00:00+               0.9908
 ```
