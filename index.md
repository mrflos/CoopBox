---
layout: page
title: Hello World!
tagline: Supporting tagline
---
{% include JB/setup %}

CoopBox
=======

## Virtual server for collaboration.

Script to create a VirtualBox image that contains a Debian Linux Distribution with a light web-server with all needed technologies for modern web-design (Nginx, PHP, MySQL, NodeJS/npm, Ruby, Python, git) with pre-installed open source (GPL) software for collaboration :
 - [Yeswiki](http://yeswiki.net)
 - [Etherpad](http://etherpad.org)
 - [Scrumblr](https://github.com/aliasaria/scrumblr)
 - [Sparkleshare](http://sparkleshare.org)

It can be used on any operating system that runs VirtualBox (Windows , MacOS, Linux)

It was designed in order to use internet collaboration tools, on local network without global internet access.

It can also be used as a virtual environment for Web-development.

News
====
<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>