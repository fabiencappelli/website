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
  {% for project in featured_projects limit: 6 %}
    <a href="{{ project.url | relative_url }}" class="card card-link">
      <h3>{{ project.title }}</h3>
      <p>{{ project.summary }}</p>

      {% if project.stack %}
      <div class="tags">
        {% for tech in project.stack limit: 6 %}
          <span class="tag">{{ tech }}</span>
        {% endfor %}
      </div>
      {% endif %}
    </a>

{% endfor %}

</div>

## Latest Posts

{% assign latest_posts = site.posts | where: "lang", "en" | sort: "date" | reverse %}

<div class="posts-grid">
  {% for post in latest_posts limit: 2 %}
    <a href="{{ post.url | relative_url }}" class="card card-link">
      <h3>{{ post.title }}</h3>
      {% if post.excerpt %}
        <p>{{ post.excerpt | strip_html | truncate: 140 }}</p>
      {% elsif post.summary %}
        <p>{{ post.summary }}</p>
      {% endif %}
    </a>
  {% endfor %}
</div>
