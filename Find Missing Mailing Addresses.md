---
tags:
    - language/sql
    - domain/crm
    - type/cleanup
date created: 2020-12-04 23:24:00
date modified: 2022-01-01 11:23:03
---

# Find Missing Mailing Addresses

> Credit: Daniel Hazelbaker

Finds all people with a home address but no mailing address. Optionally sets the mailing address flag on their first home address.

## Query

```sql
BEGIN TRANSACTION

CREATE TABLE #HasHomeAddress ([PersonId] int)
CREATE TABLE #HasMailingAddress ([PersonId] int)
CREATE TABLE #MissingMailingFlag ([PersonId] int, [GroupLocationId] int NULL)

DECLARE @FamilyGroupTypeId int = (SELECT [Id] FROM [GroupType] WHERE [Guid] = '790E3215-3B10-442B-AF69-616C0DCB998E')
DECLARE @HomeAddressValueId int = (SELECT [Id] FROM [DefinedValue] WHERE [Guid] = '8C52E53C-2A66-435A-AE6E-5EE307D9A0DC')
DECLARE @PersonRecordTypeValueId int = (SELECT [Id] FROM [DefinedValue] WHERE [Guid] = '36CF10D6-C695-413D-8E7C-4546EFEF385E')

-- Find every Person who has a Home address type.
INSERT INTO #HasHomeAddress
SELECT DISTINCT P.[Id]
FROM
    [Person] AS P
    INNER JOIN [GroupMember] AS GM ON GM.[PersonId] = P.[Id]
    INNER JOIN [Group] AS G ON G.[Id] = GM.[GroupId]
    INNER JOIN [GroupLocation] AS GL ON GL.[GroupId] = G.[Id]
WHERE
    G.[GroupTypeId] = @FamilyGroupTypeId
    AND P.[RecordTypeValueId] = @PersonRecordTypeValueId
    AND GL.[GroupLocationTypeValueId] = @HomeAddressValueId

-- Find every Person who has a mailing address.
INSERT INTO #HasMailingAddress
SELECT DISTINCT P.[Id]
FROM
    [Person] AS P
    INNER JOIN [GroupMember] AS GM ON GM.[PersonId] = P.[Id]
    INNER JOIN [Group] AS G ON G.[Id] = GM.[GroupId]
    INNER JOIN [GroupLocation] AS GL ON GL.[GroupId] = G.[Id]
WHERE
    G.[GroupTypeId] = @FamilyGroupTypeId
    AND GL.[IsMailingLocation] = 1

-- Find every Person who has a Home address but does NOT have a Mailing address.
INSERT INTO #MissingMailingFlag ([PersonId])
SELECT H.[PersonId]
FROM
    #HasHomeAddress AS H
    LEFT JOIN #HasMailingAddress AS M ON M.[PersonId] = H.[PersonId]
WHERE M.[PersonId] IS NULL

-- Update the list of people with their first found Home address.
UPDATE M
SET M.[GroupLocationId] = (
    SELECT TOP 1 GL.[Id]
    FROM
        [Person] AS P
		INNER JOIN [GroupMember] AS GM ON GM.[PersonId] = P.[Id]
		INNER JOIN [Group] AS G ON G.[Id] = GM.[GroupId]
		INNER JOIN [GroupLocation] AS GL ON GL.[GroupId] = G.[Id]
    WHERE
        P.[Id] = M.[PersonId]
        AND G.[GroupTypeId] = @FamilyGroupTypeId
        AND GL.[GroupLocationTypeValueId] = @HomeAddressValueId
)
FROM #MissingMailingFlag AS M

SELECT * FROM #MissingMailingFlag

-- Uncomment the following to mark the first Home address as mailing
/*
UPDATE GL
SET GL.[IsMailingLocation] = 1
FROM
    [GroupLocation] AS GL
    INNER JOIN #MissingMailingFlag AS M ON M.[GroupLocationId] = GL.[Id]
*/

DROP TABLE #HasHomeAddress
DROP TABLE #HasMailingAddress
DROP TABLE #MissingMailingFlag

ROLLBACK TRANSACTION
--COMMIT TRANSACTION
```
