---
tags:
    - language/sql
    - domain/fianance
    - type/reporting
date created: 2020-12-01 22:42:38
date modified: 2022-01-01 18:32:07
---

# Yearly Giving Units

For every year between the specified years, return the number of giving units that have given more than the specified minimum amount. If you don't want a minimum amount, you can set it to 0.

## Query

```sql
DECLARE @Years nvarchar(9) = '2018-2020';
DECLARE @Minimum decimal = 200.00; -- How much must they give to be counted?

WITH cte_totals AS (
    SELECT
        p.GivingLeaderId
        ,DATEPART( YEAR, ft.SundayDate ) 'Year'
        ,SUM( ftd.Amount ) 'Total'
    FROM
        [FinancialTransaction] ft
        JOIN [FinancialTransactionDetail] ftd ON ft.Id = ftd.TransactionId
        JOIN [PersonAlias] pa ON ft.AuthorizedPersonAliasId = pa.Id
        JOIN [Person] p ON pa.PersonId = p.Id
    WHERE
        ft.transactionTypeValueId = 53
        AND p.Id <> 2 --Exclude Anonymous Giver
        AND DATEPART( YEAR, ft.SundayDate ) BETWEEN LEFT( @Years, 4 ) AND RIGHT( @Years, 4 )
    GROUP BY
        p.GivingLeaderId
        ,DATEPART( YEAR, ft.SundayDate )
    HAVING
        SUM( ftd.Amount ) >= @Minimum
)
SELECT
    Year
    ,COUNT( 1 ) 'Units'
FROM cte_totals
GROUP BY Year
ORDER BY Year ASC
```
