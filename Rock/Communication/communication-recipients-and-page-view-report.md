---
tags: 
    - language/sql
    - type/reporting
    - topic/interactions
date created: 2022-09-05 14:54:11
date modified: 2022-09-05 15:07:31
---

# Communication Recipients and Page View Report

Lists everyone who received a specific communication, if the message was sent or opened, and if the recipient viewed a specific page. Since it is looking at interactions to see who visited the page, it relies on the person being logged in or using a tokenized link in the email.

## Page Parameter Filter Block

| Field Type       | Key             | Description                 |
| ---------------- | --------------- | --------------------------- |
| Communication Id | CommunicationId | The ID of the communication |
| Page Id          | PageId          | The Id of the page          |

## Dynamic Data Block

### Parameters

`CommunicationId=0;PageId=0`

### Query

```sql
--DECLARE @CommunicationId int = 54489;
--DECLARE @PageId int = 1066;

DECLARE @ComponentId int = (
    SELECT TOP 1 [Id]
    FROM [InteractionComponent]
    WHERE [EntityId] = @PageId
    ORDER BY [CreatedDateTime] DESC
);

SELECT
    p.[Id]
    ,p.[NickName]
    ,p.[LastName]
    ,[dbo].ufnCrm_GetFamilyTitleFromGivingId( p.[GivingId] ) 'Family'
    ,CONVERT( bit, CASE
        WHEN cr.[Status] IN (1, 4) THEN 1 -- (1 = Sent, 4 = Opened)
        ELSE 0
    END ) 'EmailSent'
    ,CONVERT( bit, CASE
        WHEN cr.[Status] = 4 THEN 1 -- (4 = Opened)
        ELSE 0
    END ) 'EmailOpened'
    ,CONVERT( bit, CASE
        WHEN views.[PersonId] IS NOT NULL THEN 1
        ELSE 0
    END ) 'PageViewed'
    ,CASE
        WHEN views.[PersonId] IS NOT NULL
            THEN CONCAT( 'Last viewed on ', FORMAT( views.[LastViewed], 'dddd, MMM. d' ), ', at ', FORMAT( views.[LastViewed], 'h:mm tt' ) )
        WHEN cr.[Status] NOT IN (1, 4) THEN cr.[StatusNote] -- (1 = Sent, 4 = Opened)
        ELSE ''
    END 'Note'
FROM
    [CommunicationRecipient] cr
    JOIN [PersonAlias] pa ON cr.[PersonAliasId] = pa.[Id]
    JOIN [Person] p ON pa.[PersonId] = p.[Id]
    LEFT JOIN (
        SELECT
            pa.[PersonId]
            ,MAX( I.[InteractionDateTime] ) 'LastViewed'
        FROM
            [Interaction] i
            JOIN [PersonAlias] pa ON i.[PersonAliasId] = pa.[Id]
        WHERE
            I.[InteractionComponentId] = @ComponentId
            AND I.[Operation] = 'View'
        GROUP BY
            pa.[PersonId]
    ) views ON p.[Id] = views.[PersonId]
WHERE cr.[CommunicationId] = @CommunicationId
ORDER BY
    p.[LastName]
    ,p.[NickName]
;
```
