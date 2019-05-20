---
layout: gallery
title: Gallery
permalink: /gallery/
---
<ul>
  {% for image in site.static_files %}
    {% if image.path contains 'gallery' %}
      <a href="{{ site.baseurl }}{{ image.path }}">
        <img src="{{ site.baseurl }}{{ image.path }}" width="200"/>
      </a> 
    {% endif %}
  {% endfor %}
</ul>