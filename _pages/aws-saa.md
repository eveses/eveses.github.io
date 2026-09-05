---
layout: single
title: "[AWS SAA]"
permalink: /aws-saa/
author_profile: true
classes: fs-title-only

fs_path:
  - { label: home,    url: / }
  - { label: cloud,   url: /cloud/ }
  - { label: aws-saa, url: /aws-saa/ }
---

{% include fs-path.html path=page.fs_path %}

<div class="collection-heading">
  <p class="collection-heading__eyebrow">AWS / SOLUTIONS ARCHITECT</p>
  <p class="collection-heading__description">컴퓨팅, 스토리지, 데이터베이스, 네트워크를 요구사항에 맞게 조합하기 위한 AWS 아키텍처 노트입니다.</p>
</div>

{% include cloud-tabs.html active='aws-saa' %}

{% assign filtered_posts = site.posts | where_exp: "p", "p.categories contains 'aws-saa'" | sort: "date" | reverse %}
<div class="collection-section-heading"><h2>AWS SAA notes</h2><span>{{ filtered_posts.size }} entries</span></div>
<div class="post-card-list">
  {% for post in filtered_posts %}{% include post-card.html post=post %}{% endfor %}
</div>
