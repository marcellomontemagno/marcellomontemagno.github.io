---
layout: post
title: Categories
---

{% assign rawcats = "" %}

{% for post in site.posts %}
	{% assign tcats = post.category | join:'|' | append:'|' %}
	{% assign rawcats = rawcats | append:tcats %}
{% endfor %}

{% assign rawcats = rawcats | split:'|' | sort %}

{% assign cats = "" %}

{% for cat in rawcats %}
	{% if cat != "" %}
		{% if cats == "" %}
			{% assign cats = cat | split:'|' %}
		{% endif %}
		{% unless cats contains cat %}
			{% assign cats = cats | join:'|' | append:'|' | append:cat | split:'|' %}
		{% endunless %}
	{% endif %}
{% endfor %}


<div class="posts">

{% for ct in cats %}
	
  <h2 id="{{ ct | slugify }}">{{ ct }}</h2>

  {% for post in site.posts %}
	  {% if post.category contains ct %}
	  
		  <a href="{{ post.url }}">
			{{ post.title }}
		  </a>
		  <small> - {{ post.date | date_to_string }}</small>
		  <br/>
		  
	  {% endif %}
  {% endfor %}

{% endfor %}

</div>
