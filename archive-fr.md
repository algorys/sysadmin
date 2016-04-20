---
layout: page
title: Archive du Blog
lang: fr
ref: archive
---

<div class="page-content wc-container">
	<div class="post">
		<h1>Archives du Blog</h1>
        {% assign posts=site.posts | where:"lang", page.lang %}
		{% for post in posts %}
			{% capture currentyear %}{{post.date | date: "%Y"}}{% endcapture %}
			{% if currentyear != year %}
				{% unless forloop.first %}</ul>{% endunless %}
					<h5>{{ currentyear }}</h5>
					<ul class="posts">
					{% capture year %}{{currentyear}}{% endcapture %} 
				{% endif %}
			<li>[ {{ post.categories[0] }} ] <a href="{{ post.url | prepend: site.baseurl | prepend: site.url}}">{{ post.title }}</a></li>
	{% endfor %}
	</div>
</div>
