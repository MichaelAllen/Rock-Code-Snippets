---
tags:
    - language/lava
    - type/reporting
    - topic/entity-commands
    - topic/expression
    - topic/person-alias
date created: 2021-05-04 16:15:05
date modified: 2023-06-11 19:23:11
---

# Person Aliases in Entity Commands

Many things in Rock are linked to a person's alias, not directly to the person. This makes querying for that data through an entity command tricky because you have to check each of the person's aliases to find the data.

_Both of these examples assume that you have a Person entity assigned to `person`_

## Option 1 - Chained ORs

Generate a list of person aliases formatted in a way where they can be used in an entity command.

_Because of the way conditionals are interpreted in entity commands, you will need to make sure that any additional filtering comes before the aliases in the where parameter._

```liquid
{% assign personAliases = person.Aliases | Map:'Id' | Join:'" || PersonAliasId == "' | Prepend:'PersonAliasId =="' | Append:'"' %}

{% step where:'StepTypeId == "6" || StepTypeId == "8" && {{ personAliases }}' %}
    //- Do some stuff
{% endstep %}
```

## Option 2 - Expression Parameter

This is a new option I recently found out about. It makes things like this much easier, but requires the fluid lava engine.

```liquid
{% step where:'StepTypeId == "6" || StepTypeId == "8"' expression:'PersonAlias.PersonId == {{ person.Id }}' %}
    //- Do some stuff
{% endstep %}
```
