---
tags:
    - language/sql
    - domain/finance
    - type/reporting
date created: 2020-12-04 23:24:00
date modified: 2022-01-01 11:40:46
---

# Lapsed Givers

> Originally shared by Josh Crews (from Simple Donation) at RX2019
> Modified by Micheal Allen to consider GivingLeaderId vs everyone individually

This SQL searches your Rock database for "recent lapsed givers".

The definition of a recent lapsed giver is someone who gives $3,000 / yr, who has given less than $100 in the past 8 weeks, _who was giving normally this same 8-week period last year_.

That last clause is key, because many people give a lot, but can have non-giving periods. Maybe they take the summer off. Maybe the winter off. Maybe they give a large amount in December, then don't start again til March. Without comparing their giving to the same period last year it's hard to know.

## Query

```sql
DECLARE @ANNUAL_GIVING_THRESHOLD AS INTEGER = 3000;
DECLARE @MAXIMUM_RECENT_GIVING_DOLLARS AS INTEGER = 100;
DECLARE @RECENT_PERIOD_WEEKS AS TINYINT = 8;

SELECT
    Person.Id
   ,Person.NickName
   ,Person.LastName
   ,Person.Email
   ,FORMAT( PastYear.Amount, 'c2' ) 'Past Year'
   ,FORMAT( SamePeriodLastYear.Amount, 'c2' ) 'Same Period Last Year'
   ,FORMAT( RecentPeriod.Amount, 'c2' ) 'Recent Period'
   ,FinancialScheduledTransactions.Count 'Active Scheduled Transactions'
FROM [Person]
    LEFT JOIN (
        SELECT
            SUM( td.Amount ) 'Amount'
           ,P.GivingLeaderId 'PersonId'
        FROM
            FinancialTransaction t
            INNER JOIN FinancialTransactionDetail td ON t.Id = td.TransactionId
            INNER JOIN PersonAlias pa ON t.AuthorizedPersonAliasId = pa.Id
            INNER JOIN Person p ON pa.PersonId = p.Id
        WHERE
            t.TransactionTypeValueId != 54
            AND t.TransactionDateTime < DATEADD( year, -1, getdate() )
            AND t.TransactionDateTime > DATEADD( week, -1 * @RECENT_PERIOD_WEEKS, DATEADD( year, -1, getdate() ) )
        GROUP BY p.GivingLeaderId
    ) [SamePeriodLastYear] ON SamePeriodLastYear.PersonId = Person.Id
    LEFT JOIN (
        SELECT
            SUM( td.Amount ) 'Amount'
           ,P.GivingLeaderId 'PersonId'
        FROM 
            FinancialTransaction t
            INNER JOIN FinancialTransactionDetail td ON t.Id = td.TransactionId
            INNER JOIN PersonAlias pa ON t.AuthorizedPersonAliasId = pa.Id
            INNER JOIN Person p ON pa.PersonId = p.Id
        WHERE
            t.TransactionTypeValueId != 54
            AND t.TransactionDateTime > DATEADD( week, -1 * @RECENT_PERIOD_WEEKS, DATEADD( year, -1, getdate() ) )
        GROUP BY p.GivingLeaderId
    ) [PastYear] ON PastYear.PersonId = Person.Id
    LEFT JOIN (
        SELECT
            SUM( td.Amount ) 'Amount'
           ,P.GivingLeaderId 'PersonId'
        FROM
            FinancialTransaction t
            INNER JOIN FinancialTransactionDetail td ON t.Id = td.TransactionId
            INNER JOIN PersonAlias pa ON t.AuthorizedPersonAliasId = pa.Id
            INNER JOIN Person p ON pa.PersonId = p.Id
        WHERE
            t.TransactionTypeValueId != 54
            AND t.TransactionDateTime > DATEADD( week, -1 * @RECENT_PERIOD_WEEKS, getdate() )
        GROUP BY p.GivingLeaderId
    ) [RecentPeriod] ON RecentPeriod.PersonId = Person.Id
    LEFT JOIN (
        SELECT
            COUNT( fst.Id ) 'Count'
           ,P.GivingLeaderId 'PersonId'
        FROM
            FinancialScheduledTransaction fst
            INNER JOIN PersonAlias pa ON fst.AuthorizedPersonAliasId = pa.Id
            INNER JOIN Person p ON pa.PersonId = p.Id
        WHERE fst.IsActive = 1
        GROUP BY p.GivingLeaderId
    ) [FinancialScheduledTransactions] ON FinancialScheduledTransactions.PersonId = Person.Id    
WHERE
    PastYear.Amount IS NOT NULL
    AND PastYear.Amount > @ANNUAL_GIVING_THRESHOLD
    AND ( RecentPeriod.Amount < @MAXIMUM_RECENT_GIVING_DOLLARS OR RecentPeriod.Amount IS NULL )
    AND ( SamePeriodLastYear.Amount / PastYear.Amount ) > ( ( @RECENT_PERIOD_WEEKS * 1.0 ) / 60 ) -- and last year this period they gave a normal amount for them
    --AND FinancialScheduledTransactions.Count IS NULL
ORDER BY LastName
```
