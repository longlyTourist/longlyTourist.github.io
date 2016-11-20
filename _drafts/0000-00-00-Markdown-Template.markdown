---
layout:     post
title:      "Markdown Template"
subtitle:   ""
date:       2016-11-18 12:00:00
author:     "Silence"
header-img: "img/postbg/think.jpg"
catalog:    true
tags:
    - Java
    - Thinking in Code
---

> 转载请注明出处。

# 段落

Now is the time for all good men to come to
the aid of their country. This is just a
regular paragraph.

The quick brown fox jumped over the lazy
dog's back.

## Header 2
### Header 3
#### Header 4
##### Header 5
###### Header 6

> This is a blockquote.
>
> This is the second paragraph in the blockquote.
>
> ## This is an H2 in a blockquote

# 强调

Some of these words *are emphasized*.
Some of these words _are emphasized also_.
Use two asterisks for **strong emphasis**.
Or, if you prefer, __use two underscores instead__.

# 列表

### 无序列表

* Candy.
* Gum.
* Booze.

+ Candy.
+ Gum.
+ Booze.

- Candy.
- Gum.
- Booze.

## 有序列表

1. Red
2. Green
3. Blue

## 带空行

如果你在项目之间插入空行，那项目的内容会用 <p> 包起来，你也可以在一个项目内放上多个段落，只要在它前面缩排 4 个空白或 1 个 tab 。

* A list item.
With multiple paragraphs.

* Another item in the list.


# 链接

## 行内链接

This is an [example link](http://example.com/).

This is an [example link](http://example.com/ "With a Title").

## 参考链接

title 属性是可选的，链接名称可以用字母、数字和空格，但是不分大小写

I get 10 times more traffic from [Google][1] than from
[Yahoo][2] or [MSN][AA].

[1]: http://google.com/ "Google"
[2]: http://search.yahoo.com/ "Yahoo Search"
[aa]: http://search.msn.com/ "MSN Search"

I start my morning with a cup of coffee and
[The New York Times][NY Times].

[ny times]: http://www.nytimes.com/


# 图片

## 行内形式

![alt text](/img/contact-bg.jpg "Title")

![alt text](https://cdn-images-1.medium.com/max/800/1*MRPl_SNuRGJchb6eOAnkSA.jpeg "Title")

## 参考形式

![alt text][id]

[id]: /img/contact-bg.jpg "Title"

# 代码

I strongly recommend against using any `<blink>` tags.

I wish SmartyPants used named entities like `&mdash;`
instead of decimal-encoded entites like `&#8212;`.

```html
<blockquote>
<p>For example.</p>
</blockquote>
```
