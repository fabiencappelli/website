---
title: Projects
layout: default
permalink: /en/projects/
---

{% assign projects = site.projects_en %}

{% for project in projects %}

  <h2><a href="{{ project.url }}">{{ project.title }}</a></h2>
  <p>{{ project.summary }}</p>
{% endfor %}
