# subscription-manager

#### Contents:
1. [System Context](#system-context)

## System Context

### Target Users
Individuals managing multiple recurring subscriptions across various services (streaming platforms, SaaS tools, memberships, utilities, etc.) who need a centralized system to track subscription details, billing cycles, and renewal dates.

### System Dependencies
#### External Services:

SendGrid - Email delivery service for automated renewal notifications\
PostgreSQL - Relational database for persistent data storage

#### Technology Stack:

Frontend: React.js (web-based user interface)\
Backend: Spring Boot (RESTful API and business logic)\
Client: Modern web browsers (Chrome, Firefox, Safari, Edge)

### Problem Domain
Modern consumers manage 5-15+ active subscriptions on average, with details spread across emails, bank statements, and provider websites. This fragmentation creates:

1. Missed cancellation windows
2. Unexpected renewal charges
3. Lost credentials and payment information
4. Inability to analyze spending patterns

This system consolidates subscription data, automates reminders, and provides lifecycle management to address these pain points.