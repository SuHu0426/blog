---
layout: archive
permalink: /
title: "Latest Posts"
---

<div class="tiles">
{% for post in site.posts %}
   {% include post-grid.html %}
{% endfor %}
</div><!-- /.tiles -->

## Articles

<div class="tiles">
{% for post in site.categories.articles %}
   {% include post-grid.html %}
{% endfor %}
</div><!-- /.tiles -->

## Presentation

<div class="tiles">
{% for post in site.categories.presentation %}
   {% include post-grid.html %}
{% endfor %}
</div><!-- /.tiles -->
