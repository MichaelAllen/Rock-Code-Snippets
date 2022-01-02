---
tags:
    - language/sql
    - language/lava
    - domain/plugin/project-management
    - type/report
date created: 2020-12-01 22:38:39
date modified: 2022-01-01 22:49:20
---

# Projects and Tasks Due This Week

Returns a list of tasks and projects, assigned to the specified user, that are due in the next X days, or are overdue.

We use this in a weekly email to keep people from forgetting the tasks and projects that are assigned to them.

# Code

```sql
{% sql %}
DECLARE @Person int = {{ Person.Id | AsInteger }};
DECLARE @LookAheadDays int = 7;
DECLARE @Today datetime = CAST( CAST( GETDATE() AS date ) AS datetime );

SELECT
    p.[Id]
    ,p.[Name]
    ,p.[DueDate]
    ,CASE
        WHEN p.[DueDate] < @Today THEN 'Past Due'
        ELSE NULL
    END 'PastDue'
FROM
    [_com_blueboxmoon_ProjectManagement_Project] p
    JOIN [_com_blueboxmoon_ProjectManagement_ProjectAssignee] pas ON p.[Id] = pas.[ProjectId]
    JOIN [PersonAlias] pa ON pas.[PersonAliasId] = pa.[Id]
WHERE
    [IsActive] = 1
    AND p.[State] = 'Active'
    AND pa.[PersonId] = @Person
    AND p.[DueDate] <= DATEADD( day, @LookAheadDays, @Today )

UNION

SELECT
    t.[ProjectId]
    ,CONCAT( '[', p.[Name], '] ', t.[Name] ) 'Name'
    ,t.[DueDate]
    ,CASE
        WHEN t.[DueDate] < @Today THEN 'Past Due'
        ELSE NULL
    END 'PastDue'
FROM
    [_com_blueboxmoon_ProjectManagement_Task] t
    JOIN [PersonAlias] pa ON t.[AssignedToPersonAliasId] = pa.[Id]
    JOIN [_com_blueboxmoon_ProjectManagement_Project] p ON t.[ProjectId] = p.[Id]
WHERE
    t.[IsActive] = 1
    AND t.[State] = 'Active'
    AND pa.[PersonId] = @Person
    AND t.[DueDate] <= DATEADD( day, @LookAheadDays, @Today )
{% endsql %}

{% assign items = results | OrderBy:'DueDate' %}
{% for item in items %}
    <p style:'font-size: 15px;'>
        <a href="{{ 'Global' | Attribute:'InternalApplicationRoot' }}/Project/{{ item.Id }}">
            {{ item.Name }}<br>
        </a>
        {{ item.DueDate | Date:'dddd, MMM d, yyyy'}}
    {% if item.PastDue %}
        <span style='color: red;'>- ({{ item.PastDue }})</span>
    {% endif %}
    </p>
{% endfor %}
```
