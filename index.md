---
layout: default
title: Site Index
permalink: /
---

<h1>Site Contents – Grouped by Folder</h1>

<p>This page automatically lists all pages organized by their actual folder structure.</p>

{% assign all_pages = site.pages | sort: 'url' %}

{% comment %}Clean grouping by directory - this fixes the weird spacing and extra slashes{% endcomment %}
{% assign grouped_pages = all_pages | group_by: 'dir' | sort: 'name' %}

{% for group in grouped_pages %}
  {% assign dir_path = group.name | strip | remove: '.' | remove: '/' | split: '/' | join: ' / ' %}

  {% if dir_path == '' or dir_path == blank %}
    <h2>📁 Root Folder</h2>
  {% else %}
    <h2>📁 / {{ dir_path }} /</h2>
  {% endif %}

  <ul>
    {% for page in group.items | sort: 'url' %}
      {% unless page.url == '/' or page.url contains '/index.html' or page.url contains '404.html' %}
        <li>
          <a href="{{ page.url | relative_url }}">
            {{ page.title | default: page.name | default: page.url | remove: '.html' | remove: '.md' }}
          </a>
          {% if page.excerpt %}
            <br><small>{{ page.excerpt | strip_html | truncate: 120 }}</small>
          {% endif %}
        </li>
      {% endunless %}
    {% endfor %}
  </ul>
{% endfor %}

<h2>📝 Blog Posts</h2>
<ul>
  {% for post in site.posts | sort: 'date' reversed %}
    <li>
      <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
      <small> — {{ post.date | date: "%Y-%m-%d" }}</small>
    </li>
  {% endfor %}
</ul>
