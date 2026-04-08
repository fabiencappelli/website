---
title: Projects
layout: default
permalink: /en/projects/
---

<h1>Projets</h1>

{% assign projects = site.projects_en %}

{% for project in projects %}

  <h2><a href="{{ project.url }}">{{ project.title }}</a></h2>
  <p>{{ project.summary }}</p>
{% endfor %}
