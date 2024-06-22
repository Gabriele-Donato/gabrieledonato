---
layout: default
title: Home
image: /assets/images/experiment_on_bird_1768.jpg
---

<h1>Posts</h1>

<div class="post-list">
  {% for post in site.posts %}
    <div class="post-item">
      <h2><a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a></h2>
      <p>{{ post.excerpt | strip_html | truncatewords: 50 }}</p>
    </div>
  {% endfor %}
</div>

