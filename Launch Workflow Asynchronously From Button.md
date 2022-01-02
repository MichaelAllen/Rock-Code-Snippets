---
tags:
    - language/js
    - type/utility
date created: 2020-12-04 23:24:00
date modified: 2022-01-01 23:14:30
---

# Launch Workflow Asynchronously From Button

When you press this button, it will launch a workflow without leaving the current page. Optionally you can pass in attributes to the workflow as well.

## Code

With confirmation:

```html
<script>
function launchWorkflow() {
    Rock.dialogs.confirm( 'Launch workflow?', function( c ) {
        if( c ) {
            $.post( '/api/Workflows/WorkflowEntry/[WorkflowId]?[AttributeKey]=[value]' ).done( function() {
                Rock.dialogs.alert( 'Workflow launched.' );
            });
        }
    });
}
</script>
<button class="btn btn-primary" onclick="launchWorkflow()">Click Me!</button>
```

Without confirmation:

```html
<script>
function launchWorkflow() {
    $.post( '/api/Workflows/WorkflowEntry/[WorkflowId]?[AttributeKey]=[value]' ).done( function() {
        Rock.dialogs.alert( 'Workflow launched.' );
    });
}
</script>
<button class="btn btn-primary" onclick="launchWorkflow()">Click Me!</button>
```
