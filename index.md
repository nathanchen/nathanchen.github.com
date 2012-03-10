---
layout: page
title: Hello World!
---
{% include JB/setup %}

### 欢迎！

这是我分享自己阅读和经历的博客。

从这个博客中，您可以看到我读了那些文章，做了那些事，对什么技术感兴趣还有我的想法。

我一直有读文章记笔记的习惯，我将很快把我所有的笔记整理到这个博客上。

同时我自己搭建了一个J2EE的个人网站，暂时没找到合适的托管，所以还没上线，但是我会在这个博客中分享网站所用到的技术。

<br />
## 关于我
    
    title : My Blog
    
    author :
      name : 陈嵚
      email : natechen AT me.com
      github : nathanchen

<br />
## 关于文章
<br />
<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>




