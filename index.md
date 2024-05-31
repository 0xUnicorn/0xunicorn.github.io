---
layout: default
title: Blog
---

<div class="posts">
  {% for post in site.posts %}
  <article class="post">
    <h1><a href="{{ post.url }}">{{ post.title }}</a></h1>
    <div class="post-metadata">
    <p><small><b>Date: </b><em>{{ post.date | date: "%-d %b %Y %H:%M" }}</em></small></p>
    {% if post.tags %}
    <p><small><b>Tags: </b><em>{{ post.tags | join: "</em>, <em>" }}</em></small></p>
    {% endif %}
    </div>
    <div class="entry">
      {{ post.excerpt }}
    </div>

    <a href="{{ post.url }}" class="read-more">Read More</a>
  </article>
  <hr/>
  {% endfor %}
</div>
