---
layout: page
title: About 
permalink: blogs/
description: Some Description
date: 2017-05-17 09:17:33 +05:30
tags: "some tags here"
---

<div id="home">
  <h1>Blog Posts</h1>
  <ul class="posts">
    {% for post in site.posts %}
      <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a>
        <br />
        <br />
      </li>
    {% endfor %}
  </ul>
</div>


