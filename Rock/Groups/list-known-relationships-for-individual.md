---
tags:
    - language/sql
    - language/lava
    - type/reporting
    - topic/known-relationship
date created: 2020-12-04 23:24:00
date modified: 2024-01-02 21:56:51
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

## Lava Alternative

```liquid
{% assign personKrGroup = Person | Groups:11 | Where:'GroupRoleId','5' | Select:'GroupId' | First | GroupById %}
<ul>
    {% for member in personKrGroup.Members %}
        <li>{{ member.Person.FullName }} - {{ member.GroupRole.Name }}</li>
    {% endfor %}
</ul>
```
