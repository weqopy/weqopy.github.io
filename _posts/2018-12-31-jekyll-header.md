---
layout: post
title: jekyll header
date: 2018-12-31 14:19:55 +0800
category: Default
tag: Temp
---

* content
{: toc}
---


*jekyll header snippet in vscode*
```json
"Posts": {
    "prefix": "snp-posts-header",
    // "scope": "markdown",
    "body": [
        "---",
        "layout: post",
        "title: ${1:title}",
        "date: $CURRENT_YEAR-$CURRENT_MONTH-$CURRENT_DATE $CURRENT_HOUR:$CURRENT_MINUTE:$CURRENT_SECOND +0800",
        "category: ${2:Default}",
        "tag: ${3:Temp}",
        "---",
        "",
        "* content",
        "{: toc}",
        "",
        "",
        "$0"
    ],
    "description": "weqopy site posts header"
}
```
