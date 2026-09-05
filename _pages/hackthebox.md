---
layout: single
title: "[Hackthebox]"
permalink: /hackthebox/
author_profile: true
classes: fs-title-only  

fs_path:
  - { label: home,    url: / }
  - { label: offensive,  url: /offensive/ }
  - { label: hackthebox, url: /hackthebox/ }
---

{% include fs-path.html path=page.fs_path %}

<div class="collection-heading">
  <p class="collection-heading__eyebrow">LAB NOTES / HACK THE BOX</p>
  <p class="collection-heading__description">Retired machines documented from enumeration to privilege escalation.</p>
</div>

{% include offensive-tabs.html %}

{% assign filtered_posts = site.posts | where_exp: "p", "p.categories contains 'hackthebox'" | sort: "date" | reverse %}
<div class="collection-section-heading">
  <h2>Hack The Box</h2>
  <span>{{ filtered_posts.size }} write-ups</span>
</div>

<div class="post-card-list">
  {% for post in filtered_posts %}
    {% include post-card.html post=post %}
  {% endfor %}
</div>
