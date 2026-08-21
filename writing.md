---
layout: page
title: writing
subtitle: Notes, book highlights, anything random on my mind
permalink: /writing/
---

{% assign posts_by_year = site.posts | group_by_exp: "post", "post.date | date: '%Y'" %}
<ul class="post-list">
{%- for year in posts_by_year %}
  <li class="post-list__year">
    <h2 class="post-list__year-heading">{{ year.name }}</h2>
    <ul class="post-list__items">
      {%- for post in year.items %}
      <li class="post-list__item">
        <time class="post-list__date" datetime="{{ post.date | date_to_xmlschema }}">{{ post.date | date: "%b %-d" }}</time>
        <a class="post-list__link" href="{{ post.url | relative_url }}">{{ post.title | escape }}</a>
      </li>
      {%- endfor %}
    </ul>
  </li>
{%- endfor %}
</ul>

<p class="post-list__feed"><a href="{{ '/feed.xml' | relative_url }}">subscribe via rss</a></p>
