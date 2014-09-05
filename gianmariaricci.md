---
layout: page
permalink: /about/gianmariaricci/
title: Gian Maria Ricci
tags: [profile, info]
image:
  feature: about-alkampfer.jpg
  credit: Bruno Leonangeli
share: true
---
{% assign author = site.authors_data['gianmariaricci'] %}
<h2>About me</h2>
<div class="entry">
  <ul class="bio-menu pull-right">
    {% if author.email %}<li>
      <a href="mailto:{{ author.email }}"><i class="fa fa-envelope"></i> Email</a>
    </li>{% endif %}
    {% if author.twitter %}<li>
      <a href="http://twitter.com/{{ author.twitter }}"><i class="fa fa-twitter"></i> Twitter</a>
    </li>{% endif %}
    {% if author.facebook %}<li>
      <a href="http://facebook.com/{{ author.facebook }}"><i class="fa fa-facebook"></i> Facebook</a>
    </li>{% endif %}
    {% if author.google_plus %}<li>
      <a href="https://google.com/{{ author.google_plus }}"><i class="fa fa-google-plus"></i> Google+</a>
    </li>{% endif %}
    {% if author.linkedin %}<li>
      <a href="http://linkedin.com/in/{{ author.linkedin }}"><i class="fa fa-linkedin"></i> LinkedIn</a>
    </li>{% endif %}
{% if author.slideshare %}<li>
  <a href="http://www.slideshare.net/{{ author.slideshare }}"><i class="fa fa-slideshare"></i> SlideShare</a>
</li>{% endif %}
    {% if author.github %}<li>
      <a href="http://github.com/{{ author.github }}"><i class="fa fa-github"></i> GitHub</a>
    </li>{% endif %}
    {% if author.stackexchange %}<li>
      <a href="{{ author.stackexchange }}"><i class="fa fa-stack-exchange"></i> Stackexchange</a>
    </li>{% endif %}
    {% if author.instagram %}<li>
      <a href="http://instagram.com/{{ author.instagram }}"><i class="fa fa-instagram"></i> Instagram</a>
    </li>{% endif %}
    {% if author.flickr %}<li>
      <a href="http://www.flickr.com/photos/{{ author.flickr }}"><i class="fa fa-flickr"></i> Flickr</a>
    </li>{% endif %}
    {% if author.tumblr %}<li>
      <a href="http://{{ author.tumblr }}.tumblr.com"><i class="fa fa-tumblr"></i> Tumblr</a>
    </li>{% endif %}
  </ul><!-- /.submenu -->
  <div>
    <p>
    Passionate Coder & Musician & Blogger & Ex D&D player & Manga fan & More
    </p>
    <p>
    Co-founder and staff member of @<strong>dotnet</strong>marche,  @<strong>glv_it</strong>, @<strong>xpug</strong>marche, @<strong>dev</strong>marche. <br/>
    </p>

  </div>
