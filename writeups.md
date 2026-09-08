---
layout: default
title: Writeups
permalink: /writeups/
---

## Resumen

**Total: {{ site.writeups | size }} writeups**

| Plataforma | Dificultad | Writeups |
|---|---|---|
{% assign by_source_count = site.writeups | group_by: "source" %}{% for source_group in by_source_count %}{% assign by_difficulty_count = source_group.items | group_by: "difficulty" %}{% for diff_group in by_difficulty_count %}| {{ source_group.name }} | {{ diff_group.name }} | {{ diff_group.items | size }} |
{% endfor %}{% endfor %}

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
