---
layout: single
title: "[Tryhackme]"
permalink: /tryhackme/
author_profile: true
classes: fs-title-only

fs_path:
  - { label: home,  url: / }
  - { label: tryhackme, url: /tryhackme/ }
---

{% include fs-path.html path=page.fs_path %}

<ul class="post-list--kali">
  {% assign filtered_posts = site.posts | where_exp: "p", "p.categories contains 'tryhackme'" | sort: "date" | reverse %}
  {% for post in filtered_posts %}
    <li>
      <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
      <small>{{ post.date | date: site.date_format | default: "%Y-%m-%d" }}</small>
    </li>
  {% endfor %}
</ul>
