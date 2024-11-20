---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: default
---

<html>
  <head>
    <title>Tag: #{{page.tag}}</title>
  </head>
  <body>
    {% capture tags %}
        {% for tag in site.tags %}
            {{ tag[0] }}
        {% endfor %}
    {% endcapture %}
    {% assign sortedtags = tags | split:' ' | sort %}
    
    {% for tag in sortedtags %}
        <h3 id="{{ tag }}">Tag: {{ tag }}</h3>
        <ul>
        {% for post in site.tags[tag] %}
        <li><a href="{{ post.url }}">{{ post.title }}</a></li>
        {% endfor %}
        </ul>
    {% endfor %}
  </body>
</html>