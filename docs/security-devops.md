# Security and DevOps Best Practices

## Authentication and Sessions

- Use OAuth 2.1 / OpenID Connect where possible
- Store access tokens only in memory when practical
- Store refresh tokens in platform secure storage
- Rotate refresh tokens
- Support MFA/passkeys for team accounts
- Revoke sessions per device
- Use short-lived access tokens

## Authorization

- Enforce permissions on the backend for every request
- Treat client-side authorization as UX only
- Use workspace-scoped roles
- Prefer explicit permissions over broad role checks in code
- Record sensitive permission failures as audit events

Suggested roles:

- `owner`
- `admin`
- `editor`
- `viewer`

Suggested permission format:

```txt
workspace.read
workspace.manage_members
settings.update
resource.create
resource.update
resource.delete
audit.read
```

## Local Data Protection

- Use encrypted secure storage for tokens and secrets
- Encrypt sensitive local database fields or use encrypted database support
- Support biometric app lock where appropriate
- Allow users to wipe local data
- Separate data by workspace
- Never store plaintext passwords
- Minimize offline retention for highly sensitive data

## Network and Backend Security

- HTTPS only
- HSTS for web
- Certificate pinning for high-risk apps, with a safe rotation plan
- Rate limiting on auth and write endpoints
- Input validation with shared schemas
- Output encoding for web
- CSRF protection for cookie-based web auth
- CORS allowlist
- Secure headers: CSP, X-Frame-Options or frame-ancestors, Referrer-Policy
- Secrets stored in a managed secret manager
- Principle of least privilege for database and cloud credentials

## Sync Security

- Verify workspace access before accepting sync operations
- Validate entity ownership and permissions per operation
- Make sync operation IDs idempotent
- Sign or strongly validate client-generated IDs if abuse is a concern
- Keep server-side change logs tamper-resistant
- Audit destructive operations

## Privacy

- Data export endpoint
- Account deletion flow
- Workspace deletion flow
- Configurable telemetry consent
- Clear distinction between local-only and cloud-synced data
- Regional hosting option if needed

## CI/CD

Use separate pipelines for:

- Mobile app checks
- Web checks
- Backend checks
- Infrastructure checks

Recommended checks:

- Format
- Lint
- Unit tests
- Integration tests
- API contract tests
- Database migration tests
- Static security scanning
- Dependency vulnerability scanning
- Secret scanning

## Branch and Release Strategy

- Use pull requests for all changes
- Require passing checks before merge
- Protect `main`
- Use preview environments for backend/web
- Use TestFlight and Google Play internal testing for mobile
- Use semantic versioning or date-based release versions
- Maintain rollback plans for backend deployments

## Environments

Minimum:

- `local`
- `dev`
- `staging`
- `prod`

Each environment should have:

- Separate database
- Separate storage bucket
- Separate secrets
- Separate OAuth clients
- Separate push notification credentials

## Infrastructure

Recommended:

- Infrastructure as code with Terraform/OpenTofu
- Containerized backend services
- Managed PostgreSQL
- Managed Redis
- Managed object storage
- CDN/WAF for web and public APIs
- Central logging and metrics

## Observability

Track:

- API latency and error rates
- Auth failures
- Sync failures
- Conflict rates
- Outbox queue length
- Mobile crash reports
- Background sync health
- Database slow queries

Add:

- Structured logs
- Distributed tracing
- Request IDs
- Audit logs
- Alerting for production incidents

## Testing Strategy

Client:

- Unit tests for use cases
- Repository tests with local database
- Sync conflict tests
- UI tests for login, settings, and core workflows
- Offline/online transition tests

Backend:

- Unit tests for services
- API integration tests
- Authorization tests
- Migration tests
- Sync idempotency tests
- Load tests for sync endpoints

Security:

- Dependency scanning
- Secret scanning
- SAST
- Periodic penetration testing for production apps
- Threat modeling before major releases
