---
tags:
    - type/utility
    - topic/finance
    - topic/security
date created: 2022-08-18 18:53:54
date modified: 2025-11-23 18:17:59
---

# Security Permissions For Check Scanner App

We needed a way to give permissions for someone to use the check scanner but didn't want to give them access to the Rock site. The settings below are the absolute minimum permissions needed for the check scanner app to work fully.

We created a role `APP - Check Scanner` with this narrow scope of permissions so that we can easily assign it to any user that needs access to the scanner app. This role provides all of the needed permissions and doesn't require the user to have any other roles (`RSR - Staff Workers`, `RSR - Finance Worker`, etc.)

## REST Controllers

`Admin Tools > Security > Rest Controllers`

| Controller                  | Method | Path                                          | Verbs      |
| --------------------------- | ------ | --------------------------------------------- | ---------- |
| BinaryFileTypes             | GET    | api/BinaryFileTypes                           | VIEW       |
| Campuses                    | GET    | api/Campuses                                  | VIEW       |
| DefinedTypes                | GET    | api/DefinedTypes                              | VIEW       |
| DefinedValues               | GET    | api/DefinedValues                             | VIEW       |
| ExceptionLogs               | POST   | api/ExceptionLogs/LogException                | VIEW, EDIT |
| FinancialAccounts           | GET    | api/FinancialAccounts                         | VIEW       |
| FinancialBatches            | DELETE | api/FinancialBatches/{0}                      | VIEW, EDIT |
| FinancialBatches            | GET    | api/FinancialBatches                          | VIEW       |
| FinancialBatches            | GET    | api/FinancialBatches/GetControlTotals         | VIEW       |
| FinancialBatches            | POST   | api/FinancialBatches                          | VIEW, EDIT |
| FinancialBatches            | PUT    | api/FinancialBatches/{0}                      | VIEW, EDIT |
| FinancialPaymentDetails     | GET    | api/FinancialPaymentDetails/{0}               | VIEW       |
| FinancialPaymentDetails     | POST   | api/FinancialPaymentDetails/{0}               | VIEW, EDIT |
| FinancialTransactionDetails | GET    | api/FinancialTransactionDetails               | VIEW       |
| FinancialTransactionDetails | POST   | api/FinancialTransactionDetails               | VIEW, EDIT |
| FinancialTransactionDetails | PUT    | api/FinancialTransactionDetails/{0}           | VIEW, EDIT |
| FinancialTransactionImages  | GET    | api/FinancialTransactionImages                | VIEW       |
| FinancialTransactionImages  | POST   | api/FinancialTransactionImages                | VIEW, EDIT |
| FinancialTransactions       | DELETE | api/FinancialTransactions/{0}                 | VIEW, EDIT |
| FinancialTransactions       | GET    | api/FinancialTransactions                     | VIEW       |
| FinancialTransactions       | POST   | api/FinancialTransactions                     | VIEW, EDIT |
| FinancialTransactions       | POST   | api/FinancialTransactions/AlreadyScanned      | VIEW, EDIT |
| FinancialTransactions       | POST   | api/FinancialTransactions/PostScanned         | VIEW, EDIT |
| People                      | GET    | api/People/GetByPersonAliasIs/{personAliasId} | VIEW       |
| People                      | GET    | api/People/GetByUsername/{username}           | VIEW       |

## Entities

`Admin Tools > Security > Entity Administration`

| Entity                                | Verbs              |
| ------------------------------------- | ------------------ |
| Rock.Model.FinancialAccount           | VIEW               |
| Rock.Model.FinancialBatch             | VIEW, EDIT, DELETE |
| Rock.Model.FinancialPaymentDetail     | EDIT               |
| Rock.Model.FinancialTransaction       | VIEW, EDIT         |
| Rock.Model.FinancialTransactionDetail | VIEW               |
| Rock.Model.FinancialTransactionImage  | VIEW               |

## File Types

`Admin Tools > General > File Types`

| File Type         | Verbs      |
| ----------------- | ---------- |
| Transaction Image | VIEW, EDIT |
