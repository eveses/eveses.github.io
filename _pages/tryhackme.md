---
layout: single
title: "[Tryhackme]"
permalink: /tryhackme/
author_profile: true
classes: fs-title-only

fs_path:
  - { label: home,  url: / }
  - { label: offensive,  url: /offensive/ }
  - { label: tryhackme, url: /tryhackme/ }
---

{% include fs-path.html path=page.fs_path %}

<div class="collection-heading">
  <p class="collection-heading__eyebrow">LAB NOTES / TRYHACKME</p>
  <p class="collection-heading__description">Rooms, attack paths, and the techniques learned along the way.</p>
</div>

{% include offensive-tabs.html %}

{% assign filtered_posts = site.posts | where_exp: "p", "p.categories contains 'tryhackme'" | sort: "date" | reverse %}
<div class="collection-section-heading">
  <h2>TryHackMe</h2>
  <span>{{ filtered_posts.size }} write-ups</span>
</div>

<div class="post-card-list">
  {% for post in filtered_posts %}
    {% include post-card.html post=post %}
  {% endfor %}
</div>
