---
tags:
    - language/sql
    - type/cleanup
date created: 2022-05-17 07:52:13
date modified: 2022-05-17 08:12:55
---

# Fix Corrupted Block Attribute Value

Occasionally a block's attribute value will get so messed up that you can't even load the page to fix it. (Usually caused by a bad lava template). These instructions will help to fix that issue.

## Steps

### 1) Try CMS Config

Sometimes you can't load the page itself, but you can still access its block settings through `Admin Tools > CMS Configuration > Pages`. If that page also errors out, then SQL is probably your only option.

### 2) Find the Block Id of the Offending Block

This query will give you a list of all the blocks on the specified page.

```sql
DECLARE @PageId int = 3; --Replace with the Id of your broken page

SELECT
  b.[Id] 'BlockId'
  ,bt.[Name] 'BlockType'
  ,b.[Name]
FROM
  [Block] b
  JOIN [BlockType] bt ON b.[BlockTypeId] = bt.[Id]
WHERE b.[PageId] = @PageId
```

### 3) Find the Attribute Id of the Corrupted Attribute

This query will list all of the attributes of the specified block.

```sql
DECLARE @BlockId int = 10; --Replace with the Id of the block that has the corrupted attribute

DECLARE @BlockEntityTypeId int = ( SELECT [Id] FROM [EntityType] WHERE [Name] = 'Rock.Model.Block' );
SELECT
  av.[Id] 'AttributeValueId'
  ,a.[Name]
  ,av.[Value] 
FROM
  [Attribute] a
  JOIN [AttributeValue] av ON a.[Id] = av.[AttributeId]
WHERE
  a.[EntityTypeId] = @BlockEntityTypeId
  AND av.[EntityId] = @BlockId
```

### 4) Update the Corrupted Attribute

There are 2 options here. We can either try to save the work we've done by commenting it out, or we can delete the corrupted value and let Rock set it back to the default value.

*Be careful. A typo in this step can destroy data!*

**Option 1** This query will attempt to "comment out" the value. You can use this in cases of a bad lava template causing issues, but it won't work with any other type of attribute.

```sql
DECLARE @AttributeValueId int = 123; --Replace with the Id of the corrupted attribute

UPDATE [AttributeValue]
SET [Value] = '{% comment %}{% raw %}' + [Value] + '{% endraw %}{% endcomment %}'
WHERE [Id] = @AttributeValueId
```

**Option 2** This query will delete the corrupted attribute; causing Rock to set it back to its default value.

```sql
DECLARE @AttributeValueId int = 123; --Replace with the Id of the corrupted attribute

DELETE
FROM [AttributeValue]
WHERE [Id] = @AttributeValueId
```

### 5) Clear the Rock Cache

Once you have cleared the cache and reloaded the site, you should be able to access the page again.
