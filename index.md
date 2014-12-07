---
layout: layout
title: brly.github.io
---

<p></p>

{% for post in site.posts %}
<div class="panel panel-default">
  <div class="panel-body entries">
    <a href="{{ post.url }}"> {{ post.date | date_to_long_string }} : {{ post.title }} </a>
    {% for tag in post.tags %}
    <span class="label label-primary" style="margin-left: 10px;"> {{ tag }} </span>
    {% endfor %}
  </div>
</div>
{% endfor %}
