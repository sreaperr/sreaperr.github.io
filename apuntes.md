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

{% for n in topic_group.items %}- [{{ n.title }}]({{ n.url | relative_url }})
{% endfor %}

{% endfor %}
{% endif %}
