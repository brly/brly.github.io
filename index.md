---
layout: layout
title: brly.github.io
---

# meta

<ul class="mdc-list mdc-list--dense">
  <li class="mdc-list-item">
    <span class="meta_font">twitter</span>
    <span class="mdc-list-item__meta">
      <a href="https://twitter.com/brly__" class="meta_font">https://twitter.com/brly__</a>
    </span>
  </li>
  <li class="mdc-list-item">
    <span class="meta_font">github</span>
    <span class="mdc-list-item__meta">
      <a href="https://github.com/brly" class="meta_font">https://github.com/brly</a>
    </span>
  </li>
  <li class="mdc-list-item">
    <span class="meta_font">mailto</span>
    <span class="mdc-list-item__meta">
      <span class="meta_font">receive＠brly.net</span>
    </span>
  </li>
</ul>

# publication

<p />

<div class="mdc-list-group">
{% for pub in site.data.publications %}
  <span class="mdc-list-group__subheader" style="font-size: 24px">{{pub.title}}</span>
  <ul class="mdc-list mdc-list--two-line">
    <li class="mdc-list-item">
      <span class="mdc-list-item__text">
        {{pub.desc}}
        <span class="mdc-list-item__secondary-text">
          <a href="{{pub.url}}">{{pub.url}}</a>
        </span>
      </span>
    </li>
  </ul>
{% endfor %}
</div>
