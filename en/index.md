---
layout: default
title: Home
lang: en
description: Portfolio, projects and technical journal.
hide_toc: true
---

Hello, I'm Fabien Cappelli.

Cyber-gutter punk. Former paleographer. PhD in linguistics. Trained in mathematics, data science, and AI engineering.

I work today at the intersection of technical implementation, documentation, applied AI, and real-world systems.

## Currently

- building _Robie_, a local voice assistant running on Raspberry Pi
- designing technical documentation, workflows, and information systems
- learning agentic AI for supply chain use cases: document intelligence, traceability, and decision automation

## Featured Projects

{% assign featured_projects = site.projects_en | where: "featured", true | sort: "order" %}

<div class="projects-grid">
  {% for project in featured_projects limit: 2 %}
    <div class="card">
      <h3><a href="{{ project.url | relative_url }}">{{ project.title }}</a></h3>
      <p>{{ project.summary }}</p>

      {% if project.stack %}
      <div class="tags">
        {% for tech in project.stack limit: 4 %}
          <span class="tag">{{ tech }}</span>
        {% endfor %}
      </div>
      {% endif %}
    </div>

{% endfor %}

</div>

## Latest Posts

{% assign latest_posts = site.posts | where: "lang", "en" | sort: "date" | reverse %}

<div class="posts-grid">
  {% for post in latest_posts limit: 2 %}
    <div class="card">
      <h3><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h3>
      {% if post.excerpt %}
        <p>{{ post.excerpt | strip_html | truncate: 140 }}</p>
      {% elsif post.summary %}
        <p>{{ post.summary }}</p>
      {% endif %}
    </div>
  {% endfor %}
</div>
