---
tags:
    - language/sql
    - type/reporting
    - topic/metrics
date created: 2022-01-29 22:00:34
date modified: 2022-01-29 22:05:31
---

# Recurring Giver Percentage

> Credit: I think part of this sql came from Josh Crews @ SimpleDonation

Return a single number representing the number of "regular givers" (Gave at least 1 time in each of the past 3 months) that have recurring giving setup. This is intended to be used as the source for a metric.

## Query

```sql
DECLARE @Today DATETIME = GETDATE();

SELECT
    CAST(COUNT(RecurringGivers.[PersonId]) AS DECIMAL)
        / SUM(
            CASE
                WHEN
                    LastMonthGivers.[PersonId] IS NOT NULL
                    AND TwoMonthsAgoGivers.[PersonId] IS NOT NULL
                    AND ThreeMonthsAgoGivers.[PersonId] IS NOT NULL
                THEN 1
                ELSE 0
            END
        )
        * 100 AS [Percent]
FROM (
    SELECT
        pa.[PersonId]
    FROM FinancialTransaction [t]
    INNER JOIN PersonAlias [pa]
        ON t.[AuthorizedPersonAliasId] = pa.[Id]
    WHERE t.[TransactionDateTime] > DATEADD(DAY, 1, EOMONTH(@today,-2))
        AND t.[TransactionDateTime] < DATEADD(DAY, 1, EOMONTH(@today,-1))
        AND t.[TransactionTypeValueId] != 54
    GROUP BY
        pa.[PersonId]
) [LastMonthGivers]
LEFT JOIN (
    SELECT
        pa.[PersonId]
    FROM FinancialTransaction [t]
    INNER JOIN PersonAlias [pa]
        ON t.[AuthorizedPersonAliasId] = pa.[Id]
    WHERE t.[TransactionDateTime] > DATEADD(DAY, 1, EOMONTH(@today,-3))
        AND t.[TransactionDateTime] < DATEADD(DAY, 1, EOMONTH(@today,-2))
        AND t.[TransactionTypeValueId] != 54
    GROUP BY
        pa.[PersonId]
) [TwoMonthsAgoGivers]
    ON TwoMonthsAgoGivers.[PersonId] = LastMonthGivers.[PersonId]
LEFT JOIN (
    SELECT
        pa.[PersonId]
    FROM FinancialTransaction [t]
    INNER JOIN PersonAlias [pa]
        ON t.[AuthorizedPersonAliasId] = pa.[Id]
    WHERE t.[TransactionDateTime] > DATEADD(DAY, 1, EOMONTH(@today,-4))
        AND t.[TransactionDateTime] < DATEADD(DAY, 1, EOMONTH(@today,-3))
        AND t.[TransactionTypeValueId] != 54
    GROUP BY
        pa.[PersonId]
) [ThreeMonthsAgoGivers]
    ON ThreeMonthsAgoGivers.[PersonId] = LastMonthGivers.[PersonId]
LEFT JOIN (
    SELECT
        pa.[PersonId]
    FROM FinancialScheduledTransaction [fst]
    INNER JOIN PersonAlias [pa]
         ON fst.[AuthorizedPersonAliasId] = pa.[Id]
    WHERE fst.[IsActive] = 1
    GROUP BY
        pa.[PersonId]
) [RecurringGivers]
    ON RecurringGivers.[PersonId] = LastMonthGivers.[PersonId]
```
