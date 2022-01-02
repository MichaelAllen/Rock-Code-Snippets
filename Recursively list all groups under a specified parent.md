---
tags:
    - language/sql
    - domain/groups
    - type/reporting
date created: 2020-12-04 23:24:00
date modified: 2022-01-01 17:50:55
---

# Recursively List All Groups Under a Specified Parent

Traverses a group tree and returns all descendant groups.

Data returned includes Id, Names, and Path. The path is in the following format:

- GroupName
- ParentGroupName :: GroupName
- GrandparentGroupName :: ParentGroupName :: GroupName
- etc.

## Query

```sql
DECLARE @parentGroupId int = 41;

DECLARE @groups table("Id" int, "Name" varchar(max), "Path" varchar(max));

-- Recursively get all groups under the parent
WITH CTE AS (
    SELECT
        g.Id
        ,g.ParentGroupId
        ,CAST( g.Name AS Varchar(max) ) 'Name'
        ,CAST( g.Name AS Varchar(max) ) 'Path'
    FROM [Group] g
    WHERE g.ParentGroupId = @parentGroupId

    UNION ALL

    SELECT
        g.Id
        ,g.ParentGroupId
        ,CAST( g.Name AS varchar(max) ) 'Name'
        ,CAST( CONCAT( CTE.Path, ' :: ', g.Name ) AS Varchar(max) ) 'Path'
    FROM
        [Group] g
        INNER JOIN CTE ON g.ParentGroupId = CTE.Id
)
INSERT INTO @groups
SELECT Id, Name, Path
FROM CTE;

-- Preview the selected groups
SELECT * FROM @groups ORDER BY Path;
```
