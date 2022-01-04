---
tags:
    - language/lava
    - type/reporting
date created: 2020-12-04 23:24:00
date modified: 2022-01-01 22:33:36
---

# Workflow Table with All Attributes

This will show a table with all workflows of a particular type, with a column for every attribute in the workflow.

## Lava

```liquid
{% workflow where:'WorkflowTypeId == "25"' %}
    {% assign firstWorkflow = workflowItems | First %}
    
    {% capture attributeNames -%}
        {%- for av in firstWorkflow.AttributeValues -%}
            {{ av.AttributeName | StripNewLines }}|
        {%- endfor -%}
    {%- endcapture %}
    
    {% assign attributeNames = attributeNames | ReplaceLast:'|','' | Split:'|' %}
    
<div class="table-wrapper">
    <table id="example" class="table table-striped table-bordered">
        <thead>
            <tr>
                <th>Workflow Name</th>
    {% for name in attributeNames %}
                <th>{{ name }}</th>
    {% endfor %}
            </tr>
        </thead>
        <tbody>
    {% for w in workflowItems %}
            <tr>
                <td>{{ w.Name }}</td>
        {% for a in w.AttributeValues %}
                <td>{{ a.ValueFormatted | Default:'(none)' }}</td>
        {% endfor %}
            </tr>
    {% endfor %}
        </tbody>
    </table>
</div>
{% endworkflow %}
```
