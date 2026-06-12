---
layout: default
title: Apuntes
permalink: /apuntes/
---

# Apuntes de ciberseguridad

{% if site.notes.size == 0 %}
_Próximamente._
{% else %}
{% assign by_topic = site.notes | group_by: "topic" %}
{% for topic_group in by_topic %}
## {{ topic_group.name }}

<div class="card-list">
{% for n in topic_group.items %}  <a href="{{ n.url | relative_url }}">{{ n.title }}</a>
{% endfor %}</div>

{% endfor %}
{% endif %}
