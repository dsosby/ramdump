---
layout : layout
title : RAM Dump
---

<ul class="posts">
    {% for post in site.posts  limit:5 %}
		<li>
			<div class="idea">
				{% if forloop.first and post.layout == "post" %}
					<h1><a href="{{ post.url }}">{{ post.title }}</a></h1>
					<div class="postdate">{{ post.date | date: "%e %B, %Y"  }}
						<ul>
						{% for tag in post.tags %}
							<li><a href="/tag/{{ tag }}">{{ tag }}</a></li>
						{% endfor %}
						</ul>
					</div>
					{{ post.content }}
				{% else %}
					<h2><a class="postlink" href="{{ post.url }}">{{ post.title }}</a></h2>
					<div class="postdate">{{ post.date | date: "%e %B, %Y"  }}
						<ul>
						{% for tag in post.tags %}
							<li><a href="/tag/{{ tag }}">{{ tag }}</a></li>
						{% endfor %}
						</ul>
					</div>
					{{ post.content }}
				{% endif %}
			</div>
		</li>
    {% endfor %}
</ul>

<h3>OLDER</h3>
<ul class="postArchive">
{% for post in site.posts offset:5 %}
	<li>
		<span class="olderpostdate"> {{ post.date | date: "%d %b"  }} </span> <a class="postlink" href="{{ post.url }}">{{ post.title }}</a>
	</li>
{% endfor %}
</ul>