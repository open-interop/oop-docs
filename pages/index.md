---
layout: page
title: Welcome
permalink: /
---

# Open Interop Documentation

[Open Interop](https://openinterop.org) the open-source interoperability middleware software. See the [respository]({{ site.repo }}) for more details.

The software enables data collection from varied sources, translation and manipulation of data, and forwarding on to connected endpoints.

Released under the GNU AGPLv3 open source license to ensure that anyone, anywhere can download and use the software and the only requirement is to share contributions back into the community.


## Getting started

Please see the links in the sidebar for information on how to install and use the system.

<div class="section-index">
    <hr class="panel-line">
    {% for post in site.docs  %}        
    <div class="entry">
    <h5><a href="{{ post.url | prepend: site.baseurl }}">{{ post.title }}</a></h5>
    <p>{{ post.description }}</p>
    </div>{% endfor %}
</div>
