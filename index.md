---
layout: page
title: Crazy Coder
tagline: I'm crazy, but not always!
---
{% include JB/setup %}


## Latest Posts

<ul class="posts">

  {% for post in site.posts limit:10 %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

For all the articles, click [here]({{BASE_PATH}}/archive.html)

## To-Do

This page is still under construction. If you'd like to be added as a contributor, [please fork](http://github.com/JieHust)!



