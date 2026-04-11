---
layout: default
title: Home
lang: fr
description: Portfolio, projets et journal technique.
hide_toc: true
---

Bonjour, je suis Fabien Cappelli.

Cyberpunk à chien. Ancien paléographe. Docteur en linguistique. Formé aux mathématiques, à la data science et à l’AI engineering.

Je travaille aujourd'hui à l’intersection de l’implémentation technique, de la documentation, de l’IA appliquée et des systèmes concrets.

## En ce moment

- construction de _Robie_, un assistant vocal local sur Raspberry Pi
- conception de documentation technique, workflows et systèmes d’information
- apprentissage de l’IA agentique pour la supply chain : document intelligence, traçabilité et automatisation de décisions

## Projets mis en avant

{% assign featured_projects = site.projects_fr | where: "featured", true | sort: "order" %}

<div class="projects-grid">
  {% for project in featured_projects limit: 2 %}
    <a href="{{ project.url | relative_url }}" class="card card-link">
      <h3>{{ project.title }}</h3>
      <p>{{ project.summary }}</p>

      {% if project.stack %}
      <div class="tags">
        {% for tech in project.stack limit: 4 %}
          <span class="tag">{{ tech }}</span>
        {% endfor %}
      </div>
      {% endif %}
    </a>

{% endfor %}

</div>

## Derniers articles

{% assign latest_posts = site.posts | where: "lang", "fr" | sort: "date" | reverse %}

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
