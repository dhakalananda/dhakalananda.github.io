---
layout: collection
permalink: /
author_profile: true
---

{% assign all_content = site.blogs | concat: site.advisories | concat: site.misc %}
{% assign all_content = all_content | sort: "date" | reverse %}


{% for item in all_content %}
  <article class="blog-preview" style="line-height: 1.4; margin-bottom: 0.75rem;">
    <small>{{ item.date | date: "%B %d, %Y" }}</small>
    <h2 style="margin: 0.25rem 0;">
      <a href="{{ item.url }}">{{ item.title }}</a>
    </h2>
    {% if item.description %}
      <small style="margin: 0;">{{ item.description }}</small>
    {% endif %}
  </article>
  <hr>
{% endfor %}

