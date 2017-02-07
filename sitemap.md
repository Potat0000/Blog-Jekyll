---
layout: null
---
<?xml version="1.0" encoding="UTF-8"?>
<sitemapindex xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <sitemap>
    <loc>http://www.gyj1109.tk/</loc>
    <lastmod>{{ site.time | date_to_xmlschema }}</lastmod>
  </sitemap>
  {% for post in site.posts %}
    <sitemap>
      <loc>{{ site.url }}{{ post.url }}</loc>
      {% if post.lastmod == null %}
        <lastmod>{{ post.date | date_to_xmlschema }}</lastmod>
      {% else %}
        <lastmod>{{ post.lastmod | date_to_xmlschema }}</lastmod>
      {% endif %}
    </sitemap>
  {% endfor %}
  {% for page in site.pages %}
    {% if page.sitemap != null and page.sitemap != empty %}
      <sitemap>
        <loc>{{ site.url }}{{ page.url }}</loc>
        <lastmod>{{ page.sitemap.lastmod | date_to_xmlschema }}</lastmod>
       </sitemap>
    {% endif %}
  {% endfor %}
</sitemapindex>
