# Posts

{% for post in site.posts %}
- {{page.date}} [{{ post.title }}]({{ post.url }})
{% endfor %}
