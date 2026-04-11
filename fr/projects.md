---
title: Projets
layout: default
permalink: /fr/projects/
---

{% assign projects = site.projects_fr %}

{% for project in projects %}

  <h2><a href="{{ project.url }}">{{ project.title }}</a></h2>
  <p>{{ project.summary }}</p>
{% endfor %}
