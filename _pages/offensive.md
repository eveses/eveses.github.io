---
layout: single
title: "Offensive Security"
permalink: /offensive/
author_profile: true
classes: fs-title-only

fs_path:
  - { label: home,      url: / }
  - { label: offensive, url: /offensive/ }
---

{% include fs-path.html path=page.fs_path %}

<h3>Offensive Security Archives</h3>
<ul class="post-list--kali">
  {% for post in site.posts %}
    {% if post.categories contains 'hackthebox' or post.categories contains 'tryhackme' or post.categories contains 'cheatsheet' %}
    <li>
      <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
      <small>{{ post.date | date: site.date_format | default: "%Y-%m-%d" }}</small>
    </li>
    {% endif %}
  {% endfor %}
</ul>