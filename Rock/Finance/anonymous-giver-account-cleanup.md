---
tags:
    - language/sql
    - type/cleanup
date created: 2020-12-04 23:24:00
date modified: 2022-01-04 09:45:28
---

# Anonymous Giver Account Cleanup

Delete any saved accounts from Anonymous Giver that are also associated with another person.

## Query

```sql
DELETE
FROM [FinancialPersonBankAccount]
WHERE
    [AccountNumberSecured] IN (
        --Acccounts that have more than 1 person assocciated...
        SELECT [AccountNumberSecured]
        FROM [FinancialPersonBankAccount]
        GROUP BY [AccountNumberSecured]
        HAVING COUNT(1) > 1
    )
    AND [PersonAliasId] IN (
        --...and one of those people is anonymous giver
        SELECT [Id]
        FROM [PersonAlias]
        WHERE [PersonId] = 2
    )
```
