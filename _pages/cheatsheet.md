---
layout: single
title: "[Cheatsheet]"
permalink: /cheatsheet/
author_profile: true
classes: fs-title-only

fs_path:
  - { label: home,       url: / }
  - { label: cheatsheet, url: /cheatsheet/ }   # 라벨을 "cheat"로 줄이고 싶으면 label만 변경
---

{% include fs-path.html path=page.fs_path %}

<ul class="post-list--kali">
  {% assign filtered_posts = site.posts | where_exp: "p", "p.categories contains 'cheatsheet'" | sort: "date" | reverse %}
  {% for post in filtered_posts %}
    <li>
      <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
      <small>{{ post.date | date: site.date_format | default: "%Y-%m-%d" }}</small>
    </li>
  {% endfor %}
</ul>
