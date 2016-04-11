---
layout: page
title: Tags 
lang: fr
ref: tag
---

<div class="page-content wc-container">
	<div class="post">
		<h1>Tags</h1>  
		<ul>
			{% for tag in site.tags %}
			<li><a href="{{ site.baseurl  | prepend:site.url }}/tag/{{ tag[0] }}">{{ tag[0] }}</a></li>
			{% endfor %}
		</ul>
	</div>
</div>
