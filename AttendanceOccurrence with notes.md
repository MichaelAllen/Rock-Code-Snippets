---
tags:
    - language/sql
    - domain/groups
    - type/reporting
    - attendance
date created: 2020-12-04 23:24:00
date modified: 2022-01-01 11:47:49
---

# AttendanceOccurrence with Notes

## Parameters

- `@StartDate` (Date)
- `@EndDate` (Date)

## Query

```sql
SELECT
    g.Id 'GroupId'
    ,ao.Id 'OccurrenceId'
    ,ao.OccurrenceDate 'Date'
    ,CONCAT( '<a href="/page/570?GroupId=', g.Id, '">', g.Name, '</a>' ) 'Group'
    ,ao.Notes
FROM
    [AttendanceOccurrence] ao
    JOIN [Group] g ON ao.GroupId = g.Id
    JOIN [GroupType] gt ON g.GroupTypeId = gt.Id
WHERE
    gt.Guid = @GroupType
    AND ao.OccurrenceDate BETWEEN LEFT( @StartDate, 10 ) AND LEFT( @EndDate, 10 )
    AND ao.Notes <> ''
ORDER BY
    ao.OccurrenceDate DESC
    ,g.Name
```
