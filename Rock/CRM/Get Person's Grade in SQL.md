---
tags:
    - language/sql
    - type/reporting
date created: 2022-02-28 20:03:05
date modified: 2022-02-28 20:09:15
---

# Get Person's Grade in SQL

Rock stores a person's graduation year, not their grade. It turns out that is is actually quite difficult to convert that value into a grade string since you have to take into account the transition date set in Rock.

## Query

```sql
/* Setup Info */
DECLARE @GradeDefinedTypeId int = ( SELECT [Id] FROM [DefinedType] WHERE [Guid] = '24E5A79F-1E62-467A-AD5D-0D10A2328B4D' );
DECLARE @GradeTransitionDateAttributeId int = ( SELECT [Id] FROM [Attribute] WHERE [Guid] = '265734a6-c888-45b4-a7a5-9a26478306b8' );
DECLARE @ThisYear nvarchar(4) = ( SELECT FORMAT( GETDATE(), 'yyyy' ) );
DECLARE @GradeTransitionDate datetime = ( SELECT CAST( [Value] + '/' + @ThisYear AS datetime ) FROM [AttributeValue] WHERE [AttributeId] = @GradeTransitionDateAttributeId );
  
/* Person Info */
DECLARE @PersonId int = 50532;
DECLARE @GradYear int = ( SELECT [GraduationYear] FROM [Person] WHERE [Id] = @PersonId );
DECLARE @GradeOffset int = ( SELECT dbo.[ufnCrm_GetGradeOffset]( @GradYear, @GradeTransitionDate ) );

SELECT [Description]
FROM [DefinedValue]
WHERE
    [DefinedTypeId] = @GradeDefinedTypeId
    AND [Value] = @GradeOffset
;
```
