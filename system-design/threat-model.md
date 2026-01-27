# Threat Model: Subscription Manager

## Assets
- User credentials
- Subscription data (may include payment info, personal links)
- Email addresses
- Session tokens

## Trust Boundaries
1. User browser ↔ Web server
2. Web server ↔ Database
3. Web server ↔ Email service


## Threats & Mitigations

### Threat: Email Link Tampering
- **Threat**: Attacker modifies "View & Manage" URL to access another user's subscription
- **Impact**: High - unauthorized access to personal data
- **Mitigation**: Signed tokens with user_id + subscription_id + expiration
- **Validation**: Token verified server-side before showing data

[Continue for 10-15 key threats]