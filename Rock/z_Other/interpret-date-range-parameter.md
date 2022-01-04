---
tags:
    - language/lava
    - language/sql
    - type/utility
date created: 2021-11-12 16:14:08
date modified: 2022-01-01 11:40:56
---

# Interpret a Date Range Parameter

## Option 1 - Web Request to API

```liquid
{% assign filter = 'Global' | PageParameter:'Date' | Default:'0' | UnescapeDataString %}
{% if filter == '0' %}
    SELECT 'Please provide a valid filter...'
{% else %}
    {% assign filter = filter | Split:'|' %}
    {% if filter[0] == 'DateRange' %}
        {% assign start = filter[1] | Replace:'+', ' ' | Date:'yyyy-MM-dd' %}
        {% assign end = filter[2] | Replace:'+', ' ' |  Date:'yyyy-MM-dd'  %}
    {% elseif filter[0] == 'Current' %}
        {% webrequest url:'{{ 'Global' | Attribute:'InternalApplicationRoot' }}api/Utility/CalculateSlidingDateRange' parameters:'slidingDateRangeType^{{ filter[0] }}|timeUnitType^{{ filter[1] }}' responsecontenttype:'application/json' %}
            {% assign start = results | Remove:'"' | Split:' to ' | Index:0 | Date:'yyyy-MM-ddTHH:mm:ss' %}
            {% assign end = results | Remove:'"' | Split:' to ' | Index:1 | Date:'yyyy-MM-ddTHH:mm:ss' %}
        {% endwebrequest %}
    {% else %}
        {% webrequest url:'{{ 'Global' | Attribute:'InternalApplicationRoot' }}api/Utility/CalculateSlidingDateRange' parameters:'slidingDateRangeType^{{ filter[0] }}|timeUnitType^{{ filter[2] }}|number^{{ filter[1] }}' responsecontenttype:'application/json' %}
            {% assign start = results | Remove:'"' | Split:' to ' | Index:0 | Date:'yyyy-MM-ddTHH:mm:ss' %}
            {% assign end = results | Remove:'"' | Split:' to ' | Index:1 | Date:'yyyy-MM-ddTHH:mm:ss' %}
        {% endwebrequest %}
    {% endif %}
    
    SELECT '{{ start }}' AS 'Start', '{{ end }}' AS 'End'
{% endif %}
```

## Option 2 - Execute

```liquid
{% assign filter = 'Global' | PageParameter:'Data' | Default:'0' | UnescapeDataString %}
{% if filter == '0' %}
    SELECT 'Please provide a valid filter...'
{% else %}
    {% capture result %}{% execute %}
        var range = Rock.Web.UI.Controls.SlidingDateRangePicker.CalculateDateRangeFromDelimitedValues( "{{ filter }}" );
        return string.Format( "{0:yyyy-MM-ddTHH:mm:ss},{1:yyyy-MM-ddTHH:mm:ss}", range.Start, range.End );
    {% endexecute %}{% endcapture %}
    {% assign start = result | Split:',' | Index:0 }
    {% assign end = result | Split:',' | Index:1 }
    
    SELECT '{{ start }}' AS 'Start', '{{ end }}' AS 'End'
{% endif %}
```
