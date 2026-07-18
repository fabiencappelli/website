---
layout: default
title: Home
lang: en
description: Technical implementation, structured documentation, automation, and applied AI — the portfolio and technical journal of Fabien Cappelli.
hide_toc: true
---

Hello, I’m Fabien Cappelli.

Cyber-gutter-punk. Former palaeographer. PhD in Linguistics. Trained in mathematics, data science, and AI engineering.

I design and implement technical systems built to survive contact with reality: SaaS integrations, APIs, data models, structured documentation, and AI-assisted automation.

My work often begins with a need that is still loosely defined: understanding its business and regulatory constraints, then turning it into workflows, tools, and knowledge that people can actually use.

I am less interested in flashy demos than in systems that are explainable, maintainable, and useful to the people who will rely on them.

## What I’m working on

- building _Robie_, a local voice assistant on Raspberry Pi combining speech recognition, a language model, and text-to-speech
- designing and building a MkDocs documentation ecosystem from the ground up, replacing a static PDF guide and supporting both human readers and AI-assisted retrieval
- building tools that use AI to automate concrete tasks: asking questions about a YouTube video using its transcript and a local model, or turning a Jira export into a narrated release video

## Featured projects

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

## Latest posts

{% assign latest_posts = site.posts | where: "lang", "en" | sort: "date" | reverse %}

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
