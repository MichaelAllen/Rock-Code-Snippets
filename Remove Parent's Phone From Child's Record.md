---
tags:
    - language/sql
    - domain/groups
    - type/cleanup
    - family
date created: 2020-12-04 23:24:00
date modified: 2022-01-01 18:05:48
---

# Remove Parent's Phone From Child's Record

> Credit: Daniel Hazelbaker

Finds all child records that have the same mobile phone as a parent, and removes the email from the child. It also records the change to person history for tracking purposes.

## Query

```sql
DECLARE @MobilePhoneTypeId int = (SELECT [Id] FROM [DefinedValue] WHERE [Guid] = '407E7E45-7B2E-4FCD-9605-ECB1339F2453');
DECLARE @FamilyTypeId int = (SELECT [Id] FROM [GroupType] WHERE [Guid] = '790E3215-3B10-442B-AF69-616C0DCB998E');
DECLARE @ChildPhonePersonIds TABLE ([Id] int);

-- Find all < 18 people that have the same mobile number as a family member that is > 18
INSERT INTO @ChildPhonePersonIds
SELECT DISTINCT P.[Id]
FROM
    [PhoneNumber] AS PN
    INNER JOIN [PhoneNumber] AS PN2 ON PN2.[Number] = PN.[Number] AND PN2.[Id] != PN.[Id]
    INNER JOIN [Person] AS P ON P.[Id] = PN.[PersonId]
    INNER JOIN [Person] AS P2 ON P2.[Id] = PN2.[PersonId]
    INNER JOIN [GroupMember] AS FM ON FM.[PersonId] = P.[Id]
    INNER JOIN [Group] AS F ON F.[Id] = FM.[GroupId]
    INNER JOIN [GroupMember] AS FM2 ON FM2.[PersonId] = P2.[Id]
    INNER JOIN [Group] AS F2 ON F2.[Id] = FM2.[GroupId]
WHERE
    PN.[NumberTypeValueId] = @MobilePhoneTypeId
    AND F.[GroupTypeId] = @FamilyTypeId
    AND F2.[GroupTypeId] = @FamilyTypeId
    AND F.[Id] = F2.[Id]
    AND dbo.ufnCrm_GetAge(P.[BirthDate]) < 18
    AND dbo.ufnCrm_GetAge(P2.[BirthDate]) >= 18
;
-- Find all child relationship Person records that have the same mobile phone as their parent.
DECLARE @KnownRelationshipTypeId int = (SELECT [Id] FROM [GroupType] WHERE [Guid] = 'E0C5A0E2-B7B3-4EF4-820D-BBF7F9A374EF');
DECLARE @OwnerRelationshipId int = (SELECT [Id] FROM [GroupTypeRole] WHERE [Guid] = '7BC6C12E-0CD1-4DFD-8D5B-1B35AE714C42');
DECLARE @ChildRelationshipId int = (SELECT [Id] FROM [GroupTypeRole] WHERE [Guid] = 'F87DF00F-E86D-4771-A3AE-DBF79B78CF5D');
DECLARE @StepChildRelationshipId int = (SELECT [Id] FROM [GroupTypeRole] WHERE [Guid] = 'EFD2D6D1-A407-4EFB-9086-5DF1F19B7D93');
DECLARE @ChildRelationshipPhonePersonIds TABLE ([Id] int);

INSERT INTO @ChildRelationshipPhonePersonIds
SELECT DISTINCT P2.[Id]
FROM
    [Person] AS P
    INNER JOIN [GroupMember] AS FM ON FM.[PersonId] = P.[Id]
    INNER JOIN [Group] AS KR ON KR.[Id] = FM.[GroupId]
    INNER JOIN [PhoneNumber] AS PN ON PN.[PersonId] = P.[Id]
    INNER JOIN [Group] AS KR2 ON KR2.[Id] = KR.[Id]
    INNER JOIN [GroupMember] AS FM2 ON FM2.[GroupId] = KR2.[Id]
    INNER JOIN [Person] AS P2 ON P2.[Id] = FM2.[PersonId]
    INNER JOIN [PhoneNumber] AS PN2 ON PN2.[PersonId] = P2.[Id]
WHERE
    KR.[GroupTypeId] = @KnownRelationshipTypeId
    AND KR2.[GroupTypeId] = @KnownRelationshipTypeId
    AND KR.[Id] = KR2.[Id]
    AND P.[Id] != P2.[Id]
    AND FM.[GroupRoleId] = @OwnerRelationshipId
    AND ( FM2.[GroupRoleId] = @ChildRelationshipId OR FM2.[GroupRoleId] = @StepChildRelationshipId )
    AND PN.[NumberTypeValueId] = @MobilePhoneTypeId
    AND PN2.[NumberTypeValueId] = @MobilePhoneTypeId
    AND PN.[Number] = PN2.[Number]
;
-- Combine both lists into one.
DECLARE @PersonIds TABLE ([Id] int, [OldValue] varchar(100))
INSERT INTO @PersonIds
SELECT DISTINCT IQ.[Id], NULL
FROM (
    SELECT [Id] FROM @ChildPhonePersonIds
    UNION
    SELECT [Id] FROM @ChildRelationshipPhonePersonIds
) AS IQ
ORDER BY IQ.[Id]
;
-- Add the current (old) number to the list of people.
UPDATE P
SET P.[OldValue] = PN.[NumberFormatted]
FROM @PersonIds AS P
INNER JOIN [PhoneNumber] AS PN ON PN.[PersonId] = P.[Id] AND PN.[NumberTypeValueId] = @MobilePhoneTypeId
;
-- Add History records showing that we are removing the phone number.
INSERT INTO [History] (
    [IsSystem]
    ,[CategoryId]
    ,[EntityTypeId]
    ,[EntityId]
    ,[Guid]
    ,[CreatedDateTime]
    ,[ModifiedDateTime]
    ,[Verb]
    ,[ChangeType]
    ,[ValueName]
    ,[OldValue]
    ,[IsSensitive]
)
SELECT
    0
    ,133
    ,15
    ,P.[Id]
    ,NEWID()
    ,GETDATE()
    ,GETDATE()
    ,'MODIFY'
    ,'Property'
    ,'Mobile Phone'
    ,P.OldValue
    ,0
FROM @PersonIds AS P
;
-- Remove the phone numbers.
DELETE FROM [PhoneNumber] WHERE [PersonId] IN (SELECT [Id] FROM @PersonIds) AND [NumberTypeValueId] = @MobilePhoneTypeId;
```
