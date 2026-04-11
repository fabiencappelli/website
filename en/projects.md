---
title: Projects
layout: default
permalink: /en/projects/
hide_toc: true
---

{% assign projects = site.projects_en | sort: "order" %}

<div class="projects-grid">
  {% for project in projects %}
    <div class="card">
      <h2><a href="{{ project.url | relative_url }}">{{ project.title }}</a></h2>
      <p>{{ project.summary }}</p>

      {% if project.stack %}
        <div class="tags">
          {% for tech in project.stack %}
            <span class="tag">{{ tech }}</span>
          {% endfor %}
        </div>
      {% endif %}
    </div>

{% endfor %}

</div>
