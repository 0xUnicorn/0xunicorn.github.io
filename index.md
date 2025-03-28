---
layout: default
title: Blog
---

<div class="posts">

  {% for post in site.posts %}

  <article class="post">
    <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>

    <p>
        <small>Date: <em>{{ post.date | date_to_string }}</em></small><br>
        <small><b>Tags:</b>
        {% if post.tags %}
        <em>{{ post.tags | join: "</em>, <em>" }}</em>
        {% endif %}
        </small>
    </p>

  </article>

  <hr/>

  {% endfor %}

</div>
