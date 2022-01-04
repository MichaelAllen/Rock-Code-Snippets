---
tags:
    - language/sql
    - type/reporting
date created: 2020-12-04 23:24:00
date modified: 2022-01-01 21:48:05
---

# Count of Giving Units By Month

Returns a count of known and anonymous giving units for every month between the provided start and end dates.

## Query

```sql
DECLARE @StartDate Date = '2014-01-01';
DECLARE @EndDate Date = '2020-12-31';

SELECT * FROM
(    
    -- Distinct count of non-anonymous
    SELECT
        COUNT( DISTINCT p.GivingLeaderId ) 'Units'
        ,FORMAT( ft.TransactionDateTime, 'yyyy-MM-01' ) 'Month'
        ,'Known' AS 'Type'
    FROM
        [FinancialTransaction] ft
        JOIN [PersonAlias] pa ON ft.AuthorizedPersonAliasId = pa.Id
        JOIN [Person] p ON pa.PersonId = p.Id
    WHERE
        ft.transactionTypeValueId = 53
        AND ft.TransactionDateTime BETWEEN LEFT( @StartDate, 10 ) AND LEFT( @EndDate, 10 )
        AND p.Id <> 2 --Anonymous
    GROUP BY FORMAT( ft.TransactionDateTime, 'yyyy-MM-01' )
    
    UNION
    
    -- Individual count of anonymous
    SELECT
        COUNT( p.GivingLeaderId ) 'Units'
        ,FORMAT( ft.TransactionDateTime, 'yyyy-MM-01' ) 'Month'
        ,'Anonymous' AS 'Type'
    FROM
        [FinancialTransaction] ft
        JOIN [PersonAlias] pa ON ft.AuthorizedPersonAliasId = pa.Id
        JOIN [Person] p ON pa.PersonId = p.Id
    WHERE
        ft.transactionTypeValueId = 53
        AND ft.TransactionDateTime BETWEEN LEFT( @StartDate, 10 ) AND LEFT( @EndDate, 10 )
        AND p.Id = 2 --Anonymous
    GROUP BY FORMAT( ft.TransactionDateTime, 'yyyy-MM-01' )

) AS x
PIVOT(
    SUM( Units )
    FOR Type IN ( [Known], [Anonymous] )
) AS y
ORDER BY Month
```
