# Posts

{% for post in site.posts %}
    {% if post.hidden != "true"  %}
- {{ post.date | date: "%-d %B %Y" }} [{{ post.title }}]({{ post.url }})
    {% endif %}
{% endfor %}
