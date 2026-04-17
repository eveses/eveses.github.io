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

<div class="sub-navigation" style="margin-bottom: 30px;">
  <h3>Categories</h3>
  <ul>
    <li><a href="/aws-saa/">AWS SAA</a> - Solutions Architect Associate</li>
    <li><a href="/aws-scs/">AWS SCS</a> - Security Specialty</li>
    <li><a href="/azure-az-500/">Azure AZ-500</a> - Security Engineer Associate</li>
  </ul>
</div>

<hr>

<h3>All Cloud Posts</h3>
<ul class="post-list--kali">
  {% for post in site.posts %}
    {% if post.categories contains 'aws-saa' or post.categories contains 'aws-scs' or post.categories contains 'azure-az-500' %}
    <li>
      <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
      <small>{{ post.date | date: site.date_format | default: "%Y-%m-%d" }}</small>
    </li>
    {% endif %}
  {% endfor %}
</ul>