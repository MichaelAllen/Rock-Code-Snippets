---
title: Tags
---

# Contents grouped by tag
{% for tag, pages in tags %}
## <span class="tag">{{tag}}</span>

{% for page in pages %}
  - [
  {%- set path = page.filename.split("/") -%}
  {%- for item in path -%}
    {%- if not loop.last %}{{ item }} :: {% endif -%}
  {%- endfor -%}
  {{page.title}}]({{page.filename}})
{% endfor %}

{% endfor %}
