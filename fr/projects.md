---
title: Projets
layout: default
permalink: /fr/projects/
hide_toc: true
---

{% assign projects = site.projects_fr | sort: "order" %}

<div class="projects-grid">
  {% for project in projects %}
    <a href="{{ project.url | relative_url }}" class="card card-link">
      <h3>{{ project.title }}</h3>
      <p>{{ project.summary }}</p>

      {% if project.stack %}
      <div class="tags">
        {% for tech in project.stack %}
          <span class="tag">{{ tech }}</span>
        {% endfor %}
      </div>
      {% endif %}
    </a>

{% endfor %}

</div>
