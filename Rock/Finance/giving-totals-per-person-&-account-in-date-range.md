---
tags:
    - language/sql
    - type/reporting
date created: 2021-12-08 10:41:57
date modified: 2022-01-01 21:48:05
---

# Giving Totals Per Person & Account in Date Range

Lists everyone that gave in a specified date range (Sunday Dates), and how much they gave to each account.

1 Row per person, with a column per account

## Query

```sql
DECLARE @StartDate date = '2020-12-01';
DECLARE @EndDate date = '2021-11-30';

IF OBJECT_ID('#TempData', 'U') IS NOT NULL DROP TABLE #TempData;
CREATE TABLE #TempData(
    [PersonId] int
    ,[Name] varchar(max)
    ,[Account] varchar(max)
    ,[Ammount] decimal
);

INSERT INTO #TempData(
    [PersonId]
    ,[Name]
    ,[Account]
    ,[Ammount]
)
SELECT
    pa.[PersonId]
    ,CONCAT_WS( ', ', p.[LastName], p.[FirstName] )
    ,fa.[Name]
    ,ftd.[Amount]
FROM
    [FinancialTransaction] ft
    JOIN [FinancialTransactionDetail] ftd ON ftd.[TransactionId] = ft.[Id]
    JOIN [FinancialAccount] fa ON ftd.[AccountId] = fa.[Id]
    JOIN [PersonAlias] pa ON ft.[AuthorizedPersonAliasId] = pa.[Id]
    JOIN [Person] p ON pa.[PersonId] = p.[Id]
WHERE
    ft.[TransactionTypeValueId] = 53 --Contribution
    AND ft.[SundayDate] BETWEEN @StartDate AND @EndDate

-- Use Dynamic SQL to Pivot to 1 col per row
DECLARE @DynamicCol nvarchar(max);
DECLARE @sql nvarchar(max);

SELECT @DynamicCol = STUFF( (
    SELECT DISTINCT ', ' + QUOTENAME( [Account] ) FROM #TempData FOR XML PATH ('')
), 1, 2, '' );

SET @Sql = '
SELECT
    [PersonId]
    --,[Name]
    ,'+@DynamicCol+'
FROM (   
    SELECT * FROM #TempData
) AS Src
PIVOT (
    SUM( [Ammount] ) FOR [Account] IN ( '+@DynamicCol+' )
) AS Pvt
ORDER BY
    [Name]
';

PRINT @Sql;
EXEC(@sql);

DROP TABLE #TempData;
```
