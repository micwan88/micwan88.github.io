---
title: Home
---

## Home
Web notes home page

### Public Repository
Here are list of pulbic repositories.
- [gatecoinapi4j]({{ site.url }}/gatecoinapi4j) - This is a java implementation of [Gatecoin](https://gatecoin.com) api [![Build Status](https://travis-ci.org/micwan88/gatecoinapi4j.svg?branch=master)](https://travis-ci.org/micwan88/gatecoinapi4j)

### Note Posts
Here are list of note posts.
{% for post in site.posts %}
- [{{ post.date }} - {{ post.title }}]({{ post.url }})
{% endfor %}
