---
tags:
    - language/sql
    - type/reporting
date created: 2024-07-21 16:55:10
date modified: 2024-07-21 16:59:55
---

# Checkin Code Historical Lookup

This will find all historical usage of a given checkin code.

*Note: Every checkin gets assigned a code, even if it doesn't print on the label.*

## Query

```sql
DECLARE @Code varchar(max) = 'BMD';

SELECT
    ac.[IssueDateTime]
    ,p.[FirstName]
    ,p.[LastName]
    ,g.[Name] 'Group'
    ,l.[Name] 'Location'
    ,s.[Name] 'Schedule'
FROM
    [AttendanceCode] ac
    JOIN [Attendance] a ON a.[AttendanceCodeId] = ac.[Id]
    JOIN [PersonAlias] pa ON pa.[Id] = a.[PersonAliasId]
    JOIN [Person] p ON p.[Id] = pa.[PersonId]
    JOIN [AttendanceOccurrence] ao ON ao.[Id] = a.[OccurrenceId]
    JOIN [Group] g ON g.[Id] = ao.[GroupId]
    JOIN [Location] l ON l.[Id] = ao.[LocationId]
    JOIN [Schedule] s ON s.[Id] = ao.[ScheduleId]
WHERE ac.[Code] = @Code
ORDER BY ac.[IssueDateTime] DESC
```
