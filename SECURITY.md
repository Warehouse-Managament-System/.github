# Security Policy

## Reporting a Vulnerability

If you discover a security vulnerability in this project, please report it responsibly.

**Do not open a public issue.**

Instead, please email the maintainers directly or use GitHub's private vulnerability reporting feature on the affected repository.

### What to include

- Description of the vulnerability
- Steps to reproduce
- Potential impact
- Suggested fix (if any)

### Response timeline

We aim to acknowledge reports within 48 hours and provide a fix or mitigation plan within 7 days for critical issues.

## Security Considerations

This platform handles:

- **Authentication credentials** — passwords are BCrypt-hashed, auth uses JWT with refresh tokens
- **Payment data** — processed through Stripe (PCI-compliant), no card data stored locally
- **Business data** — warehouse configurations, booking details, goods inventories

### Architecture security measures

- JWT-based authentication with role-based access control (5 roles)
- API Gateway as single entry point — internal service endpoints are not publicly exposed
- Service-to-service communication authenticated via internal tokens
- Input validation at API boundaries
- Stripe webhook signature verification for payment events
