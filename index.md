---
layout: single
title: "Home"
permalink: /
author_profile: false
classes: fs-title-only

# filesystem-style path (kept as-is: home)
fs_path:
  - { label: home, url: / }
---

{% include fs-path.html path=page.fs_path %}

> Hello — this is a small blog where I document what I’m learning in **Offensive Security** and while preparing for the **OSCP**.

You’ll mainly find [Hack The Box]({{ '/hackthebox/' | relative_url }}) and [TryHackMe]({{ '/tryhackme/' | relative_url }}) write-ups, plus a practical [cheatsheet]({{ '/cheatsheet/' | relative_url }}) of commands and configs I actually use. The focus is on **hands-on**, **reproducible** notes over polish.

---

### Notes
- Practice only on **authorized targets**.
- Posts can become **outdated or incorrect** — please double-check with current sources.