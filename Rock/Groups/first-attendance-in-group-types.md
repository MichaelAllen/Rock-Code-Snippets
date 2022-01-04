---
tags: 
    - language/sql
    - type/reporting
    - topic/attendance
date created: 2020-12-04 23:24:00
date modified: 2022-01-04 10:11:04
---

# First Attendance in Group Types

Return anyone that attended any group of the specified types for the first time between `@StartDate` and `@EndDate`.

## Query

```sql
DECLARE @GroupTypes varchar(max) = '38,41,58,59'; --Serve Teams
DECLARE @StartDate Date = '2019-11-01';
DECLARE @EndDate Date = '2019-12-31';

WITH cte_Ranked AS (
    SELECT
        a.PersonAliasId
        ,g.Id 'GroupId'
        ,ao.OccurrenceDate
        ,ROW_NUMBER() OVER ( PARTITION BY a.PersonAliasId ORDER BY ao.OccurrenceDate ASC ) 'Rank'
    FROM
        [AttendanceOccurrence] ao
        JOIN [Attendance] a
            ON ao.Id = a.OccurrenceId
            AND a.DidAttend = 1
        INNER JOIN [Group] g
            ON ao.GroupId = g.Id
            AND g.GroupTypeId IN ( SELECT * FROM dbo.ufnUtility_CsvToTable( @GroupTypes ) )
            AND g.IsActive = 1
            AND g.IsArchived = 0
    WHERE
        ao.DidNotOccur = 0
)
SELECT 
    r.OccurrenceDate 'First Attendance'
    ,p.Id 'PersonId'
    ,p.FirstName
    ,p.LastName
    ,g.Id 'GroupId'
    ,g.Name 'GroupName'
FROM
    [cte_Ranked] r
    JOIN [Group] g ON r.GroupId = g.Id
    JOIN [PersonAlias] pa ON r.PersonAliasId = pa.Id
    JOIN [Person] p ON pa.PersonId = p.Id
WHERE
    r.Rank = 1
    AND r.OccurrenceDate BETWEEN @StartDate AND @EndDate
ORDER BY
    r.OccurrenceDate DESC
    ,p.LastName
    ,p.FirstName
```
