## Table of Contents
1. [High Level Diagram](high-level-diagram)
2. [Example: User Updates Subscription](example)

## High Level Diagram
```
┌─────────────────────────────────────────────────────────────────┐
│                         User's Browser                          │
│                    (HTTPS - TLS 1.3)                           │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             │ Encrypted HTTP Traffic
                             │ (Credentials, Subscription Data)
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                     Spring Boot Application                      │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │              Security Layer (Spring Security)              │ │
│  │  • Authentication (BCrypt password hashing)                │ │
│  │  • CSRF Protection                                         │ │
│  │  • Session Management                                      │ │
│  │  • Rate Limiting (Bucket4j)                               │ │
│  └───────────────────────────────────────────────────────────┘ │
│                             │                                    │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │              Business Logic Layer                          │ │
│  │  • Input Validation (Hibernate Validator)                 │ │
│  │  • Authorization Checks (User owns subscription?)         │ │
│  │  • Encryption/Decryption Service                          │ │
│  │    - AES-256-GCM for sensitive fields                     │ │
│  │    - Key fetched from environment/secrets manager         │ │
│  └───────────────────────────────────────────────────────────┘ │
│                             │                                    │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │         Data Access Layer (Spring Data JPA/Hibernate)      │ │
│  │  • Prepared Statements (SQL Injection Prevention)         │ │
│  │  • Entity Validation                                       │ │
│  │  • Audit Logging (@CreatedDate, @LastModifiedDate)       │ │
│  └───────────────────────────────────────────────────────────┘ │
│                             │                                    │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │         Connection Pool (HikariCP)                         │ │
│  │  • Min Connections: 5                                      │ │
│  │  • Max Connections: 20                                     │ │
│  │  • Connection Timeout: 30s                                │ │
│  │  • SSL Mode: REQUIRE                                      │ │
│  └───────────────────────────────────────────────────────────┘ │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             │ SSL/TLS Connection
                             │ jdbc:postgresql://db:5432/sub_mgr?
                             │ sslmode=require&sslrootcert=...
                             │
                             │ Credentials from Environment:
                             │ ${SPRING_DATASOURCE_USERNAME}
                             │ ${SPRING_DATASOURCE_PASSWORD}
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                      PostgreSQL Database                         │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │              Authentication Layer                          │ │
│  │  • Role-Based Access Control                              │ │
│  │  • app_user: SELECT, INSERT, UPDATE, DELETE on tables    │ │
│  │  • No SUPERUSER privileges                                │ │
│  └───────────────────────────────────────────────────────────┘ │
│                             │                                    │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │              Data Storage Layer                            │ │
│  │                                                             │ │
│  │  Table: users                                              │ │
│  │  ├─ id (UUID, PK)                                         │ │
│  │  ├─ email (VARCHAR, UNIQUE)                               │ │
│  │  ├─ password_hash (VARCHAR) ← BCrypt hash                │ │
│  │  ├─ mfa_secret_encrypted (TEXT) ← AES-256 encrypted      │ │
│  │  └─ created_at, updated_at                                │ │
│  │                                                             │ │
│  │  Table: subscriptions                                      │ │
│  │  ├─ id (UUID, PK)                                         │ │
│  │  ├─ user_id (UUID, FK → users.id)                        │ │
│  │  ├─ product_name (VARCHAR)                                │ │
│  │  ├─ payment_method_encrypted (TEXT) ← AES-256 encrypted  │ │
│  │  ├─ cancellation_link_encrypted (TEXT) ← AES-256         │ │
│  │  ├─ renewal_link_encrypted (TEXT) ← AES-256              │ │
│  │  └─ next_billing_date, status, etc.                       │ │
│  │                                                             │ │
│  │  Table: audit_logs                                         │ │
│  │  ├─ id (UUID, PK)                                         │ │
│  │  ├─ user_id (UUID, FK)                                    │ │
│  │  ├─ action (VARCHAR) e.g., "LOGIN", "UPDATE_SUB"         │ │
│  │  ├─ ip_address (INET)                                     │ │
│  │  └─ timestamp                                              │ │
│  └───────────────────────────────────────────────────────────┘ │
│                             │                                    │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │              Encryption at Rest (Optional)                 │ │
│  │  • PostgreSQL TDE (pgcrypto extension)                    │ │
│  │  OR                                                         │ │
│  │  • Filesystem-level encryption (LUKS, dm-crypt)           │ │
│  │  OR                                                         │ │
│  │  • Cloud provider encryption (AWS RDS, GCP Cloud SQL)     │ │
│  └───────────────────────────────────────────────────────────┘ │
│                             │                                    │
│                             ▼                                    │
│                    Physical Storage (Disk)                       │
│                    Volume: /var/lib/postgresql/data              │
└─────────────────────────────────────────────────────────────────┘
```

