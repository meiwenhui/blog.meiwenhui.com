---
layout: default
title: Open Source Projects | Code that Might be Useful to You
---


<section class="post-list">
  <div class="container">

{% for category in site.categories %}
{% if category[0] == "project" %}
    {% for posts in category %}
      {% for post in posts %}
        {% unless post.next %}
          <h2 class="category-title">{{ post.date | date: '%Y' }}</h2>
        {% else %}
          {% capture year %}{{ post.date | date: '%Y' }}{% endcapture %}
          {% capture nyear %}{{ post.next.date | date: '%Y' }}{% endcapture %}
          {% if year != nyear %}
            <h2 class="category-title">{{ post.date | date: '%Y' }}</h2>
          {% endif %}
        {% endunless %}
        <article class="post-item">
          <span class="post-meta date-label">{{ post.date | date: "%b %d" }}</span>
          <div class="article-title"><a class="post-link" href="{{ post.url | prepend: site.baseurl }}">{{ post.title }}</a></div>
        </article>
      {% endfor %}
    {% endfor %}
{% endif %}
{% endfor %}

</div>

</section>
