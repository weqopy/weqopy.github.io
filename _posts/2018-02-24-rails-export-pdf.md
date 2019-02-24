---
layout: post
title: rails export pdf
date: 2019-02-24 21:51:01 +0800
category: Note
tag: Ruby & Rails
---

* content
{: toc}
---

# Rails 导出 PDF 的两种方式

## 调用浏览器打印功能

通过 `target='_blank'` 跳转至新页面，通过 `window.print()` 调出浏览器打印页面、确认打印

*可通过参数判断处理查看、打印双重功能*

## 使用 PDFkit gem

PDFkit 通过页面链接生成并保存 PDF 文件，页面需进行跳过登录验证处理

```ruby
skip_before_action :authenticate_account!, only: [:export_pdf, :page_view]

def page_view
  render :layout => nil
end

def export_pdf
  dm = Digest::SHA1.hexdigest("#{}|#{}|_redeem")
  kit = PDFKit.new "http://example.com/controller_name/page_view/?dm=#{dm}"
  output_path = VPS_PATH
  kit.to_file(output_path)
  send_file(output_path, :filename => "name.pdf")
end
```
