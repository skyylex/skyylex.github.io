---
layout: default
---
<div width="100%">
  <div class="wrapper context_text">
  	<div style="font-weight: bold; font-size: 20px;"> Tags </div>
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
      <div style="font-weight: bold; font-size: 20px;"> Posts </div>
{% for post in site.posts %}
		<br>
		<br>
		<hr class="noprint">
		<br>
		<a href="{{ post.url }}" style="font-weight: bold; font-size: 18px;">
		<b> {{ post.title | replace: '_', ' ' }} </b>
		</a>
		<span style="font-weight: bold; font-size: 18px;">{{ post.date | date: "%m/%d/%Y" }}
		</span>
		<br>
		<br>	
		<span style="font-size: 18px;">{{ post.description }}</span>
		<br>
		<br>
		<span>
			{% for tag in post.tags %}
    			{% capture tag_name %}{{ tag }}{% endcapture %}
    			<a href="/tag/{{ tag_name }}"><code class="highligher-rouge"><nobr>{{ tag_name }}</nobr></code>&nbsp;</a>
  			{% endfor %}
		</span>
		{% endfor %}
	</div>
</div>