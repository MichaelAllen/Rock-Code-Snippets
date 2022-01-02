---
tags:
    - language/lava
    - language/sql
    - domain/checkin
    - type/utility
date created: 2020-12-04 23:24:00
date modified: 2022-01-01 11:35:35
---

# Auto-select Group Types for Given Checkin Config

This will lookup all group types under a given check-in config, then redirect to the check-in page with those areas selected.

## Page Route

`checkin-launch, checkin-launch/{KioskId}/{CheckinConfigId}`

## HTML Block Contents

```sql
{% assign kioskId = 'Global' | PageParameter:'KioskId' | AsInteger %}
{% assign configId = 'Global' | PageParameter:'CheckinConfigId' | AsInteger %}

{% unless kioskId and configId and kioskId > 0 and configId > 0 %}
    <div class="alert alert-warning">Error: Invalid parameters</div>
{% else %}
    {% sql %}
        DECLARE @CheckinConfigId int = {{ configId }};
        
        WITH CheckInAreas AS 
            (
                SELECT
                    parent.[Id] AS ParentId
                   ,parent.[Name] AS Parent
                   ,child.[Id] AS ChildId
                   ,child.[Name] AS Child
                FROM
                    [GroupTypeAssociation] gta
                    INNER JOIN [GroupType] parent ON gta.[GroupTypeId] = parent.[Id]
                    INNER JOIN [GroupType] child ON gta.[ChildGroupTypeId] = child.[Id]
                WHERE
                    parent.[Id] = @CheckinConfigId
        
                UNION ALL
                
                SELECT
                    parent.[Id] AS ParentId
                   ,parent.[Name] AS Parent
                   ,child.[Id] AS ChildId
                   ,child.[Name] AS Child
                FROM
                    [CheckinAreas] ca
                    JOIN [GroupTypeAssociation] gta ON ca.[ChildId] = gta.[GroupTypeId]
                    INNER JOIN [GroupType] parent ON gta.[GroupTypeId] = parent.[Id]
                    INNER JOIN [GroupType] child ON gta.[ChildGroupTypeId] = child.[Id]
                WHERE
                    parent.[Id] <> child.[Id]
            )
        SELECT STRING_AGG(ChildId, ',') 'GroupTypeIds'
        FROM [CheckInAreas]
    {% endsql %}
    
    {% capture url %}/checkin/{{ kioskId }}/{{ configId }}/{{ results | First | Property:'GroupTypeIds' }}{% endcapture%}
    
    {% assign canEdit = 'Global' | Page:'Id' | HasRightsTo:'Edit','Rock.Model.Page' %}
    {% if canEdit %}
        <p class="alert alert-warning">If you could not edit this page you would be redirected to: <a href="{{ url }}">{{ url }}</a>.</p>
    {% else %}
        {{ url | PageRedirect }}
    {% endif %}
{% endunless %}
```
