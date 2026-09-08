---
layout: default
title: Writeups
permalink: /writeups/
---

<div class="stat-grid">
  <div class="stat-tile stat-tile--total">
    <span class="stat-label">Total</span>
    <span class="stat-value">{{ site.writeups | size }}</span>
  </div>
{% assign by_source_count = site.writeups | group_by: "source" %}{% for source_group in by_source_count %}{% assign by_difficulty_count = source_group.items | group_by: "difficulty" %}{% for diff_group in by_difficulty_count %}  <div class="stat-tile">
    <span class="stat-label">{{ source_group.name }}</span>
    <span class="stat-value">{{ diff_group.items | size }}</span>
    <span class="stat-sub">{{ diff_group.name }}</span>
  </div>
{% endfor %}{% endfor %}</div>

{% assign by_source = site.writeups | group_by: "source" %}
{% for source_group in by_source %}
## {{ source_group.name }}

{% assign by_difficulty = source_group.items | group_by: "difficulty" %}
{% for diff_group in by_difficulty %}
### {{ diff_group.name }}

<div class="writeup-grid">
{% for w in diff_group.items %}  <a href="{{ w.url | relative_url }}" class="writeup-card">
    <span class="writeup-card-title">{{ w.title | split: " - " | last }}</span>
    <span class="writeup-card-meta">{{ source_group.name }} · {{ diff_group.name }}</span>
  </a>
{% endfor %}</div>

{% endfor %}
{% endfor %}
