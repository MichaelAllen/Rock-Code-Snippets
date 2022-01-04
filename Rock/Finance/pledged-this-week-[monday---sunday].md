---
tags:
    - language/sql
    - type/reporting
    - topic/pledge
date created: 2020-12-04 23:24:00
date modified: 2022-01-04 10:10:07
---

# Pledged Last Week

Return a list of everyone that created a new pledge last week. (Monday - Sunday by default. See note below)

_Note: This query uses the `GetPreviousSundayDate` function. That means that it will respect your configured "Starting Day of Week" under "Admin Tools > System Settings > System Configuration"._

## Query

```sql
DECLARE @PledgeAccountID int = 59; --Which account to look at?

SELECT
    CONCAT_WS( ' '
        ,p.FirstName
        ,NULLIF( p.MiddleName, '' )
        ,p.LastName
        ,( SELECT Value FROM [DefinedValue] WHERE ID = p.SuffixValueId )
    ) 'FormalName'
    ,CONCAT_WS( ' '
        ,p.NickName
        ,p.LastName
        ,( SELECT Value FROM [DefinedValue] WHERE ID = p.SuffixValueId )
    ) 'FullName'
    ,p.NickName
    ,CONCAT_WS( ' '
        ,l.Street1
        ,NULLIF( l.Street2, '' )
    ) 'Street'
    ,COALESCE( l.City, '' ) 'City'
    ,COALESCE( l.State, '' ) 'State'
    ,COALESCE( l.PostalCode, '' ) 'PostalCode'
FROM
    [FinancialPledge] fp
    JOIN [PersonAlias] pa ON fp.PersonAliasId = pa.Id
    JOIN [Person] p ON pa.PersonId = p.Id
    JOIN [Group] g ON p.PrimaryFamilyId = g.Id
    LEFT JOIN [GroupLocation] gl
        ON g.Id = gl.GroupId
        AND gl.GroupLocationTypeValueId = 19 --Home
    LEFT JOIN [Location] l
        ON gl.LocationId = l.Id
        AND l.ISActive = 1
WHERE
    fp.AccountId = @PledgeAccountID
    AND fp.CreatedDateTime BETWEEN --Monday through Sunday (inclusive)
        DATEADD( day, -6, dbo.[ufnUtility_GetPreviousSundayDate]() )
        AND dbo.[ufnUtility_GetPreviousSundayDate]()
ORDER BY p.LastName, p.FirstName
```
