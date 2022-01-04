---
tags:
    - language/sql
    - type/reporting
    - topic/attendance 
date created: 2020-12-04 23:24:00
date modified: 2022-01-04 10:09:16
---

# Last Time Attending Per Serve Team

Lists everyone in a group of the specified types that hasn't attended at least once in the past X months.

## Query

```sql
DECLARE @GroupTypes varchar(max) = '38,41,58'; --Serve Teams (KW, Students, Weekend)
DECLARE @Months int = 6; --Filter out people that have attended in the past X months
DECLARE @AttendanceTable TABLE (
    PersonId int
    ,GroupId int
    ,OccurrenceDate Date
    ,Rank int
);

INSERT INTO @AttendanceTable ( PersonId, GroupId, OccurrenceDate, Rank )
(
    SELECT
        pa.PersonId
        ,g.Id 'GroupId'
        ,ao.OccurrenceDate
        ,ROW_NUMBER() OVER (
            PARTITION BY pa.PersonId, g.Id
            ORDER BY ao.OccurrenceDate DESC
        )
    FROM
        [AttendanceOccurrence] ao
        JOIN [Attendance] a
            ON ao.Id = a.OccurrenceId
            AND a.DidAttend = 1
        INNER JOIN [Group] g ON ao.GroupId = g.Id
        JOIN [PersonAlias] pa ON a.PersonAliasId = pa.Id
    WHERE
        ao.DidNotOccur = 0
        OR ao.DidNotOccur IS NULL
);


SELECT 
    p.Id 'PersonId'
    ,g.Id 'GroupId'
    ,g.Name 'Group'
    ,CONCAT( p.NickName, ' ', p.LastName) 'Person'
    ,a.OccurrenceDate 'LastCheckin'
FROM
    [GroupMember] gm
    INNER JOIN [Group] g
        ON gm.GroupId = g.Id
        AND g.GroupTypeId IN (
            SELECT * FROM dbo.ufnUtility_CsvToTable( @GroupTypes )
        )
        AND g.IsActive = 1
        AND g.IsArchived = 0
    JOIN [Person] p ON gm.PersonId = p.Id
    LEFT JOIN @AttendanceTable a
        ON g.Id = a.GroupId
        AND p.Id = a.PersonId
        AND a.Rank = 1
WHERE
    gm.GroupMemberStatus = 1 --Active
    AND gm.IsArchived = 0
    AND (
        a.OccurrenceDate < DATEADD( m, -1 * @Months, GETDATE() )
        OR a.OccurrenceDate IS NULL
    )
ORDER BY
    g.Name
    ,LastCheckin desc
    ,p.LastName
    ,p.FirstName
```
