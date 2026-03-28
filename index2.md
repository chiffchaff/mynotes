---
layout: default
title: Site Index - Grouped by Folder
permalink: /
---

<h1>Site Contents – Grouped by Folder</h1>

<p>This page automatically shows all pages organized by their folder structure.</p>

{% assign all_pages = site.pages | sort: 'url' %}

{% comment %}Group pages by their top-level directory{% endcomment %}
{% assign grouped = all_pages | group_by: 'dir' %}

{% for group in grouped %}
  {% assign dir = group.name | strip %}
  
  {% if dir == '' or dir == '/' or dir == '.' %}
    <h2>Root Folder</h2>
  {% else %}
    <h2>{{ dir | replace: '/', ' / ' }}/</h2>
  {% endif %}

  <ul>
    {% for page in group.items %}
      {% unless page.url == '/' or page.url contains '/index.html' or page.url contains '404' %}
        <li>
          <a href="{{ page.url | relative_url }}">
            {{ page.title | default: page.name | default: page.url }}
          </a>
          {% if page.excerpt %}
            <br><small>{{ page.excerpt | strip_html | truncate: 120 }}</small>
          {% endif %}
        </li>
      {% endunless %}
    {% endfor %}
  </ul>
{% endfor %}

<!-- Optional: Separate section for blog posts -->
<h2>Blog Posts</h2>
<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
      <small> — {{ post.date | date: "%Y-%m-%d" }}</small>
    </li>
  {% endfor %}
</ul>
