---
title: Blog
layout: default
permalink: /en/blog/
hide_toc: true
---

{% assign posts_en = site.posts | where: "lang", "en" | sort: "date" | reverse %}

{% assign categories_list = "" | split: "" %}
{% for post in posts_en %}
{% if post.categories %}
{% for category in post.categories %}
{% assign categories_list = categories_list | push: category %}
{% endfor %}
{% endif %}
{% endfor %}
{% assign categories_list = categories_list | uniq | sort %}

<section class="page-intro">
  <p>
    Articles, project notes, feedback, and technical experiments.
  </p>
</section>

{% if categories_list.size > 0 %}

<section class="blog-filters">
  <p class="blog-filters__label">Filter by category</p>
  <div class="blog-filters__tags">
    <button class="tag tag-filter is-active" type="button" data-filter="all">
      All
    </button>

    {% for category in categories_list %}
      <button
        class="tag tag-filter"
        type="button"
        data-filter="{{ category | slugify }}"
      >
        {{ category }}
      </button>
    {% endfor %}

  </div>
</section>
{% endif %}

<div class="projects-grid blog-grid">
  {% for post in posts_en %}
    {% assign post_categories_slug = "" %}
    {% if post.categories %}
      {% for category in post.categories %}
        {% assign category_slug = category | slugify %}
        {% assign post_categories_slug = post_categories_slug | append: category_slug %}
        {% unless forloop.last %}
          {% assign post_categories_slug = post_categories_slug | append: " " %}
        {% endunless %}
      {% endfor %}
    {% endif %}

    <a
      href="{{ post.url | relative_url }}"
      class="card card-link blog-card"
      data-categories="{{ post_categories_slug }}"
    >
      <h3>{{ post.title }}</h3>

      {% if post.summary %}
        <p>{{ post.summary }}</p>
      {% elsif post.excerpt %}
        <p>{{ post.excerpt | strip_html | truncate: 160 }}</p>
      {% endif %}

      <div class="tags">
        {% if post.categories %}
          {% for category in post.categories %}
            <span class="tag">{{ category }}</span>
          {% endfor %}
        {% endif %}
      </div>
    </a>

{% endfor %}

</div>

<script>
  document.addEventListener("DOMContentLoaded", function () {
    const filterButtons = document.querySelectorAll(".tag-filter");
    const cards = document.querySelectorAll(".blog-card");

    filterButtons.forEach((button) => {
      button.addEventListener("click", function () {
        const filter = this.dataset.filter;

        filterButtons.forEach((btn) => btn.classList.remove("is-active"));
        this.classList.add("is-active");

        cards.forEach((card) => {
          const categories = card.dataset.categories
            ? card.dataset.categories.split(" ")
            : [];

          if (filter === "all" || categories.includes(filter)) {
            card.style.display = "";
          } else {
            card.style.display = "none";
          }
        });
      });
    });
  });
</script>
