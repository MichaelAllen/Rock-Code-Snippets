---
tags:
    - language/sql
    - type/reporting
date created: 2021-12-08 10:41:57
date modified: 2023-03-08 10:54:19
---

# Giving Totals Per Unit & Account in Date Range

Lists every giving unit that gave in a specified date range (Sunday Dates), and how much was given to each account.

1 Row per giving unit, with a column per account

## Query

```sql
DECLARE @StartDate date = '2022-01-01';
DECLARE @EndDate date = '2022-12-31';

DROP TABLE IF EXISTS #TempData;

CREATE TABLE #TempData(
    [GivingId] varchar(30)
    ,[Name] varchar(max)
    ,[Account] varchar(max)
    ,[Ammount] decimal
);

INSERT INTO #TempData(
    [GivingId]
    ,[Account]
    ,[Ammount]
)
SELECT
    p.[GivingId]
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
    [GivingId]
    ,dbo.ufnCrm_GetFamilyTitleFromGivingId( [GivingId] ) AS [Name]
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
