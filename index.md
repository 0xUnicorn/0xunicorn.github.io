---
layout: default
title: Blog
---

<div class="posts">
  {% for post in site.posts %}
  <article class="post">
    <h1><a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a></h1>
    <small>Date: <em>{{ post.date | date: "%-d %b %Y %H:%M" }}</em></small>
    {% if post.tags %}
    <small>| Tags: <em>{{ post.tags | join: "</em> - <em>" }}</em></small>
    {% endif %}
    <div class="entry">
      {{ post.excerpt }}
    </div>

    <a href="{{ site.baseurl }}{{ post.url }}" class="read-more">Read More</a>
  </article>
  <hr/>
  {% endfor %}
</div>
