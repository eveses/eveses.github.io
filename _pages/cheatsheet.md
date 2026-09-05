---
layout: single
title: "[Cheatsheet]"
permalink: /cheatsheet/
author_profile: true
classes: fs-title-only

fs_path:
  - { label: home,       url: / }
  - { label: offensive,  url: /offensive/ }
  - { label: cheatsheet, url: /cheatsheet/ }  
---

{% include fs-path.html path=page.fs_path %}

<div class="collection-heading">
  <p class="collection-heading__eyebrow">QUICK REFERENCE / CHEATSHEETS</p>
  <p class="collection-heading__description">Compact commands and attack notes built for use during practice.</p>
</div>

{% include offensive-tabs.html %}

{% assign filtered_posts = site.posts | where_exp: "p", "p.categories contains 'cheatsheet'" | sort: "date" | reverse %}
<div class="collection-section-heading">
  <h2>Cheatsheets</h2>
  <span>{{ filtered_posts.size }} references</span>
</div>

<div class="post-card-list">
  {% for post in filtered_posts %}
    {% include post-card.html post=post %}
  {% endfor %}
</div>
