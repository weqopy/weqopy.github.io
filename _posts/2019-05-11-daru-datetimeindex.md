---
layout: post
title: Daru::DateTimeIndex
date: 2019-05-11 22:59:49 +0800
category: Note
tag: Ruby & Rails
---

* content
{: toc}
---

## init data

```ruby
[7] pry(main)> ids = Daru::DateTimeIndex.new(['2019-01-01','2019-01-04','2019-02-01','2019-02-02','2019-03-03'])
=> #<Daru::DateTimeIndex(5) 2019-01-01T00:00:00+00:00...2019-03-03T00:00:00+00:00>
[8] pry(main)> df = Daru::DataFrame.new(
[8] pry(main)*   {
[8] pry(main)*     'row_1' => [1, 2, 3, 4, 5],
[8] pry(main)*     'row_2' => [1, 2, 3, 4, 5],
[8] pry(main)*     'row_3' => [1, 2, 3, 4, 5]
[8] pry(main)*   },
[8] pry(main)*   index: ids
[8] pry(main)* )
=> #<Daru::DataFrame(5x3)>
                 row_1      row_2      row_3
 2019-01-01          1          1          1
 2019-01-04          2          2          2
 2019-02-01          3          3          3
 2019-02-02          4          4          4
 2019-03-03          5          5          5
```

## split by date

```ruby
[10] pry(main)> df.row['2019-02']
=> #<Daru::DataFrame(2x3)>
                 row_1      row_2      row_3
 2019-02-01          3          3          3
 2019-02-02          4          4          4
[11] pry(main)> df.row['2019-01-02','2019-02-02']
=> #<Daru::DataFrame(3x3)>
                 row_1      row_2      row_3
 2019-01-04          2          2          2
 2019-02-01          3          3          3
 2019-02-02          4          4          4
```

## error

```ruby
[12] pry(main)> ids = Daru::DateTimeIndex.new([])
=> #<Daru::DateTimeIndex(0)>
[13] pry(main)> df = Daru::DataFrame.new(
[13] pry(main)*   {
[13] pry(main)*     'row_1' => [1, 2, 3, 4, 5],
[13] pry(main)*     'row_2' => [1, 2, 3, 4, 5],
[13] pry(main)*     'row_3' => [1, 2, 3, 4, 5]
[13] pry(main)*   },
[13] pry(main)*   index: ids
[13] pry(main)* )
IndexError: Expected index size >= vector size. Index size : 0, vector size : 5
from /usr/local/lib/ruby/gems/2.4.0/gems/daru-0.2.1/lib/daru/vector.rb:1534:in `guard_sizes!'
```
