---
tags:
    - language/sql
    - domain/checkin
    - type/reporting
    - attendance
date created: 2021-01-11 15:39:34
date modified: 2022-01-01 11:17:11
---

# Checkin Times Under Parent Group

## Page Parameter Block

Filters:

- Sunday Date [Date]
- Parent Group [Group]
- Service Time [Schedule]

## Dynamic Data Block

```sql
{% unless PageParameter.ParentGroup and PageParameter.ParentGroup != empty and PageParameter.ServiceTime and PageParameter.ServiceTime != empty and PageParameter.SundayDate and PageParameter.SundayDate != empty %}
SELECT 'Please fill in the filters above.';
{% else %}
DECLARE @ParentGroupID int = ( SELECT [Id] FROM [Group] WHERE [Guid] = '{{ PageParameter.ParentGroup | SanitizeSql }}' );
DECLARE @ScheduleID int = ( SELECT [Id] FROM [Schedule] WHERE [Guid] = '{{ PageParameter.ServiceTime | SanitizeSql }}' );
DECLARE @Date date = CAST ( '{{ PageParameter.SundayDate | SanitizeSql }}' AS date );

SELECT
    FORMAT( a.[CreatedDateTime], 'HH:mm:ss' ) 'Check in time'
    ,p.[FirstName]
    ,p.[LastName]
    ,g.[Name]
    ,p.[Id]
FROM
    [Group] g
    JOIN [AttendanceOccurrence] ao ON g.[Id] = ao.[GroupId]
    JOIN [Attendance] a ON ao.[Id] = a.[OccurrenceId]
    JOIN [PersonAlias] pa ON a.[PersonAliasId] = pa.[Id]
    JOIN [Person] p ON pa.[PersonId] = p.[Id]
WHERE
    g.[ParentGroupId] = @ParentGroupID
    AND a.[DidAttend] = 1
    AND ao.[SundayDate] = @Date
    AND ao.[ScheduleId] = @ScheduleID
ORDER BY
    a.[CreatedDateTime]
{% endunless %}
```
