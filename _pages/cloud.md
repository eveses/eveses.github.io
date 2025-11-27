---
layout: single
title: "Cloud Security"
permalink: /cloud/
author_profile: true
classes: fs-title-only

fs_path:
  - { label: home,  url: / }
  - { label: cloud, url: /cloud/ }
---

{% include fs-path.html path=page.fs_path %}

<h3>Cloud & Architecture Archives</h3>
<ul class="post-list--kali">
  {% for post in site.posts %}
    {% if post.categories contains 'aws-saa' or post.categories contains 'aws-security' or post.categories contains 'cloud-study' %}
    <li>
      <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
      <small>{{ post.date | date: site.date_format | default: "%Y-%m-%d" }}</small>
    </li>
    {% endif %}
  {% endfor %}
</ul>