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

<div class="collection-heading">
  <p class="collection-heading__eyebrow">FIELD NOTES / OFFENSIVE SECURITY</p>
  <p class="collection-heading__description">Walkthroughs and practical references, organized for quick scanning.</p>
</div>

{% include offensive-tabs.html %}

{% assign offensive_count = 0 %}
{% for post in site.posts %}
  {% if post.categories contains 'hackthebox' or post.categories contains 'tryhackme' or post.categories contains 'cheatsheet' %}
    {% assign offensive_count = offensive_count | plus: 1 %}
  {% endif %}
{% endfor %}
<div class="collection-section-heading">
  <h2>All notes</h2>
  <span>{{ offensive_count }} entries</span>
</div>

<div class="post-card-list">
  {% for post in site.posts %}
    {% if post.categories contains 'hackthebox' or post.categories contains 'tryhackme' or post.categories contains 'cheatsheet' %}
      {% include post-card.html post=post %}
    {% endif %}
  {% endfor %}
</div>
