---
tags:
    - language/lava
    - type/utility
    - topic/grid
date created: 2022-01-01 22:19:05
date modified: 2022-01-01 22:28:12
---

# Reset Grid-Size Preference

Occasionally someone will decide to crank a particularly complicated grid up to 5,000 items and cause the page to timeout. When that happens, they can't load that page anymore since that grid size is saved to their personal preferences.

This lava will allow you to reset the grid size of a specified block for a specified person.

The easiest way to run this is using the Lava Tester plugin. You can select the person, then run the lava to do the reset.

## Lava

Replace the ??? with BlockId of the problem grid. If you don't know the BlockId, you can open the block settings and grab it from the header.

`{{ Person.Id | DeleteUserPreference:'grid-page-size-preference_???' }}`  
