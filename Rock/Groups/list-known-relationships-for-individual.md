---
tags:
    - language/sql
    - type/reporting
    - topic/known-relationship
date created: 2020-12-04 23:24:00
date modified: 2022-01-04 10:11:30
---

# List Known Relationships for Individual

Returns a list of all of the known relationships for the provided person

## Query

```sql
DECLARE @PersonId int = 515; --Who are we looking at?
DECLARE @OwnerRoleId int = 5; --Owner

DECLARE @KnownRelationshipGroupId int = (
    SELECT TOP 1 GroupId
    FROM [GroupMember]
    WHERE
        PersonId = @PersonId
        AND GroupRoleId = @OwnerRoleId
);

SELECT
    r.Name 'Role'
    ,CONCAT( p.NickName, ' ', p.LastName ) 'Person'
FROM
    [Group] g
    JOIN [GroupMember] gm ON g.Id = gm.GroupId
    JOIN [GroupTypeRole] r ON gm.GroupRoleId = r.Id
    JOIN [Person] p ON gm.PersonId = p.Id
WHERE
    g.Id = @KnownRelationshipGroupId
    AND gm.GroupRoleId <> @OwnerRoleId
ORDER BY 'Role', 'Person'
```
