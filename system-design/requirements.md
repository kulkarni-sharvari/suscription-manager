## Core Features (MVP - Must Have)
### User Management

1. User registration with email verification
2. Login with secure session management
3. Multi-Factor Authentication (MFA) setup
4. Password reset flow

### Subscription Management

1. Add new subscription (all 11 fields)
2. View all subscriptions (grouped by status: Active/Inactive)
3. Edit subscription details
4. Update subscription status (active/cancelled/inactive/expired_pending)
5. Reactivate cancelled/inactive subscriptions
6. Custom reminder period per subscription (1-365 days)

### Email Notification System

1. Daily scheduled job (8:00 AM)
2. Send reminder emails based on reminder_days_before
3. Secure "View & Manage" links with signed, time-limited tokens
4. Auto-mark subscriptions as 'expired_pending' after billing date passes
5. Stop notifications for non-active statuses

### Security & Compliance Features (AppSec Focus)
Security Controls

1. E-mail verification before account activation
2. Tokenized email links (signed, time-limited, single-use preferred)
3. Rate limiting on:
    - Login attempts (prevent brute force)
    - Email sending (prevent abuse)
    - API endpoints (prevent DoS)


4. Audit logging for:

    - Authentication events (login, logout, failed attempts)
    - Subscription changes (create, update, delete, status changes)
    - Security events (MFA setup, password changes)


5. Input validation and sanitization on all user inputs
6. CSRF protection on all state-changing operations

### Data Management

1. Export subscriptions (CSV/JSON) with secure delivery
2. Account deletion with data purge option

## Enhanced Features (Phase 2 - Nice to Have)
1. Collaboration (Authorization Complexity)
    - Share subscriptions with family members
    - Role-based access control (Owner/Viewer)
    - View-only vs edit permissions

2. Analytics & Insights
    - Dashboard showing:
    - Total monthly/yearly subscription costs
    - Upcoming renewals (next 30 days)
    - Spending trends over time
    - Cost breakdown by category/service type

4. Advanced Notifications
    - Notification preferences (email frequency, digest mode)
    - Webhook support for integration with other tools
    - SMS notifications (optional channel)