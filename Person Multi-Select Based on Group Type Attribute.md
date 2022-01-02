---
tags:
    - language/lava
    - domain/workflows
    - type/utility
date created: 2020-12-01 22:28:08
date modified: 2022-01-01 23:01:16
---

# Person Multi-Select Based on Group Type Attribute

This allows you to have a nicely formatted list of group members on a workflow form. Similar to what you would see on the group attendance page.

Once the form is submitted, you will have a comma separated list of person alias guids which you can loop over and drop into a Person type attribute.

_Note: The person filling out the form must have view permissions on the group you are using. If that won't work for you, you can re-write the capture statement as an entity command with `securityenabled:'false'` rather than referencing `Group.Members`._

## Needed Attributes:

Group [Group]

- This must have a value set before you display the form.
- You can either specify a default value, or set it with some logic in the workflow.

SelectedPeople [Text]

- After the form is submitted, this will contain a comma-separated list of person alias guids

## Workflow Form

Display the `SelectedPeople` attribute on a form with the following pre and post html.

### Pre-HTML

```liquid
<div class="gm-select">
```

### Post-HTML

```liquid
</div>
{% capture select %}
<div class="controls rockcheckboxlist rockcheckboxlist-horizontal in-columns in-columns-1">
    {% assign group = Workflow | Attribute:'Group','Object' %}
    {% assign members = group.Members | OrderBy:'Person.NickName,Person.LastName' %}
    {% for member in members %}
        <div class="checkbox">
            <label class="checkbox-inline" for="{{ member.Id }}">
                <input id="{{ member.Id }}" type="checkbox" name="{{ member.Id }}" value="{{ member.Person.PrimaryAlias.Guid }}">
                <span class="label-text">
                    <img src="{{ member.Person.PhotoUrl }}&w=80" style="width:80px;border-radius:100%;" />
                    {{ member.Person.FullName }}
                </span>
            </label>
        </div>
    {% endfor %}
</div>
{% endcapture %}
<script>
Sys.Application.add_load( function () {
    // Replace the control
    var select = $.parseHTML( '{{ select | StripNewlines }}' );
    $( '.gm-select input[type=text]' ).hide();
    $( '.gm-select .control-wrapper' ).append( select );
    // Event handler
    $('.gm-select input[type=checkbox]').change( function(){
        var selectedIDs = $('.gm-select input:checked').map( function(){ return this.value } ).get().join( ',' );
        $( '.gm-select input[type=text]' ).val( selectedIDs );
    })
} );
</script>
```
