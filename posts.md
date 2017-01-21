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
				<a href="{{ post.url }}">
					{{ post.date | date: "%m/%d/%Y" }}
					<b> &ensp; ||| &ensp; {{ post.title | replace: '_', ' ' }} </b>
				</a>

			</li>
			{% endfor %}
		</ul>
  </div>
</div>