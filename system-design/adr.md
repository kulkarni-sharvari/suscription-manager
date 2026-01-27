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

## Status
Proposed

## System design
[Database Connection Diagrams](./architecture-diagrams/database-connection-security-flow.md)


## Context
The Subscription Manager requires persistent storage for:
- User account data (credentials, email, MFA secrets)
- Subscription records (11 fields including sensitive links and payment methods)
- Audit logs (authentication events, data changes)
- Session data (temporary authentication state)

Data Characteristics
- Structured data with well-defined relationships (users → subscriptions, users → audit_logs)
- Transactional integrity required for subscription updates and status changes
- Sensitive data including passwords, payment method references, cancellation/renewal URLs
- Query patterns involve filtering by status, date ranges, and user ownership
- Expected volume: Moderate (hundreds of subscriptions per user, thousands of users at scale)

Requirements

- ACID compliance for data consistency during concurrent updates
- Referential integrity to maintain user-subscription relationships
- Encryption support for data at rest and in transit
- Docker compatibility for containerized deployment
- Development-to-production parity (same DB in localhost and production)
## Decision
We will use PostgreSQL 15+ as our primary database.\
Implementation Details
Security Measures:

#### Data at Rest:

Enable PostgreSQL Transparent Data Encryption (TDE) via pgcrypto extension
Alternatively, use filesystem-level encryption (LUKS on Linux, FileVault on macOS)


#### Data in Transit:

Require SSL/TLS connections between Spring Boot and PostgreSQL
Configure spring.datasource.url with sslmode=require
Use certificate validation in production


#### Sensitive Data Handling:

- Passwords: Hashed using BCrypt (Spring Security default, cost factor 12)
- MFA secrets: Encrypted using AES-256-GCM before storage
- Payment method strings: Encrypted using AES-256-GCM with application-level keys
- Cancellation/renewal URLs: Encrypted or stored as-is depending on sensitivity assessment


#### Credential Management:

- Development: Spring profiles with application-dev.yml (gitignored)
- Production: Environment variables via Docker secrets or cloud provider - secret managers
- Database credentials never hardcoded in source code


#### Connection Security:

- Use connection pooling (HikariCP, Spring Boot default) with minimum connections
- Limit database user permissions (principle of least privilege)
- Separate users for application vs. migrations



#### Docker Strategy:

- Use official postgres:15-alpine image for lightweight deployment
- Mount volumes for data persistence: /var/lib/postgresql/data
- Network isolation: Spring Boot and PostgreSQL in same Docker network,
- PostgreSQL not exposed externally
- Health checks configured for container orchestration

## Consequences
1. Strong Security Foundation

- Built-in encryption options and SSL support\
- Row-level security (RLS) available for future multi-tenancy needs\
- Mature audit logging capabilities

2. Development Velocity

- Excellent Spring Data JPA integration with minimal configuration
- Hibernate ORM maps cleanly to relational schema
- Rich SQL feature set (JSON columns, full-text search if needed later)

3. Production Readiness

- Battle-tested in production environments globally
- Robust backup and recovery tools (pg_dump, pg_restore, WAL archiving)
- Proven scalability for applications of this size

4. Docker Compatibility

- Official Docker images available and well-maintained
- Easy to spin up local development environment: docker-compose up
- Development-production parity achieved

5. Operational Simplicity

- Same database for localhost, Docker, and cloud deployments
- Standard SQL skills applicable throughout
- Strong tooling (pgAdmin, DBeaver, CLI tools)

Negative
1. Initial Setup Complexity

- Requires Docker or native PostgreSQL installation for local development
- SSL certificate management adds configuration overhead
- More complex than embedded databases like H2 or SQLite

2. Resource Overhead

- PostgreSQL consumes more memory than lightweight alternatives (~100MB+ RAM)
- Docker adds another layer of resource usage in development

3. Encryption Key Management

- Application-level encryption requires secure key storage strategy
- Key rotation procedures must be planned
- Lost keys = permanently inaccessible data

Security Implications
1. Mitigated Risks:

-  SQL Injection: Spring Data JPA uses parameterized queries by default
-  Data breach: Encryption at rest protects against disk theft
-  MITM attacks: SSL/TLS protects credentials in transit
-  Unauthorized access: PostgreSQL role-based access control

2. Residual Risks:

-  Application-level vulnerabilities could still expose decrypted data
-  Backup files must also be encrypted (addressed in backup strategy)
-  Database logs may contain sensitive data (configure PostgreSQL logging carefully)

3. Migration Strategy

- Use Flyway or Liquibase for version-controlled schema migrations
- All schema changes go through migration scripts (no manual ALTER statements)
- Rollback procedures documented for each migration

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