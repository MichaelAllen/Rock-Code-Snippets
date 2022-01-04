---
tags:
    - language/sql
    - type/reporting
date created: 2020-12-04 23:24:00
date modified: 2022-01-01 21:48:05
---

# Checkin Config

> Credit: Chris @ Life.Church

This will list out all of your checkin configs, areas, and groups.

_NOTE: It doesn't show groups that are multiple group types deep_

## Query

```sql
DECLARE
    @ConfigPurposeID int
    ,@FilterPurposeID int;
    
SELECT @ConfigPurposeID = [Id] FROM [DefinedValue] DV WHERE DV.[Guid] = '4A406CB0-495B-4795-B788-52BDFDE00B01';
SELECT @FilterPurposeID = [Id] FROM [DefinedValue] DV WHERE DV.[Guid] = '6BCED84C-69AD-4F5A-9197-5C0F9C02DD34';;

WITH CheckInAreas AS 
(
    SELECT
        PT.[Id] 'ConfigId'
        ,CT.[Id] 'AreaId'
        ,PT.[Name] 'Config'
        ,PT.[Order] 'ConfigSort'
        ,CT.[Name] 'Area'
        ,CT.[Order] 'AreaSort'
    FROM
        [GroupTypeAssociation] GTA
        INNER JOIN [GroupType] PT ON GTA.[GroupTypeId] = PT.[Id]
        INNER JOIN [GroupType] CT ON GTA.[ChildGroupTypeId] = CT.[Id]
    WHERE
        PT.[GroupTypePurposeValueId] = @ConfigPurposeID
        AND CT.[GroupTypePurposeValueId] <> @FilterPurposeID

    UNION ALL

    SELECT
        CA.[ConfigId]
        ,GTA.[ChildGroupTypeId] 'AreaId'
        ,CA.[Config]
        ,CA.[ConfigSort]
        ,CT.[Name] 'Area'
        ,CT.[Order] 'AreaSort'
    FROM
        [GroupTypeAssociation] GTA
        INNER JOIN [CheckInAreas] CA ON CA.[AreaId] = GTA.[GroupTypeId]
        INNER JOIN [GroupType] CT ON GTA.[ChildGroupTypeId] = CT.[Id] 
    WHERE
        CT.[GroupTypePurposeValueId] <> @FilterPurposeId
        AND GTA.[GroupTypeId] <> GTA.[ChildGroupTypeId]
)
SELECT
    A.[ConfigId]
    ,A.[AreaId]
    ,G.[Id] 'GroupId'
    ,A.[Config]
    ,A.[Area]
    ,G.[Name] 'Group'
FROM
    CheckInAreas A
    INNER JOIN [Group] G ON A.[AreaId] = G.[GroupTypeId]
WHERE G.[IsActive] = 1
ORDER BY 
    A.[ConfigSort]
    ,A.[AreaSort]
    ,G.[Order]
```
