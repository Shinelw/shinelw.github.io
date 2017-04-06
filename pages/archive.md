---
layout: page
title: "Archive"
description: "Here are all the articles archived."
header-img: "img/home-bg.jpg"
---

### Blogs
<hr>

{% for post in site.posts %}
<div class="post-preview">
    <font color="blue">[{{ post.date | date: "%B %-d, %Y" }}]  </font>
     <a target="_blank" href="{{ post.url | prepend: site.baseurl }}"> {{ post.title }}  </a>
</div>
<hr>
{% endfor %}
