---
tags:
    - language/js
    - type/utility
date created: 2024-01-25 20:04:07
date modified: 2024-01-25 20:38:53
---

# Obsidian Mutation Observer

If you are trying to make changes to the DOM of an Obsidian control, you will have to use a mutation observer.

> [!Warning]
> Because the observer fires on any change, you need to be careful about having a stop condition. Otherwise, your change will trigger the observer again; potentially causing an infinite loop.

## Example 1 - Registration Block

I want to change the text "How many ___ will you be registering?" to "How many ___ will you be purchasing?"

```liquid
<script>
function updateTitle() {
    var title = $('.registrationentry-intro h1');
    var needsUpdate = title.text().includes('registering');
    if (title && needsUpdate) {
        title.text(title.text().replace('registering','purchasing'));
    }
}
const observer = new MutationObserver(updateTitle);

$(function() {
    updateTitle();
    observer.observe(document, { childList: true, subtree: true });
});
</script>
```

## Example 2 - Add "Gross, Fee, Net" Table to Batch Detail Screen

I want to add a summary table to the batch detail page that shows me the Gross, Fee, and Net for each account in the batch.

```liquid
{% capture CustomDetails %}
[
{% for Transaction in Context.FinancialBatch.Transactions %}
    {% for Detail in Transaction.TransactionDetails %}
    {
        'AccountName' : {{ Detail.Account.Name | ToJSON }},
        'Amount' : {{ Detail.Amount | AsDecimal }},
        'Fee' : {{ Detail.FeeAmount | Default:'0' | AsDecimal }}
    },
    {% endfor %}
{% endfor %}
]
{% endcapture %}
{% assign CustomDetails = CustomDetails | FromJSON | GroupBy:'AccountName' %}
<script>
function addTable() {
    var updated = ($('.financial-batch-detail .grid-table').first().parent().html() ?? '').includes("Fee");
    if (! updated) {
        var tableContent = '' +
'<table class="grid-table table table-auto">' + 
    '<tbody>' +
        '<tr>' +
            '<th align="left">Account</th>' +
            '<th align="right">Gross</th>' +
            '<th align="right">Fee</th>' +
            '<th align="right">Net</th>' +
        '</tr>' +
{%- for Account in CustomDetails -%}
    {%- assign AccountParts = Account | PropertyToKeyValue -%}
    {%- assign Name = AccountParts.Key -%}
    {%- assign Gross = AccountParts.Value | Select:'Amount' | Sum -%}
    {%- assign Fee = AccountParts.Value | Select:'Fee' | Sum -%}
        '<tr>' +
            '<td align="left">{{ Name }}</td>' +
            '<td align="right">${{ Gross | Format:'#,##0.00' }}</td>' +
            '<td align="right">$({{ Fee | Format:'#,##0.00' }})</td>' +
            '<td align="right">${{ Gross | Minus:Fee | Format:'#,##0.00'}}</td>' +
        '</tr>' +
{%- endfor -%}
    '</tbody>' +
'</table>';
        $('.financial-batch-detail .grid-table').first().parent().html(tableContent);
    }
}
const observer = new MutationObserver(addTable);

$(function() {
    addTable();
    observer.observe(document.getElementById('bid_2847'), { childList: true, subtree: true });
});
</script>
```
