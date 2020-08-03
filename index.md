---
title: Home
---

## Public Repository
Here are list of public repositories.

Repository | Description | Build Status
---------- | ----------- | ------------
[gatecoinapi4j]({{ site.url }}/gatecoinapi4j) | This is a java implementation of [Gatecoin](https://gatecoin.com) api | [![Build Status](https://travis-ci.org/micwan88/gatecoinapi4j.svg?branch=master)](https://travis-ci.org/micwan88/gatecoinapi4j)
[d3js-neo4j-example](https://github.com/micwan88/d3js-neo4j-example) | This is [D3.js v5](https://d3js.org/) example visualize the result from [Neo4j](https://neo4j.com/). | N/A
[helperclass4j](https://github.com/micwan88/helperclass4j) | This is a java helper class library for the author (Michael Wan) self usage | [![Build Status](https://travis-ci.org/micwan88/helperclass4j.svg?branch=master)](https://travis-ci.org/micwan88/helperclass4j)
[dockerfile-example](https://github.com/micwan88/dockerfile-example) | This repository is for the author (Michael Wan) to store his Dockerfile. | N/A

## Note Posts
Here are list of note posts.
{% for post in site.posts %}
- [{{ post.date | date_to_string }} - {{ post.title }}]({{ post.url }})
{% endfor %}
