---
tags:
    - language/sql
    - domain/connection
    - type/cleanup
date created: 2021-01-11 12:33:27
date modified: 2022-01-01 11:14:31
---

# Bulk Update Connection Status

First, you will want to run a select to make sure it is finding all the requests you want to change, and none that you don't.

## Query

```sql
DECLARE @ConnectionOpportunityId int = 4; -- Which Opportunity?

SELECT
    cr.[CreatedDateTime]
    ,p.[FirstName]
    ,p.[LastName]
    ,cr.[Comments]

FROM
    [ConnectionRequest] cr
    JOIN [PersonAlias] pa ON cr.[PersonAliasId] = pa.[Id]
    JOIN [Person] p ON pa.[PersonId] = p.[Id]

WHERE
    cr.[ConnectionOpportunityId] = @ConnectionOpportunityId
    AND cr.[ConnectionState] = 0 --active

```

Once you have verified that it found the right requests, you can run this update to make the change.

```sql
DECLARE @ConnectionOpportunityId int = 4; -- Which Opprotunity?

UPDATE [ConnectionRequest]
SET ConnectionState = 3 --connected

WHERE
    [ConnectionOpportunityId] = @ConnectionOpportunityId
    AND [ConnectionState] = 0 --active
```
