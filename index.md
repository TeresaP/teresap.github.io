---
layout: page
title: Welcome!
tagline: 
---
{% include JB/setup %}

	

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
      {{ post.excerpt }}
    </li>
  {% endfor %}
</ul>