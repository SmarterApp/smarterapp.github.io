---
title:  TDS Configuration Tasks
permalink: "/deployment/configuration/index.html"
categories: ["config_index"]
---
this is the configuration index page

# Overview
This page hosts instructions on how to configure various options within the TDS Student application.

## Conventions Used in the Configuration Tasks Instructions
The following conventions are used throughout the checklist:

* [*some text*{: style="color: red"}] indicates text that should be replaced by the user.  When replacing text, the surrounding square brackets should be omitted.<br>For example,

> <span style="font-family: 'Lucida Console', Monaco, monospace">sudo git clone ssh://git@github.com:*[your github user name]*{: style="color: red"}/IM_OpenDJ</span>
>
> would look like
>
> `sudo git clone ssh://git@github.com:`<span class="placeholder-example">hansolo</span>`/IM_OpenDJ`
>
> after the text has been replaced.
>
>

* <span class="placeholder-example">some text</span> indicates an example value for the [*some text*{: style="color: red"}] placeholder.
* ***IMPORTANT*:**{: style="color: #f00"} <span style=" background-color: #ff0;">These are example values for demonstration purposes only; the proper values for your environment may differ.</span>

# Application Configuration
<ul id="dc_toc" style="list-style: none">
    {% for p in site.deployment %}
        {% if p.categories contains "tds-config" %}
            <li><a href="{{ p.url }}">{{ p.title }}</a></li>
        {% endif %}
    {% endfor %}
</ul>

# Accommodation Configuration
<ul id="dc_toc" style="list-style: none">
    {% for p in site.deployment %}
        {% if p.categories contains "accommodations" %}
            <li><a href="{{ p.url }}">{{ p.title }}</a></li>
        {% endif %}
    {% endfor %}
</ul>