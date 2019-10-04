---
layout: post
title: daru from_excel
date: 2019-10-04 16:12:56 +0800
category: Note
tag: Ruby & Rails
---

* content
{: toc}
---

### Error

When reading an excel file by Daru, it may only return the first column sometimes.

![first column](/assets/first_column.png)

### Reason

The file's first row is a merged cell.

### Edit source code

`/usr/local/lib/ruby/gems/2.6.0/gems/daru-0.2.1/lib/daru/io/io.rb`

```ruby
def from_excel path, opts={}
  optional_gem 'spreadsheet', '~>1.1.1'
  opts = {
    worksheet_id: 0
  }.merge opts

  worksheet_id = opts[:worksheet_id]
  book         = Spreadsheet.open path
  worksheet    = book.worksheet worksheet_id
  headers      = ArrayHelper.recode_repeated(worksheet.row(0)).map(&:to_sym)
  ...
end
```

Changed it to:

```ruby      
def from_excel path, opts={}
  optional_gem 'spreadsheet', '~>1.1.1'
  opts = {
    worksheet_id: 0,
    row_id: 0
  }.merge opts

  worksheet_id = opts[:worksheet_id]
  row_id       = opts[:row_id]
  book         = Spreadsheet.open path
  worksheet    = book.worksheet worksheet_id
  headers      = ArrayHelper.recode_repeated(worksheet.row(row_id)).map(&:to_sym)
  ...
end
```

### Change the way to call it

```ruby
df = nil
init_size = 0
row_id = 0
10.times.each do |i|
  begin
    df = Daru::DataFrame.from_excel(file, {:row_id => i})
    new_size = df.vectors.size
  rescue Exception => e
  end
  if !new_size.blank? && new_size > init_size
    init_size = new_size
    row_id = i
  end
end
df = Daru::DataFrame.from_excel(file, {:row_id => row_id})
```
