---
layout: page
title: Hello World!
---
{% include JB/setup %}

### 欢迎！

这是我分享自己阅读笔记和项目经历的博客。

从这个博客中，您可以看到我读了那些文章，做了那些项目，对什么技术感兴趣还有我的一些想法。

<br />
## 关于我
    
   
    name : 陈嵚

    email : { site.author.email }

    github : { site.author.github }

<br />
## 最近更新
<br />
<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>