---
layout: post
title: "Chrome Connecting to Random Domains On Start"
category: Reading Notes 
tags: ["读文章", "Chrome", "浏览器"]
---
{% include JB/setup %}

我们知道，很多ISP都喜欢拦截你访问错误网址的页面，来显示一个充满广告的网页，然后用小字告诉你这个网址不存在，比如中国联通这样的。而如果你用Chrome的话，就不会看到ISP 的广告页面，因为Chrome有自己的一套。

在Chrome刚刚启动的时候，它就会访问三个随机生成的不存在的网址，它们都应该返回502错误，这样Chrome在一开始就可以通过错误网址返回的IP来探测到你的ISP会不会用自己的页面来代替502错误网页。

如果ISP确实有这个小动作，那么Chrome就会阻止用户用ISP来查询DNS，而是转而使用自己的查询服务。比如你错误的输入了http://guao/，那么Chrome会提示你是否要访问的是guao.hk（因为guao的Google搜索结果第一个网站就是guao.hk），而不会让你进入一个满是广告而一点忙都帮不上的ISP提供的网页。

很聪明的做法。

## Reference

gHacks.net, http://www.ghacks.net/2012/02/18/chrome-connecting-to-random-domains-on-start-here-is-why/, viewed on 19/02/2012
