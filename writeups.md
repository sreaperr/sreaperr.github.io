---
layout: default
title: Writeups
permalink: /writeups/
---

# Writeups

{% assign by_source = site.writeups | group_by: "source" %}
{% for source_group in by_source %}
## {{ source_group.name }}

{% assign by_difficulty = source_group.items | group_by: "difficulty" %}
{% for diff_group in by_difficulty %}
### {{ diff_group.name }}

<div class="card-list">
{% for w in diff_group.items %}  <a href="{{ w.url | relative_url }}">{{ w.title }}</a>
{% endfor %}</div>

{% endfor %}
{% endfor %}
