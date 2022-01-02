---
tags:
    - language/js
    - type/utility
date created: 2020-12-01 22:37:24
date modified: 2022-01-01 18:16:37
---

# Run Javascript on Page Load

There are 2 options here. Which one to use depends on what you are trying to do. I tent to use option 1 as a default, and only switch to option 2 if I need to have my code run on every postback (ex: modifying a control on the page that changes on postback)

## Option 1 - jQuery

This will execute once the page has finished loading. It _will not_ run on postbacks.

```js
$(function(){
    // Do some stuff
});
```

## Option 2 - ASP.Net Load Hook

This will execute once the page has finished loading. It _will_ also run on postbacks.

```js
Sys.Application.add_load(function () {
    // Do some stuff
});
```
