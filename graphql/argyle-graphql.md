# Argyle GraphQL Schema

Conceptual GraphQL schema for the Argyle income and employment verification API.
Derived from the Argyle REST API v2 — https://docs.argyle.com/

## Overview

Argyle provides user-consented access to payroll and employment data across
thousands of traditional employers, gig platforms, and government payroll
systems. This GraphQL schema models the core Argyle domain — Links, Accounts,
Employments, Incomes, PayStubs, Gigs, Identity, Webhooks, and Auth — as a
unified graph that mirrors the REST API's resource hierarchy.

## Schema File

[argyle-schema.graphql](argyle-schema.graphql)

## Type Summary

| Category | Types |
|---|---|
| Link | Link, LinkDetails, LinkStatus |
| Account | Account, AccountDetails, AccountSource |
| Platform | Platform, PlatformDetails, PlatformType |
| Employer | Employer, EmployerDetails, EmployerLogo, EmployerName |
| Employment | Employment, EmploymentDetails, EmploymentStatus, EmploymentType, EmploymentUnit |
| Position | Position, PositionDetails, PositionTitle |
| Dates | StartDate, EndDate |
| Income | Income, IncomeDetails, IncomeType, IncomeFrequency, IncomeGross, IncomeNet |
| Deductions | Deductions, DeductionDetails |
| Taxes | Taxes, TaxDetails, TaxSummary, TaxFilingStatus |
| PayStub | PayStub, PayStubDetails, PayPeriod, PayDate, PayStubConnection |
| Gig | Gig, GigDetails, GigPlatform, GigPlatformCategory, GigIncome, VehicleDetails |
| Identity | IdentityDetails, PersonName, PersonAddress, ContactInfo |
| Annual | AnnualSummary |
| Webhooks | Webhook, WebhookEvent, WebhookEventType |
| Auth | APIKey, Token |
| Errors | Error, ErrorCode |
| Pagination | PageInfo |

## Named Types: 62

Link, LinkDetails, LinkStatus, Account, AccountDetails, AccountSource,
Platform, PlatformDetails, PlatformType, Employer, EmployerDetails,
EmployerLogo, EmployerName, Employment, EmploymentDetails, EmploymentStatus,
EmploymentType, EmploymentUnit, Position, PositionDetails, PositionTitle,
StartDate, EndDate, Income, IncomeDetails, IncomeType, IncomeFrequency,
IncomeGross, IncomeNet, Deductions, DeductionDetails, Taxes, TaxDetails,
TaxSummary, TaxFilingStatus, PayStub, PayStubDetails, PayPeriod, PayDate,
PayStubConnection, Gig, GigDetails, GigPlatform, GigPlatformCategory,
GigIncome, VehicleDetails, IdentityDetails, PersonName, PersonAddress,
ContactInfo, AnnualSummary, Webhook, WebhookEvent, WebhookEventType,
APIKey, Token, Error, ErrorCode, PageInfo, Query, Mutation, Subscription

## Key Concepts

### Link
The entry point to the Argyle data flow. A Link session is created server-side
and handed to the Argyle Link SDK on the client. Once a user authenticates with
their employer through the widget, an Account is provisioned and data sync
begins automatically.

### Account
Represents a single connected data source (employer or gig platform) for a
user. One user may have multiple Accounts across different employers. An Account
carries Employments, PayStubs, Gigs, and Identity information.

### Employment
Describes the formal relationship between a worker and an employer: status
(active / terminated), type (full-time / gig / contract), position held,
compensation basis, and tenure dates.

### Income
Normalized compensation for an Employment, decomposed into gross, net,
deductions, and taxes with year-to-date totals.

### PayStub
An individual payroll event. Contains gross/net pay, hours worked, per-period
deductions and taxes, payment method, and a link to the original payroll
document.

### Gig
A single completed task on a gig platform (ride, delivery, shift). Captures
duration, distance, platform-specific income breakdown (base fare, tips,
bonuses, surcharges), and the vehicle used.

### Identity
Employment-reported identity for the account holder including legal name,
address, SSN (masked), date of birth, and contact info — distinct from any
Argyle-collected profile.

### Webhooks
Real-time push notifications when data changes. Supports events across the
full lifecycle: link connection, account status, employment changes, new
pay stubs, gig activities, and document additions.

## Example Query

```graphql
query GetEmployeePayStubs($accountId: UUID!, $from: Date, $to: Date) {
  account(id: $accountId) {
    id
    source {
      name
    }
    identity {
      name {
        fullName
      }
    }
    employments {
      status
      type
      employer {
        name {
          canonical
        }
      }
      startDate {
        date
      }
    }
  }
  payStubs(accountId: $accountId, from: $from, to: $to, limit: 10) {
    totalCount
    pageInfo {
      hasNextPage
      endCursor
    }
    nodes {
      id
      payDate {
        date
      }
      period {
        startDate
        endDate
      }
      details {
        grossPay
        netPay
        hoursWorked
        paymentMethod
      }
      deductions {
        total
        items {
          name
          amount
          preTax
        }
      }
      taxes {
        total
        items {
          name
          amount
          jurisdiction
        }
      }
    }
  }
}
```

## Example Mutation

```graphql
mutation CreateUserLink($userId: String!) {
  createLink(userId: $userId, items: ["employer_search"]) {
    id
    status
    details {
      userToken
    }
    createdAt
  }
}
```

## References

- Argyle REST API v2: https://api.argyle.com/v2
- Argyle Documentation: https://docs.argyle.com/
- API Reference: https://docs.argyle.com/api-reference
- Webhooks Guide: https://docs.argyle.com/api-guide/webhooks
- Sandbox: https://api-sandbox.argyle.com/v2