## Example: User Updates Subscription
```
1. Browser Request
   ├─ HTTPS POST to /api/subscriptions/123 \
   ├─ Headers: Cookie: JSESSIONID=xxx; X-CSRF-TOKEN=yyy 
   └─ Body: { "payment_method": "Visa ending 4242", ... }
        │
        ▼
2. Spring Boot: Security Filter Chain
   ├─ [Filter 1] HTTPS Enforcer
   │   └─ Reject if not HTTPS
   ├─ [Filter 2] CSRF Token Validator
   │   └─ Verify X-CSRF-TOKEN matches session
   ├─ [Filter 3] Authentication
   │   └─ Validate JSESSIONID → Load UserDetails
   ├─ [Filter 4] Rate Limiter
   │   └─ Check: User exceeded 100 req/min? → 429 Too Many Requests
   └─ [Filter 5] Authorization
       └─ Does user own subscription 123? → Load from DB
            │
            ▼
3. Controller Layer
   └─ @PreAuthorize("@subscriptionService.isOwner(#id, principal)")
       ├─ Input Validation: @Valid SubscriptionUpdateDTO
       │   └─ Check: payment_method length, format, etc.
       └─ Pass to Service
            │
            ▼
4. Service Layer (Business Logic)
   ├─ Retrieve existing subscription from database
   ├─ Encrypt sensitive field:
   │   └─ encryptedPaymentMethod = AES.encrypt(
   │       "Visa ending 4242", 
   │       getKeyFromEnvironment()
   │   )
   ├─ Update entity fields
   └─ Call repository.save()
        │
        ▼
5. Repository Layer (Spring Data JPA)
   ├─ Generate SQL: UPDATE subscriptions SET 
   │                payment_method_encrypted = ?,
   │                updated_at = ?
   │                WHERE id = ? AND user_id = ?
   ├─ Bind parameters (Prepared Statement)
   │   └─ Prevents SQL injection
   └─ Execute via HikariCP connection pool
        │
        ▼
6. HikariCP Connection Pool
   ├─ Retrieve existing connection OR create new one
   ├─ Connection properties:
   │   └─ jdbc:postgresql://db:5432/sub_mgr?
   │       sslmode=require&
   │       sslrootcert=/path/to/root.crt&
   │       user=${DB_USER}&password=${DB_PASSWORD}
   └─ Send query over SSL/TLS encrypted connection
        │
        ▼
7. PostgreSQL: Authentication
   ├─ Verify SSL certificate
   ├─ Authenticate user credentials
   ├─ Check GRANT permissions:
   │   └─ app_user has UPDATE on subscriptions table?
   └─ Execute query
        │
        ▼
8. PostgreSQL: Query Execution
   ├─ Parse and plan query
   ├─ Acquire row-level lock on subscription record
   ├─ Update row in memory
   ├─ Write to Write-Ahead Log (WAL)
   └─ Commit transaction
        │
        ▼
9. PostgreSQL: Persistence
   ├─ Flush to disk (encrypted if TDE enabled)
   └─ Return success to connection pool
        │
        ▼
10. Response Path (reverse flow)
    ├─ HikariCP returns connection to pool
    ├─ JPA updates entity @Version (optimistic locking)
    ├─ Service logs audit entry:
    │   └─ INSERT INTO audit_logs (user_id, action, timestamp)
    ├─ Controller returns 200 OK
    └─ Browser receives response over HTTPS
```
