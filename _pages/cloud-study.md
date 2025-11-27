---
layout: single
title: "[Cloud Study]"
permalink: /cloud-study/
author_profile: true
classes: fs-title-only

fs_path:
  - { label: home,    url: / }
  - { label: cloud,        url: /cloud/ }
  - { label: cloud-study, url: /cloud-study/ }
---

{% include fs-path.html path=page.fs_path %}

<ul class="post-list--kali">
  {% assign filtered_posts = site.posts | where_exp: "p", "p.categories contains 'cloud-study'" | sort: "date" | reverse %}
  {% for post in filtered_posts %}
    <li>
      <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
      <small>{{ post.date | date: site.date_format | default: "%Y-%m-%d" }}</small>
    </li>
  {% endfor %}
</ul>