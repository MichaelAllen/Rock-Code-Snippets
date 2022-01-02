---
tags:
    - language/sql
    - domain/workflow
    - type/utility
    - type/cleanup
date created: 2020-12-27 10:23:48
date modified: 2022-01-01 18:08:49
---

# Round Robin Reassign Workflows

This script will reassign all active instance of a workflow to a list of people in a round-robin fashion. It assigns the activity, and also updates a person attribute in the workflow to match the assignment.

## Needed Information

- List of `PersonId`s for assignment
- The `ActivityTypeId` that you want to assign
- The `AttributeId` that you want to set

## Query

```sql
DECLARE @Workers TABLE ( [Id] int NOT NULL identity(1,1), [PersonId] int, [PrimaryAliasId] int, [PrimaryAliasGuid] uniqueidentifier );
DECLARE @ToUpdate TABLE( [Id] int NOT NULL identity(1,1), [ActivityId] int,  [AttributeId] int );

-- Replace this with the ID of your activity type that you need to assign
-- Use this to help find that Id: SELECT [Id], [Name] FROM [WorkflowActivityType] WHERE [WorkflowTypeId] = 141;
DECLARE @ActivityTypeId int = 364;

-- Replace this with the ID of the attribute that holds the person
DECLARE @AttributeId int = 9501;

-- Put your workers PersonIDs here. They have to be wrapped in ()
INSERT INTO @Workers ([PersonId]) VALUES (515), (50532), (6992);

/* Calculate the info we'll need later */
DECLARE @NumWorkers int = ( SELECT COUNT(1) FROM @Workers );

UPDATE w
SET
    w.[PrimaryAliasId] = pa.[Id]
    ,w.[PrimaryAliasGuid] = pa.[Guid]
FROM
    @Workers w
    JOIN [PersonAlias] pa
        ON w.[PersonId] = pa.[PersonId]
        AND w.[PersonId] = pa.[AliasPersonId]
;

INSERT INTO @ToUpdate ( [ActivityId], [AttributeId] )
SELECT
    act.[Id]
    ,av.[Id]
FROM
    [WorkflowActivity] act
    LEFT JOIN [AttributeValue] av
        ON act.[WorkflowId] = av.[EntityId]
        AND av.[AttributeId] = @AttributeId
WHERE
    act.[ActivityTypeId] = @ActivityTypeId
    AND act.[CompletedDateTime] IS NULL
;

BEGIN TRANSACTION

/* Update the activity assignment */
UPDATE act
    SET act.[AssignedPersonAliasId] = w.[PrimaryAliasId]
FROM
    @ToUpdate upd
    JOIN @Workers w ON @NumWorkers - ( upd.[Id] % @NumWorkers ) = w.[Id]
    JOIN [WorkflowActivity] act ON upd.[ActivityId] = act.[Id]
;

/* Update the attribute */
UPDATE av
    SET av.[Value] = w.[PrimaryAliasGuid]
FROM
    @ToUpdate upd
    JOIN @Workers w ON @NumWorkers - ( upd.[Id] % @NumWorkers ) = w.[Id]
    JOIN [AttributeValue] av ON upd.[AttributeId] = av.[Id]
;

COMMIT TRANSACTION
```
