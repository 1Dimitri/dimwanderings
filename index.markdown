---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: home
---

{% for post in site.posts %}
	<article class="post">
		<header class="post-header">
			<span class="post-meta">
				<time datetime="{{post.date | date_to_xmlschema}}">
					{{post.date | date_to_long_string}}
				</time>
			</span>
			<h2 class="post-title">
				<a href="{{site.url}}{{post.url}}">{{post.title}}</a>
			</h2>
		</header>
		<section class="post-excerpt">
			<p>{{post.excerpt}}&hellip;</p>
		</section>
		<footer class="tags">
			<span class="post-meta">
				{% for category in post.categories %}
					{% if forloop.index > 1 %}, {% endif%}
					{{category}}
				{% endfor %}
			</span>
		</footer>
	</article>
{% endfor %}
