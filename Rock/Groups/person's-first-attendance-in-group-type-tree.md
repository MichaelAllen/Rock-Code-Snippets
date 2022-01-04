---
tags:
    - language/sql
    - type/reporting
    - topic/attendance
date created: 2020-12-04 23:24:00
date modified: 2022-01-04 09:49:23
---

# Person's First Attendance in Group Type Tree

Search every group of the specified type, or any of its child types, to find the first time the specified person attended.

This can be easily modified to return their most recent attendance datetime by changing line 18 from `MIN` to `MAX`.

## Example

Given the following checkin structure:

```markdown
Volunteer Checkin (Checkin Configuration / Group Type)
  - Weekend Volunteers (Checkin Area / Group Type)
      - Worship (Group)
      - Production (Group)
          - Camera (Group)
          - Audio (Group)
  - Non-Weekend Volunteers (Checkin Area / Group Type)
      - Facilities (Group)
      - Reception (Group)
```

If you were to run this on "Volunteer Checkin", it will look through every group in the tree, regardless of how many levels deep it is.

## Query

_Note, this query will blow up if you have any circular group type associations_

```sql
DECLARE @parentGroupType INT = 49; --Volunteer Checkin
DECLARE @personId INT = 515;

WITH cte_grouptypes AS (
    SELECT CAST( @parentGroupType AS INT ) 'Id'
        
    UNION ALL
    
    SELECT
        ChildGroupTypeId 'Id'
    FROM
        [GroupTypeAssociation] gta
        INNER JOIN cte_grouptypes cte ON gta.GroupTypeId = cte.Id
    WHERE
        gta.GroupTypeID <> gta.ChildGroupTypeID
)
SELECT
    CONCAT( MIN( OccurrenceDate ),'T00:00:00'  )
FROM
    [Attendance] a
    JOIN [PersonAlias] pa ON a.PersonAliasId = pa.Id
    JOIN [AttendanceOccurrence] ao ON a.OccurrenceId = ao.Id
    JOIN [Group] g ON ao.GroupId = g.Id
    INNER JOIN [cte_grouptypes] cte ON g.GroupTypeId = cte.Id
WHERE
    a.DidAttend = 1
    AND pa.PersonId = @personId
```
