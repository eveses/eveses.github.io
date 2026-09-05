---
layout: single
title: "Home"
permalink: /
author_profile: false
classes: fs-title-only home-dashboard

fs_path:
  - { label: home, url: / }
---

{% include fs-path.html path=page.fs_path %}

{% assign htb_posts = site.posts | where_exp: "p", "p.categories contains 'hackthebox'" %}
{% assign thm_posts = site.posts | where_exp: "p", "p.categories contains 'tryhackme'" %}
{% assign cheat_posts = site.posts | where_exp: "p", "p.categories contains 'cheatsheet'" %}
{% assign saa_posts = site.posts | where_exp: "p", "p.categories contains 'aws-saa'" %}
{% assign scs_posts = site.posts | where_exp: "p", "p.categories contains 'aws-scs'" %}
{% assign azure_posts = site.posts | where_exp: "p", "p.categories contains 'azure-az-500'" %}
{% assign offensive_count = htb_posts.size | plus: thm_posts.size | plus: cheat_posts.size %}
{% assign cloud_count = saa_posts.size | plus: scs_posts.size | plus: azure_posts.size %}

<section class="home-overview" aria-labelledby="home-title">
  <div class="home-overview__copy">
    <p class="home-overview__eyebrow">SECURITY STUDY ARCHIVE</p>
    <h1 id="home-title">공격 보안과 클라우드를<br>직접 실습하며 기록합니다.</h1>
    <p>실습 과정, 자주 쓰는 명령, 클라우드 아키텍처와 보안 설계 포인트를 다시 찾아보기 쉬운 형태로 정리한 개인 노트입니다.</p>
  </div>

  <dl class="home-overview__stats" aria-label="Archive statistics">
    <div><dt>{{ site.posts.size }}</dt><dd>Total notes</dd></div>
    <div><dt>{{ offensive_count }}</dt><dd>Offensive</dd></div>
    <div><dt>{{ cloud_count }}</dt><dd>Cloud</dd></div>
  </dl>
</section>

<section class="home-domains" aria-label="Main collections">
  <article class="home-domain-card home-domain-card--offensive">
    <header>
      <div>
        <span class="home-domain-card__index">01 / OFFENSIVE</span>
        <h2>Offensive Security</h2>
      </div>
      <strong>{{ offensive_count }}</strong>
    </header>
    <p>Hack The Box와 TryHackMe 실습 흐름, 침투 테스트 과정에서 반복해서 사용하는 명령과 공격 기법을 정리합니다.</p>
    <nav aria-label="Offensive categories">
      <a href="{{ '/hackthebox/' | relative_url }}">Hack The Box <span>{{ htb_posts.size }}</span></a>
      <a href="{{ '/tryhackme/' | relative_url }}">TryHackMe <span>{{ thm_posts.size }}</span></a>
      <a href="{{ '/cheatsheet/' | relative_url }}">Cheatsheet <span>{{ cheat_posts.size }}</span></a>
    </nav>
    <a class="home-domain-card__open" href="{{ '/offensive/' | relative_url }}">Explore offensive notes <span>→</span></a>
  </article>

  <article class="home-domain-card home-domain-card--cloud">
    <header>
      <div>
        <span class="home-domain-card__index">02 / CLOUD</span>
        <h2>Cloud Security</h2>
      </div>
      <strong>{{ cloud_count }}</strong>
    </header>
    <p>AWS와 Azure 서비스를 단순 암기하지 않고, 가용성·확장성·권한·데이터 보호 관점에서 비교하고 연결합니다.</p>
    <nav aria-label="Cloud categories">
      <a href="{{ '/aws-saa/' | relative_url }}">AWS SAA <span>{{ saa_posts.size }}</span></a>
      <a href="{{ '/aws-scs/' | relative_url }}">AWS Security <span>{{ scs_posts.size }}</span></a>
      <a href="{{ '/azure-az-500/' | relative_url }}">Azure AZ-500 <span>{{ azure_posts.size }}</span></a>
    </nav>
    <a class="home-domain-card__open" href="{{ '/cloud/' | relative_url }}">Explore cloud notes <span>→</span></a>
  </article>
</section>

{% assign recent_posts = site.posts | sort: "date" | reverse %}
<section class="home-recent" aria-labelledby="recent-title">
  <div class="collection-section-heading">
    <h2 id="recent-title">Latest notes</h2>
    <span>Latest 6 notes</span>
  </div>
  <div class="post-card-list">
    {% for post in recent_posts limit: 6 %}{% include post-card.html post=post %}{% endfor %}
  </div>
</section>

<aside class="home-practice-note">
  <strong>Practice responsibly.</strong>
  <span>공격 기법은 허가받은 실습 환경에서만 사용하고, 클라우드 정보는 적용 전 최신 공식 문서를 다시 확인합니다.</span>
</aside>
