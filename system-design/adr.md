# ADR-001: Authentication Strategy

Helping notes -
Cookies vs JWT vs sessions
Security trade-offs of each
Why you chose what you chose

## Status
Proposed / Accepted / Deprecated

## Context
What's the situation forcing this decision?
What constraints exist?

## Decision
What are you doing?

## Consequences
What becomes easier? What becomes harder?
What are the security implications?

# ADR-002: Email Service Selection
Helping notes:
Which email provider and why
How you'll handle email deliverability
Security: SPF, DKIM, DMARC considerations

## Status
Proposed

## Context
What's the situation forcing this decision?
What constraints exist?

## Decision
What are you doing?

## Consequences
What becomes easier? What becomes harder?
What are the security implications?

# ADR-003: Database choice
Helping notes:
Relational vs NoSQL
Why it matters for your data model
Encryption at rest considerations

## Status
Proposed / Accepted / Deprecated

## Context
What's the situation forcing this decision?
What constraints exist?

## Decision
What are you doing?

## Consequences
What becomes easier? What becomes harder?
What are the security implications?

# ADR-004: Handling sensitive data
Helping notes:
What data is considered sensitive
Where you'll store payment methods, cancellation links
Encryption strategy

## Status
Proposed / Accepted / Deprecated

## Context
What's the situation forcing this decision?
What constraints exist?

## Decision
What are you doing?

## Consequences
What becomes easier? What becomes harder?
What are the security implications?

# ADR-005: Scheduled job architecture
Helping notes:
Cron job vs message queue vs cloud scheduler
Reliability and failure handling
How you ensure emails go out even if one fails

## Status
Proposed / Accepted / Deprecated

## Context
What's the situation forcing this decision?
What constraints exist?

## Decision
What are you doing?

## Consequences
What becomes easier? What becomes harder?
What are the security implications?

# ADR-006: Email link security
Helping notes:
How you'll generate secure "View & Manage" tokens
Time-to-live for links
Preventing replay attacks

## Status
Proposed / Accepted / Deprecated

## Context
What's the situation forcing this decision?
What constraints exist?

## Decision
What are you doing?

## Consequences
What becomes easier? What becomes harder?
What are the security implications?