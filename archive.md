---
layout: page
title: Archive
---

{% for tag in site.tags %}
<div class="archive-group">
  <h3>{{ tag[0] }}</h3>
  <ul>
    {% for post in tag[1] %}
    <li>
      <time style="font-size:0.75rem;font-weight:500;letter-spacing:0.06em;text-transform:uppercase;color:#6B6B6B;min-width:96px;flex-shrink:0;">{{ post.date | date: "%b %-d, %Y" }}</time>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
    {% endfor %}
  </ul>
</div>
{% endfor %}
