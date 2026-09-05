---
layout: single
title: "[Azure AZ-500]"
permalink: /azure-az-500/
author_profile: true
classes: fs-title-only

fs_path:
  - { label: home,    url: / }
  - { label: cloud,        url: /cloud/ }
  - { label: azure-az-500, url: /azure-az-500/ }
---

{% include fs-path.html path=page.fs_path %}

<div class="collection-heading">
  <p class="collection-heading__eyebrow">AZURE / SECURITY ENGINEER</p>
  <p class="collection-heading__description">Microsoft Entra ID, 네트워크, 컴퓨팅, 데이터 보호와 보안 운영을 AZ-500 관점에서 연결한 노트입니다.</p>
</div>

{% include cloud-tabs.html active='azure-az-500' %}

{% assign filtered_posts = site.posts | where_exp: "p", "p.categories contains 'azure-az-500'" | sort: "date" | reverse %}
<div class="collection-section-heading"><h2>Azure AZ-500 notes</h2><span>{{ filtered_posts.size }} entries</span></div>
<div class="post-card-list">
  {% for post in filtered_posts %}{% include post-card.html post=post %}{% endfor %}
</div>
