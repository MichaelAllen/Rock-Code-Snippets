---
tags:
    - language/sql
    - domain/groups
    - type/reporting
date created: 2020-12-04 23:24:00
date modified: 2022-01-01 16:44:11
---

# Latest Attendance Dates Per-group

For each group of the specified type, list the most recent attendance occurrence, the number of people attending, and any attendance notes entered.

# Query

```sql
DECLARE @GroupType int = 25; --Life Groups

WITH cte_Ranked AS (
    SELECT
        g.Id 'GroupId'
        ,ao.Id 'AttendanceOccurrenceId'
        ,RANK() OVER ( PARTITION BY g.Id ORDER BY ao.OccurrenceDate DESC ) 'Rank'
    FROM
        [Group] g
        LEFT JOIN [AttendanceOccurrence] ao
            ON g.Id = ao.GroupId
            AND ao.DidNotOccur = 0
            
    WHERE
        g.GroupTypeId = @GroupType
        AND g.IsActive = 1
        AND g.IsArchived = 0
)
SELECT 
    g.Id
    ,g.Name
    ,ao.OccurrenceDate 'Last Attendance'
    ,NULLIF( (
        SELECT COUNT( 1 )
        FROM [Attendance]
        WHERE
            OccurrenceId = r.AttendanceOccurrenceId
            AND DidAttend = 1
    ), 0 ) 'Attendee Count'
    ,ao.Notes 'Attendance Note'
FROM
    [cte_Ranked] r
    JOIN [Group] g ON r.GroupId = g.Id
    LEFT JOIN [AttendanceOccurrence] ao ON r.AttendanceOccurrenceId = ao.Id
WHERE r.Rank = 1
ORDER BY ao.OccurrenceDate DESC
```
