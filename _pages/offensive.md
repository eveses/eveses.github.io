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

<div class="sub-navigation" style="margin-bottom: 30px;">
  <h3>Categories</h3>
  <ul>
    <li><a href="/hackthebox/">📁 Hack The Box</a> - HTB Write-ups</li>
    <li><a href="/tryhackme/">📁 TryHackMe</a> - THM Write-ups</li>
    <li><a href="/cheatsheet/">📝 Cheatsheet</a> - Pentest Notes</li>
  </ul>
</div>

<hr>

<h3>All Offensive Posts</h3>
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