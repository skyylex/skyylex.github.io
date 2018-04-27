---
layout: default
---
<div width="100%">
  <div class="wrapper context_text">
  	<div style="font-weight: bold;"> Tags </div>
  	<br>
		{% capture temptags %}
			{% for tag in site.tags %}
				{{ tag[1].size | plus: 1000 }}#{{ tag[0] }}#{{ tag[1].size }}
			{% endfor %}
		{% endcapture %}

		{% assign sortedtemptags = temptags | split:' ' | sort | reverse %}

		{% for temptag in sortedtemptags %}
			{% assign tagitems = temptag | split: '#' %}
			{% capture tagname %}{{ tagitems[1] }}{% endcapture %}
				<a href="/tag/{{ tagname }}"><code class="highligher-rouge"><nobr>{{ tagname }}</nobr></code></a>
		{% endfor %}
	</div>
</div>
<br>
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