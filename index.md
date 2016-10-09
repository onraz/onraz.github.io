---
layout: default
---
<div>
{% for post in site.posts %}
      <div>
        <p>
          <a href="{{ post.url }}"><h2>{{ post.title }}</h2></a>
        </p>
        <p>
          {{ post.excerpt }}
        </p>
      </div>
{% endfor %}
</div>
