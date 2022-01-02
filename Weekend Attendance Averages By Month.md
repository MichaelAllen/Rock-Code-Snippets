---
tags:
    - language/sql
    - domain/metrics
    - type/reporting
    - attendance
date created: 2020-12-01 22:39:57
date modified: 2022-01-01 18:27:07
---

# Weekend Attendance Averages By Month

For every month in the specified year, return the average attendance (based on metrics) for each campus.

This can be easily modified to report on monthly averages for any campus partitioned metric.

## Query

### List All Campus Partitions, for Use in the Next Query

```sql
DECLARE @CampusEntityTypeId int = (
    SELECT Id FROM [EntityType]
    WHERE Guid = '00096BED-9587-415E-8AD4-4E076AE8FBF0' --Campus
);

SELECT
    m.Title 'Metric'
    ,mp.Label 'Partition'
    ,mp.Id 'Partition Id'
FROM
    [Metric] m
    JOIN [MetricPartition] mp
        ON mp.MetricId = m.Id
        AND mp.EntityTypeId = @CampusEntityTypeId
ORDER BY m.Title
```

### Get the Data

```sql
DECLARE @Year int = 2018;

WITH MetricValues as (
    --Campuses
    SELECT
       SUM( mv.YValue ) 'Sum'
       ,mvp.EntityId
       ,[dbo].[ufnUtility_GetSundayDate]( mv.MetricValueDateTime ) 'SundayDate'
    FROM
        [MetricValuePartition] mvp
        JOIN [MetricValue] mv ON mvp.MetricValueId = mv.Id
    WHERE
        mvp.MetricPartitionId IN ( 27, 29, 23, 25 ) --Campus Partitions
        AND DATEPART( year, mv.MetricValueDateTime ) >= @Year
    GROUP BY
        [dbo].[ufnUtility_GetSundayDate]( mv.MetricValueDateTime )
        ,mvp.EntityId
    
    UNION

    --ONLINE (Metric has no campus partition)
    SELECT
       SUM( mv.YValue ) 'Sum'
       ,0 'EntityId'
       ,[dbo].[ufnUtility_GetSundayDate]( mv.MetricValueDateTime ) 'SundayDate'
    FROM
        [MetricValue] mv
    WHERE
        mv.MetricId = 40 --Online Headcounts
        AND DATEPART( year, mv.MetricValueDateTime ) >= @Year
    GROUP BY
        [dbo].[ufnUtility_GetSundayDate]( mv.MetricValueDateTime )
)
SELECT
    ROUND( AVG( v.[Sum] ), 0 ) 'Average'
    ,CASE v.EntityId
        WHEN 0 THEN 'Online'
        ELSE ( SELECT Name FROM Campus c WHERE c.Id = v.EntityId )
    END 'Campus'
    ,FORMAT( v.SundayDate, 'yyyy-MM' ) 'Month'
FROM [MetricValues] v
GROUP BY
    EntityId
    ,FORMAT( v.SundayDate, 'yyyy-MM' )
ORDER BY
    'Campus'
    ,'Month' ASC
```
