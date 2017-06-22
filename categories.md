---
title: Categories
layout: page
permalink: "/categories.html"
---

{% assign default_paths = site.pages | map: "path" %}
{% assign page_paths = site.header_pages | default: default_paths %}
{% if page_paths %}
  <div>
    {% for path in page_paths %}
      {% assign my_page = site.pages | where: "path", path | first %}
        {% if my_page.type == "category" %}
          {% if my_page.title %}
            <div style="padding: 5px 0px;">
              <a class="page-link" href="{{ my_page.url | relative_url }}">{{ my_page.title | escape }}</a>
            </div>
          {% endif %}
        {% endif %}
    {% endfor %}
  </div>
{% endif %}