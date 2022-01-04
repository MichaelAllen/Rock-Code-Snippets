---
tags:
    - language/sql
    - type/reporting
date created: 2020-12-04 23:24:00
date modified: 2022-01-01 21:48:05
---

# Checkin Times

List all people that checked into the specified group types on the specified date, along with the time that they checked in.

## Query

```sql
DECLARE @GroupTypes varchar(max) = '58,59'; --Serve Team GroupType IDs

SELECT
    DISTINCT(p.Id)
   ,a.CreatedDateTime 'CheckinDateTime'
   ,p.FirstName
   ,p.LastName
   ,FORMAT(a.CreatedDateTime, 'h:mm tt') 'CheckinTime'
   ,g.Name 'Group'
FROM
    [Attendance] a
    JOIN [AttendanceOccurrence] ao ON a.OccurrenceId = ao.Id
    JOIN [Group] g on AO.GroupId = g.Id
    JOIN [PersonAlias] pa ON a.PersonAliasId = pa.Id
    JOIN [Person] p ON pa.PersonId = p.Id
WHERE
    g.GroupTypeId IN ( SELECT * FROM dbo.ufnUtility_CsvToTable( @GroupTypes ) )
    AND ao.OccurrenceDate = @Date
ORDER BY a.CreatedDateTime
```
