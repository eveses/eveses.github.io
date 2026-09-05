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

<div class="collection-heading">
  <p class="collection-heading__eyebrow">CLOUD / ARCHITECTURE &amp; SECURITY</p>
  <p class="collection-heading__description">AWS와 Azure의 핵심 서비스를 설계 관점에서 정리한 노트입니다. 단순 암기보다 서비스 선택 기준, 보안 경계, 장애 대응을 빠르게 비교할 수 있도록 구성했습니다.</p>
</div>

{% include cloud-tabs.html active='all' %}

{% assign saa_posts = site.posts | where_exp: "p", "p.categories contains 'aws-saa'" %}
{% assign scs_posts = site.posts | where_exp: "p", "p.categories contains 'aws-scs'" %}
{% assign azure_posts = site.posts | where_exp: "p", "p.categories contains 'azure-az-500'" %}
{% assign filtered_posts = saa_posts | concat: scs_posts | concat: azure_posts | sort: "date" | reverse %}
<div class="collection-section-heading">
  <h2>All cloud notes</h2>
  <span>{{ filtered_posts.size }} entries</span>
</div>
<div class="post-card-list">
  {% for post in filtered_posts %}{% include post-card.html post=post %}{% endfor %}
</div>
