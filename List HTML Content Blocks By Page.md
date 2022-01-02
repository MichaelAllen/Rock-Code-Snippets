---
tags:
    - language/sql
    - domain/cms
    - type/reporting
date created: 2020-12-01 22:40:59
date modified: 2022-01-01 16:50:44
---

# List HTML Content Blocks By Page

Returns a list of all HTML content blocks on a specified page, along with their current contents.

## Query

```sql
DECLARE @PageId int = 12; --Which page to report on?
DECLARE @HtmlContentBlockType int = ( SELECT [Id] FROM [BlockType] WHERE [Guid] = '19B61D65-37E3-459F-A44F-DEF0089118A3' );

SELECT
    b.[Id] 'BlockId'
    ,b.[Zone]
    --,b.[Order]
    ,b.[Name]
    ,h.[Id] 'HtmlContentId'
    ,h.[Content]
    --,h.[Version]
FROM
    [Block] b
    JOIN [HtmlContent] h ON b.[Id] = h.[BlockId]
WHERE
    b.[BlockTypeId] = @HtmlContentBlockType
    AND b.[PageId] = @PageId
ORDER BY
    b.[Zone]
    ,b.[Order]
    ,h.[Version] DESC
```
