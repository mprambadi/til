---
layout: default
---

# Today I Learned

Short notes on things I learned today.

## Posts

{% for post in site.posts %}
- [{{ post.title }}]({{ post.url }}) — {{ post.date | date: "%B %d, %Y" }}
{% endfor %}
