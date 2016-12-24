---
layout: default
---

<div width="100%">
  <div class="wrapper context_text">
      <div style="font-weight: bold;"> Posts </div>
      <br/>
      	<ul>
			{% for post in site.posts %}
			<li>
				<a href="{{ post.url }}">{{ post.title }}</a>
			</li>
			{% endfor %}
		</ul>
  </div>
</div>