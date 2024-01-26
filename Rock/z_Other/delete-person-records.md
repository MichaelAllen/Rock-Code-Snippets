---
tags:
    - language/sql
    - type/cleanup
date created: 2024-01-25 18:04:56
date modified: 2024-01-25 20:33:43
---

# Delete Person Records

> Source: David Turner [9 Embers]

Delete person records in bulk. Handles cleaning up date from all of the related tables.

This is primarily used to cleanup after a bot attack.

> [!Danger]
> This will **permanently delete** data from Rock. Do not run this unless you are 100% confident that the SELECT statement is correct and you have made a backup.

# Query

```sql
SET NOCOUNT ON
GO

-- Modify Query below to select people that should be deleted from databse.
CREATE TABLE #PersonTbl ( [Id] int, PRIMARY KEY ([Id]) )
INSERT INTO #PersonTbl
SELECT [Id]
FROM [Person]
WHERE [FirstName] LIKE '%Get BitCoin%'

-- Create Temp Person Alias Id Table
CREATE TABLE #PersonAliasTbl ( [Id] int, PRIMARY KEY ([Id]) )
INSERT INTO #PersonAliasTbl
SELECT [Id]
FROM [PersonAlias]
WHERE
    [AliasPersonId] IN ( SELECT [Id] FROM #PersonTbl )
    OR [PersonId] IN ( SELECT [Id] FROM #PersonTbl )

DECLARE @Msg varchar(200)
DECLARE @Rows int = 1
DECLARE @DeletedRows int = 0
DECLARE @PersonEntityTypeId int = ( SELECT TOP 1 [Id] FROM [EntityType] WHERE [Name] = 'Rock.Model.Person' )

-- Delete Person Viewed Records
SELECT @Rows = 1, @DeletedRows = 0
WHILE ( @Rows > 0 )
BEGIN
    DELETE TOP (10000) V
    FROM
        #PersonAliasTbl PA
        INNER JOIN [PersonViewed] V ON
            V.[TargetPersonAliasId] = PA.[Id]
            OR V.[ViewerPersonAliasId] = PA.[Id]

    SET @Rows = @@ROWCOUNT
    SET @DeletedRows = @DeletedRows + @Rows
    SET @Msg = CONCAT( FORMAT( @DeletedRows, 'N0'), ' Person Viewed records deleted' )
    IF @Rows > 0 RAISERROR( @Msg, 0, 0 ) WITH NOWAIT
END

-- Delete Registrant Records
SELECT @Rows = 1, @DeletedRows = 0
WHILE ( @Rows > 0 )
BEGIN
    DELETE TOP (10000) R
    FROM
        #PersonAliasTbl PA
        INNER JOIN [RegistrationRegistrant] R ON R.[PersonAliasId] = PA.[Id]

    SET @Rows = @@ROWCOUNT
    SET @DeletedRows = @DeletedRows + @Rows
    SET @Msg = CONCAT( FORMAT( @DeletedRows, 'N0'), ' Registrant records deleted' )
    IF @Rows > 0 RAISERROR( @Msg, 0, 0 ) WITH NOWAIT
END

-- Delete Registration Records
SELECT @Rows = 1, @DeletedRows = 0
WHILE ( @Rows > 0 )
BEGIN
    DELETE TOP (10000) R
    FROM
        #PersonAliasTbl PA
        INNER JOIN [Registration] R ON R.[PersonAliasId] = PA.[Id]

    SET @Rows = @@ROWCOUNT
    SET @DeletedRows = @DeletedRows + @Rows
    SET @Msg = CONCAT( FORMAT( @DeletedRows, 'N0'), ' Registration records deleted' )
    IF @Rows > 0 RAISERROR( @Msg, 0, 0 ) WITH NOWAIT
END

-- Delete Interaction Records
SELECT @Rows = 1, @DeletedRows = 0
WHILE ( @Rows > 0 )
BEGIN
    DELETE TOP (10000) I
    FROM
        #PersonAliasTbl PA
        INNER JOIN [Interaction] I ON I.[PersonAliasId] = PA.[Id] 

    SET @Rows = @@ROWCOUNT
    SET @DeletedRows = @DeletedRows + @Rows
    SET @Msg = CONCAT( FORMAT( @DeletedRows, 'N0'), ' Interaction records deleted' )
    IF @Rows > 0 RAISERROR( @Msg, 0, 0 ) WITH NOWAIT
END

-- Delete Person Search Records
SELECT @Rows = 1, @DeletedRows = 0
WHILE ( @Rows > 0 )
BEGIN
    DELETE TOP (10000) S
    FROM
        #PersonAliasTbl PA
        INNER JOIN [PersonSearchKey] S ON S.[PersonAliasId] = PA.[Id] 

    SET @Rows = @@ROWCOUNT
    SET @DeletedRows = @DeletedRows + @Rows
    SET @Msg = CONCAT( FORMAT( @DeletedRows, 'N0'), ' Person Search Key records deleted' )
    IF @Rows > 0 RAISERROR( @Msg, 0, 0 ) WITH NOWAIT
END

-- Delete Person Duplicate Records
SELECT @Rows = 1, @DeletedRows = 0
WHILE ( @Rows > 0 )
BEGIN
    DELETE TOP (10000) PD
    FROM
        #PersonAliasTbl PA
        INNER JOIN [PersonDuplicate] PD ON
            PD.[PersonAliasId] = PA.[Id]
            OR PD.[DuplicatePersonAliasId] = PA.[Id]

    SET @Rows = @@ROWCOUNT
    SET @DeletedRows = @DeletedRows + @Rows
    SET @Msg = CONCAT( FORMAT( @DeletedRows, 'N0'), ' Person Duplicate records deleted' )
    IF @Rows > 0 RAISERROR( @Msg, 0, 0 ) WITH NOWAIT
END

-- Delete Audit Records
SELECT @Rows = 1, @DeletedRows = 0
WHILE ( @Rows > 0 )
BEGIN
    DELETE TOP (10000) A
    FROM
        #PersonAliasTbl PA
        INNER JOIN [Audit] A ON A.[PersonAliasId] = PA.[Id] 

    SET @Rows = @@ROWCOUNT
    SET @DeletedRows = @DeletedRows + @Rows
    SET @Msg = CONCAT( FORMAT( @DeletedRows, 'N0'), ' Audit records deleted' )
    IF @Rows > 0 RAISERROR( @Msg, 0, 0 ) WITH NOWAIT
END

-- Delete Personal Device Records
SELECT @Rows = 1, @DeletedRows = 0
WHILE ( @Rows > 0 )
BEGIN
    DELETE TOP (10000) D
    FROM
        #PersonAliasTbl PA
        INNER JOIN [PersonalDevice] D ON D.[PersonAliasId] = PA.[Id] 

    SET @Rows = @@ROWCOUNT
    SET @DeletedRows = @DeletedRows + @Rows
    SET @Msg = CONCAT( FORMAT( @DeletedRows, 'N0'), ' Personal Device records deleted' )
    IF @Rows > 0 RAISERROR( @Msg, 0, 0 ) WITH NOWAIT
END

-- Delete Communication Recipient Records
SELECT @Rows = 1, @DeletedRows = 0
WHILE ( @Rows > 0 )
BEGIN
    DELETE TOP (10000) R
    FROM
        #PersonAliasTbl PA
        INNER JOIN [CommunicationRecipient] R ON R.[PersonAliasId] = PA.[Id] 

    SET @Rows = @@ROWCOUNT
    SET @DeletedRows = @DeletedRows + @Rows
    SET @Msg = CONCAT( FORMAT( @DeletedRows, 'N0'), ' Communication Recipient records deleted' )
    IF @Rows > 0 RAISERROR( @Msg, 0, 0 ) WITH NOWAIT
END

-- Update all ModifedByPersonAliasId and CreatedByPersonAliasId Field to NULL
RAISERROR( 'Updating all CreatedByPersonAliasId and ModifiedByPersonAliasId fileds to NULL', 0, 0 ) WITH NOWAIT
DECLARE @Sql varchar(max)
DECLARE ForeignKeyCursor INSENSITIVE CURSOR FOR
SELECT CONCAT (
    'UPDATE T',
    'SET [', tac.name, '] = NULL ',
    'FROM',
        ' [', tso.name, '] T ',
        'INNER JOIN #PersonAliasTbl PA ON PA.[Id] = T.[', tac.name, '] '
)
FROM sys.foreign_key_columns kc
    INNER JOIN sys.foreign_keys k ON kc.constraint_object_id = k.object_id
    INNER JOIN sys.all_objects so ON so.object_id = kc.referenced_object_id
    INNER JOIN sys.all_columns rac ON
        rac.column_id = kc.referenced_column_id
        AND rac.object_id = so.object_id
    INNER JOIN sys.all_objects tso ON tso.object_id = kc.parent_object_id
    INNER JOIN sys.all_columns tac ON
        tac.column_id = kc.parent_column_id
        AND tac.object_id = tso.object_id
WHERE so.name = 'PersonAlias'
    AND rac.name = 'Id'
    AND tac.name IN ( 'ModifiedByPersonAliasId', 'CreatedByPersonAliasId' )
OPEN ForeignKeyCursor

FETCH NEXT FROM ForeignKeyCursor INTO @Sql
WHILE (@@FETCH_STATUS <> -1)
BEGIN
    IF (@@FETCH_STATUS = 0)
    BEGIN
        EXEC(@Sql)
    END
    FETCH NEXT FROM ForeignKeyCursor INTO @Sql
END

CLOSE ForeignKeyCursor
DEALLOCATE ForeignKeyCursor

-- Delete Person Alias Records
SELECT @Rows = 1, @DeletedRows = 0
WHILE ( @Rows > 0 )
BEGIN
    DELETE TOP (10000)
    FROM [PersonAlias]
    WHERE [Id] IN ( SELECT [Id] FROM #PersonAliasTbl )

    SET @Rows = @@ROWCOUNT
    SET @DeletedRows = @DeletedRows + @Rows
    SET @Msg = CONCAT( FORMAT( @DeletedRows, 'N0'), ' Person Alias records deleted' )
    IF @Rows > 0 RAISERROR( @Msg, 0, 0 ) WITH NOWAIT
END

DROP TABLE #PersonAliasTbl

-- Delete Login Records
SELECT @Rows = 1, @DeletedRows = 0
WHILE ( @Rows > 0 )
BEGIN
    DELETE TOP (10000)
    FROM [UserLogin]
    WHERE [PersonId] IN ( SELECT [Id] FROM #PersonTbl )

    SET @Rows = @@ROWCOUNT
    SET @DeletedRows = @DeletedRows + @Rows
    SET @Msg = CONCAT( FORMAT( @DeletedRows, 'N0'), ' User Login records deleted' )
    IF @Rows > 0 RAISERROR( @Msg, 0, 0 ) WITH NOWAIT
END

-- Save the groups that these people are members of
CREATE TABLE #GroupTbl ( [Id] int, PRIMARY KEY ([Id]) )
INSERT INTO #GroupTbl
SELECT DISTINCT [GroupId]
FROM [GroupMember]
WHERE [PersonId] IN ( SELECT [Id] FROM #PersonTbl )

-- Delete Group Member Records
SELECT @Rows = 1, @DeletedRows = 0
WHILE ( @Rows > 0 )
BEGIN
    DELETE TOP (10000)
    FROM [GroupMember]
    WHERE [PersonId] IN ( SELECT [Id] FROM #PersonTbl )
    
    SET @Rows = @@ROWCOUNT
    SET @DeletedRows = @DeletedRows + @Rows
    SET @Msg = CONCAT( FORMAT( @DeletedRows, 'N0'), ' Group Member records deleted' )
    IF @Rows > 0 RAISERROR( @Msg, 0, 0 ) WITH NOWAIT
END

-- Delete Person Records
SELECT @Rows = 1, @DeletedRows = 0
WHILE ( @Rows > 0 )
BEGIN
    DELETE TOP (10000)
    FROM [Person]
    WHERE [Id] IN ( SELECT [Id] FROM #PersonTbl )

    SET @Rows = @@ROWCOUNT
    SET @DeletedRows = @DeletedRows + @Rows
    SET @Msg = CONCAT( FORMAT( @DeletedRows, 'N0'), ' Person records deleted' )
    IF @Rows > 0 RAISERROR( @Msg, 0, 0 ) WITH NOWAIT
END

-- Delete Person History Records
SELECT @Rows = 1, @DeletedRows = 0
WHILE ( @Rows > 0 )
BEGIN
    DELETE TOP (10000)
    FROM [History]
    WHERE [EntityTypeId] = @PersonEntityTypeId
    AND [EntityId] IN ( SELECT [Id] FROM #PersonTbl )

    SET @Rows = @@ROWCOUNT
    SET @DeletedRows = @DeletedRows + @Rows
    SET @Msg = CONCAT( FORMAT( @DeletedRows, 'N0'), ' Person History records deleted' )
    IF @Rows > 0 RAISERROR( @Msg, 0, 0 ) WITH NOWAIT
END

-- Delete Person Attribute Values
SELECT @Rows = 1, @DeletedRows = 0
WHILE ( @Rows > 0 )
BEGIN
    DELETE TOP (10000) V
    FROM
        [Attribute] A
        INNER JOIN [AttributeValue] V ON V.[AttributeId] = A.[Id]
    WHERE
        A.[EntityTypeId] = @PersonEntityTypeId
        AND V.[EntityId] IN ( SELECT [Id] FROM #PersonTbl )

    SET @Rows = @@ROWCOUNT
    SET @DeletedRows = @DeletedRows + @Rows
    SET @Msg = CONCAT( FORMAT( @DeletedRows, 'N0'), ' Person Attribute Value records deleted' )
    IF @Rows > 0 RAISERROR( @Msg, 0, 0 ) WITH NOWAIT
END

-- Delete Person Notes
SELECT @Rows = 1, @DeletedRows = 0
WHILE ( @Rows > 0 )
BEGIN
    DELETE TOP (10000) N
    FROM
        [NoteType] T
        INNER JOIN [Note] N ON N.[NoteTypeId] = T.[Id]
    WHERE
        T.[EntityTypeId] = @PersonEntityTypeId
        AND N.[EntityId] IN ( SELECT [Id] FROM #PersonTbl )

    SET @Rows = @@ROWCOUNT
    SET @DeletedRows = @DeletedRows + @Rows
    SET @Msg = CONCAT( FORMAT( @DeletedRows, 'N0'), ' Person Note records deleted' )
    IF @Rows > 0 RAISERROR( @Msg, 0, 0 ) WITH NOWAIT
END

-- Delete Groups that people were in that no longer have any group members
SELECT @Rows = 1, @DeletedRows = 0
WHILE ( @Rows > 0 )
BEGIN
    DELETE TOP (10000)
    FROM [Group]
    WHERE
        [Id] IN ( SELECT [Id] FROM #GroupTbl )
        AND [Id] NOT IN ( SELECT [GroupId] FROM [GroupMember] )

    SET @Rows = @@ROWCOUNT
    SET @DeletedRows = @DeletedRows + @Rows
    SET @Msg = CONCAT( FORMAT( @DeletedRows, 'N0'), ' Group records deleted' )
    IF @Rows > 0 RAISERROR( @Msg, 0, 0 ) WITH NOWAIT
END

DROP TABLE #GroupTbl
DROP TABLE #PersonTbl

SET NOCOUNT OFF
GO
```
