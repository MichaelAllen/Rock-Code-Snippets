---
tags:
    - language/sql
    - type/reporting
    - topic/media
date created: 2023-06-11 19:09:26
date modified: 2023-06-11 19:25:43
---

# Media Interactions by Content Channel Item

> Credit: Randy Aufrecht

Returns a list of all content channel items in a specified content channel. Includes media watch stats for the media element linked to them.

## Result Columns

| Name                      | Description                                                                        |
| ------------------------- | ---------------------------------------------------------------------------------- |
| Id                        | Content Channel Item Id                                                            |
| MediaId                   | Media Element Id                                                                   |
| Title                     | Content Channel Item Title                                                         |
| MinutesWatched            | Total number of minutes watched                                                    |
| TotalInteractions         | Total number of times someone has interacted with the video                        |
| AvgMinutesPerInteracction | MinutesWatched ÷ TotalInteractions                                                 |
| UniqueUsers               | Number of unique users that interacted with the content (must have been logged in) |
| AvgMinutesPerUser         | MinutesWatched ÷ UniqueUsers                                                       |
| CreatedDateTime           | Created Date Time for the Media Element                                            |

## Query

```sql
/* Messages */
DECLARE @ContentChannelId int = 5;
DECLARE @VideoAttributeId int = 12233;

/* Worhsip Archives - Worship 
DECLARE @ContentChannelId int = 68;
DECLARE @VideoAttributeId int = 12528;
*/
/* Worhsip Archives - Multiviewer 
DECLARE @ContentChannelId int = 68;
DECLARE @VideoAttributeId int = 12523;
*/

DECLARE @MediaEventsInteractionChannelId int = (SELECT [Id] FROM [InteractionChannel] WHERE [Guid] = 'D5B9BDAF-6E52-40D5-8E74-4E23973DF159');

SELECT
    cci.[Id]
    ,me.[Id] 'MediaId'
    ,cci.[Title]
    ,ROUND((SUM(i.[InteractionLength]) / 100 * me.[DurationSeconds]) / 60, 0) 'MinutesWatched'
    ,COUNT(i.[Id]) 'TotalInteractions'
    ,ROUND(((SUM(i.[InteractionLength]) / 100 * me.[DurationSeconds]) / 60) / NULLIF(COUNT(i.[Id]),0), 1) 'AvgMinutesPerInteraction'
    ,COUNT(DISTINCT pa.[PersonId]) 'UniqueUsers-LoggedIn'
    ,ROUND(((SUM(i.[InteractionLength]) / 100 * me.[DurationSeconds]) / 60) / NULLIF(COUNT(DISTINCT pa.[PersonId]),0), 1) 'AvgMinutesPerUser'
    ,me.[CreatedDateTime]
FROM
    [ContentChannelItem] cci
    INNER JOIN [AttributeValue] av ON
        av.[AttributeId] = @VideoAttributeId
        AND cci.[Id] = av.[EntityId]
    INNER JOIN [MediaElement] me ON TRY_CAST(av.[Value] AS UNIQUEIDENTIFIER) = me.[Guid]
    LEFT JOIN [InteractionComponent] ic ON
        ic.[InteractionChannelId] = @MediaEventsInteractionChannelId
        AND ic.[EntityId] = me.[Id]
    LEFT JOIN [Interaction] i ON i.[InteractionComponentId] = ic.[Id]
    LEFT JOIN [PersonAlias] pa ON i.[PersonAliasId] = pa.[Id]
WHERE cci.[ContentChannelId] = @ContentChannelId
GROUP BY
    cci.[Id]
    ,cci.[Title]
    ,cci.[StartDateTime]
    ,me.[Id]
    ,me.[DurationSeconds]
    ,me.[CreatedDateTime]
ORDER BY cci.[StartDateTime] DESC
```
