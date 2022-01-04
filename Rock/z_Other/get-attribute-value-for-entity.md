---
tags:
    - language/sql
    - type/utility
date created: 2021-01-23 19:16:58
date modified: 2022-01-01 11:42:13
---

# Get Attribute Value for Entity

Given an entity Id and an attribute Key, return the raw value of that attribute for that entity.

Roughly equivalent to `{{ Entity | Attribute:'Key','RawValue' }}`

## Query

```sql
DECLARE @EntityTypeId int = (
    SELECT [Id]
    FROM [EntityType]
    WHERE [Guid] = 'ef79f12c-dd73-4a82-a1e2-7be76e3c5282' -- Rock.Model.Person
);
DECLARE @EntityId int = 27; -- Person Id
DECLARE @AttributeKey varchar(max) = 'TestFile'; -- Key of the attribute

SELECT av.[Value]
FROM
    [Attribute] a
    JOIN [AttributeValue] av ON av.[AttributeId] = a.[Id]
WHERE
    a.[EntityTypeId] = @EntityTypeId
    AND av.[EntityId] = @EntityId
    AND a.[Key] = @AttributeKey
```
