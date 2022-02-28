# Posts

{% for post in site.posts %}
- {{ page.date | date: "%-d %B %Y" }} [{{ post.title }}]({{ post.url }})
{% endfor %}
