---
tags:
    - language/sql
    - domain/groups
    - type/cleanup
    - family
date created: 2020-12-04 23:24:00
date modified: 2022-01-01 18:00:35
---

# Remove Parent's Email From Child's Record

> Credit: Daniel Hazelbaker

Finds all child records that have the same email as a parent, and removes the email from the child. It also records the change to person history for tracking purposes.

## Query

```sql
DECLARE @FamilyTypeId int = (SELECT [Id] FROM [GroupType] WHERE [Guid] = '790E3215-3B10-442B-AF69-616C0DCB998E');
DECLARE @ChildEmailPersonIds TABLE ([Id] int);

-- Find all < 18 people that have the same email as a family member that is > 18
INSERT INTO @ChildEmailPersonIds
SELECT DISTINCT P.[Id]
FROM
    [Person] AS P
    INNER JOIN [Person] AS P2 ON P2.[Email] = P.[Email]
    INNER JOIN [GroupMember] AS FM ON FM.[PersonId] = P.[Id]
    INNER JOIN [Group] AS F ON F.[Id] = FM.[GroupId]
    INNER JOIN [GroupMember] AS FM2 ON FM2.[PersonId] = P2.[Id]
    INNER JOIN [Group] AS F2 ON F2.[Id] = FM2.[GroupId]
WHERE
    F.[GroupTypeId] = @FamilyTypeId
    AND F2.[GroupTypeId] = @FamilyTypeId
    AND F.[Id] = F2.[Id]
    AND dbo.ufnCrm_GetAge(P.[BirthDate]) < 18
    AND dbo.ufnCrm_GetAge(P2.[BirthDate]) >= 18
    AND P.[Email] IS NOT NULL
    AND P.[Email] != ''
;
-- Find all child relationship Person records that have the same e-mail as their parent.
DECLARE @KnownRelationshipTypeId int = (SELECT [Id] FROM [GroupType] WHERE [Guid] = 'E0C5A0E2-B7B3-4EF4-820D-BBF7F9A374EF');
DECLARE @OwnerRelationshipId int = (SELECT [Id] FROM [GroupTypeRole] WHERE [Guid] = '7BC6C12E-0CD1-4DFD-8D5B-1B35AE714C42');
DECLARE @ChildRelationshipId int = (SELECT [Id] FROM [GroupTypeRole] WHERE [Guid] = 'F87DF00F-E86D-4771-A3AE-DBF79B78CF5D');
DECLARE @StepChildRelationshipId int = (SELECT [Id] FROM [GroupTypeRole] WHERE [Guid] = 'EFD2D6D1-A407-4EFB-9086-5DF1F19B7D93');
DECLARE @ChildRelationshipEmailPersonIds TABLE ([Id] int);

INSERT INTO @ChildRelationshipEmailPersonIds
SELECT DISTINCT P2.[Id]
FROM
    [Person] AS P
    INNER JOIN [GroupMember] AS FM ON FM.[PersonId] = P.[Id]
    INNER JOIN [Group] AS KR ON KR.[Id] = FM.[GroupId]
    INNER JOIN [Group] AS KR2 ON KR2.[Id] = KR.[Id]
    INNER JOIN [GroupMember] AS FM2 ON FM2.[GroupId] = KR2.[Id]
    INNER JOIN [Person] AS P2 ON P2.[Id] = FM2.[PersonId]
WHERE
    KR.[GroupTypeId] = @KnownRelationshipTypeId
    AND KR2.[GroupTypeId] = @KnownRelationshipTypeId
    AND KR.[Id] = KR2.[Id]
    AND P.[Id] != P2.[Id]
    AND FM.[GroupRoleId] = @OwnerRelationshipId
    AND ( FM2.[GroupRoleId] = @ChildRelationshipId OR FM2.[GroupRoleId] = @StepChildRelationshipId )
    AND P.[Email] = P2.[Email]
    AND P.[Email] IS NOT NULL
    AND P.[Email] != ''
;
-- Combine both lists into one.
DECLARE @PersonIds TABLE ([Id] int, [OldValue] varchar(100))
INSERT INTO @PersonIds
SELECT DISTINCT IQ.[Id], NULL
FROM (
    SELECT [Id] FROM @ChildEmailPersonIds
    UNION
    SELECT [Id] FROM @ChildRelationshipEmailPersonIds
) AS IQ
ORDER BY IQ.[Id]
;
-- Add the current (old) email to the list of people.
UPDATE P
SET P.[OldValue] = P2.[Email]
FROM
    @PersonIds AS P
    INNER JOIN [Person] AS P2 ON P2.[Id] = P.[Id]
;
-- Add History records showing that we are removing the email.
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
    ,'Email'
    ,P.OldValue
    ,0
FROM @PersonIds AS P
;
-- Remove the email addresses.
UPDATE [Person] SET [Email] = '' WHERE [Id] IN (SELECT [Id] FROM @PersonIds);
```
