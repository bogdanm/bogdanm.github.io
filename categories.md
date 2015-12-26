---
layout: page
title: Categories
---

{% for cat in site.categories %}
  * [{{ cat[0] }}](#{{ cat[0] }})
{% endfor %}

***

{% for cat in site.categories %}
# {{ cat[0] }}
{% for post in cat[1] %}
  * {{ post.date | date_to_string }} &raquo; [ {{ post.title }} ]({{ post.url }})
{% endfor %}
{% unless forloop.last %}
<hr>
{% endunless %}
{% endfor %}

***
