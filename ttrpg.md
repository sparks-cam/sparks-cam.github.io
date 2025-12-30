---
layout: page
title: TTRPG
permalink: /ttrpg/
description: Characters, campaigns, rules, and reviews.
---

<p class="lead">Pick a section:</p>

<div class="columns columns-break">
  {% for item in site.data.ttrpg_hub %}
  <div class="column column-1-2">
    <article class="project-card">
      <a href="{{ item.url | relative_url }}" class="no-hover no-print-link flip-project" tabindex="-1">
        <div class="project-card-img aspect-ratio sixteen-nine">
          {% if item.image %}
            {% include_cached components/hy-img.html img=item.image alt=item.title sizes="(min-width: 54em) 22rem, 42rem" width=864 height=486 %}
          {% endif %}
        </div>
      </a>
      <h3 class="project-card-title flip-project-title">
        <a href="{{ item.url | relative_url }}">{{ item.title }}</a>
      </h3>
      <p class="project-card-text">{{ item.description }}</p>
    </article>
  </div>
  {% endfor %}
</div>

<hr/>

<p>
  Want everything? Browse <a href="{{ '/ttrpg/all/' | relative_url }}">all TTRPG posts</a>.
</p>
