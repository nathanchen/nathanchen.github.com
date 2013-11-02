---
layout: frontpage
title: Hello World!
---
{% include JB/setup %}


## 关于我
   
    name : 陈嵚

    email : {{ site.author.email }}

    github : {{ site.author.github }}

<br />

## 最近更新


<ul class="posts">
  {% for post in site.posts %}
    {% assign option_index = forloop.index0 %}
    {% if option_index < 5 %}
          <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
    {% endif %}
  {% endfor %}
</ul>