---
title: Exercises for Azure developers
permalink: index.html
layout: home
---

# Develop solutions for Microsoft Azure

This page lists exercises associated with Microsoft skilling content on [Microsoft Learn](https://learn.microsoft.com)

The following exercises are designed to provide you with a hands-on learning experience in which you'll explore common tasks that developers perform when building and deploying solutions to Microsoft Azure.

> **Note**: To complete the exercises, you'll need an Azure subscription in which you have sufficient permissions and quota to provision the necessary Azure resources. If you don't already have one, you can sign up for an [Azure account](https://azure.microsoft.com/free). Be sure to review the **Before you start** section in each exercise for the full list of requirements specific to each exercise.

## Topic areas
{% assign exercises = site.pages | where_exp:"page", "page.url contains '/Instructions'" %}
{% assign grouped_exercises = exercises | group_by: "lab.topic" %}

<ul>
{% for group in grouped_exercises %}
<li><a href="#{{ group.name | slugify }}">{{ group.name }}</a></li>
{% endfor %}
</ul>

{% for group in grouped_exercises %}

## <a id="{{ group.name | slugify }}"></a>{{ group.name }} 
{% for activity in group.items %}
| [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) <br/> {{ activity.lab.description }} |

{% endfor %}
| <a href="#develop-solutions-for-microsoft-azure">Return to top</a> |
{% endfor %}

