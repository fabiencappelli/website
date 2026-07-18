---
layout: default
title: Accueil
lang: fr
description: Implémentation technique, documentation structurée, automatisation et IA appliquée — portfolio et journal de Fabien Cappelli.
hide_toc: true
---

Bonjour, je suis Fabien Cappelli.

Cyberpunk à chien. Ancien paléographe. Docteur en linguistique. Formé aux mathématiques, à la data science et à l’ingénierie de l’IA.

Je conçois et mets en œuvre des systèmes techniques qui doivent survivre au contact du réel : intégrations SaaS, API, modèles de données, documentation structurée et automatisations assistées par l’IA.

Mon travail consiste souvent à partir d’un besoin encore flou, à en comprendre les contraintes métier et réglementaires, puis à le transformer en workflows, outils et connaissances réellement utilisables.

Je m’intéresse moins aux démonstrations spectaculaires qu’aux systèmes explicables, maintenables et utiles aux personnes qui devront s’en servir.

## En ce moment

- je construis _Robie_, un assistant vocal local sur Raspberry Pi combinant reconnaissance vocale, modèle de langage et synthèse vocale
- je conçois de bout en bout un écosystème documentaire MkDocs, en remplacement d’un guide PDF statique, pour un usage humain comme pour la recherche assistée par l’IA
- je construis des outils qui exploitent l’IA pour automatiser des tâches concrètes : interroger le contenu d’une vidéo YouTube avec un modèle local, ou transformer un export Jira en vidéo de release

## Projets mis en avant

{% assign featured_projects = site.projects_fr | where: "featured", true | sort: "order" %}

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

## Derniers articles

{% assign latest_posts = site.posts | where: "lang", "fr" | sort: "date" | reverse %}

<div class="posts-grid">
  {% for post in latest_posts limit: 2 %}
    <a href="{{ post.url | relative_url }}" class="card card-link">
      <h3>{{ post.title }}</h3>
      {% if post.summary %}
        <p>{{ post.summary }}</p>
      {% elsif post.excerpt %}
        <p>{{ post.excerpt | strip_html | truncate: 140 }}</p>
      {% endif %}
    </a>
  {% endfor %}
</div>
