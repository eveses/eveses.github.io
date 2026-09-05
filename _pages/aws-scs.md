---
layout: single
title: "[AWS SCS]"
permalink: /aws-scs/
author_profile: true
classes: fs-title-only

fs_path:
  - { label: home,    url: / }
  - { label: cloud,        url: /cloud/ }
  - { label: aws-scs, url: /aws-scs/ }
---

{% include fs-path.html path=page.fs_path %}

<div class="collection-heading">
  <p class="collection-heading__eyebrow">AWS / SECURITY SPECIALTY</p>
  <p class="collection-heading__description">탐지, 사고 대응, IAM, 데이터 보호와 거버넌스를 실제 보안 통제 흐름으로 연결한 AWS 보안 노트입니다.</p>
</div>

{% include cloud-tabs.html active='aws-scs' %}

{% assign filtered_posts = site.posts | where_exp: "p", "p.categories contains 'aws-scs'" | sort: "date" | reverse %}
<div class="collection-section-heading"><h2>AWS Security notes</h2><span>{{ filtered_posts.size }} entries</span></div>
<div class="post-card-list">
  {% for post in filtered_posts %}{% include post-card.html post=post %}{% endfor %}
</div>
