# ADR: PII Protection in Logs & Errors
Status: Proposed

## Context
Risk: R-03 "PII leakage through observability" (L=4, I=4, Score=16)
DFD: API Controller Logs, Auth Service Error Handling
NFR: NFR-003, NFR-004
Assumptions:
- GDPR compliance is mandatory
- Current logging may contain PII (emails, IPs, tokens)
- Debugging capability must be maintained

## Decision
Implement comprehensive PII protection through structured logging with automatic masking and standardized error responses to prevent sensitive data exposure.
- **Param/Policy:** All PII fields (email, IP, verification tokens) masked in logs as `[REDACTED]` (scope: all application logs)
- **Param/Policy:** Error responses follow RFC7807 format without stack traces (scope: all API error responses)
- **Param/Policy:** Data retention: 30 days maximum for raw PII and incomplete registrations (scope: database and logs)
- **Param/Policy:** Structured JSON logging with correlation_id for traceability (scope: all application components)
- **Notes:** Applied at application layer across all services and components.

## Alternatives
- **Log Redaction Service** - Rejected because it allows PII to be written to logs initially, creating exposure risk.
- **Selective Logging** - Rejected because it reduces observability and makes debugging difficult for critical authentication flows.

## Consequences
+ **Positive:** GDPR compliance achieved; significantly reduced risk of PII exposure
+ **Positive:** Consistent error handling improves API contract per NFR-003
+ **Positive:** Maintains observability through correlation IDs and structured logs
- **Negative:** More difficult to debug production issues without raw PII
- **Negative:** Additional development and testing effort for masking logic

## DoD / Acceptance
**Given** DTO containing email="user@example.com" and IP="192.168.1.1"
**When** logged at any application level
**Then** log output shows email="[REDACTED]" and ip="[REDACTED]"

**Given** internal server error during registration
**When** client receives error response
**Then** response has Content-Type=application/problem+json with type/title/status/detail fields and no stack traces

**Given** unverified user registration record
**When** 30 days pass
**Then** record and associated verification tokens are automatically purged

**Checks:**
- test: Unit tests verify PII masking for all sensitive fields
- log: Sample log review confirms no unmasked PII in production logs
- scan: Automated security scan detects no PII in error responses
- policy: Data retention job logs show successful purging after 30 days

## Rollback / Fallback
- Rollback: Disable PII masking middleware (environment variable)
- Fallback: Maintain detailed debugging logs in non-production environments
- Monitoring: Watch for increased debugging time or support requests

## Trace
- DFD: API Controller, Auth Service nodes and their log outputs
- STRIDE: Rows for PII leakage in logs and errors
- Risk scoring: R-03 (Top-3, Score=16)
- NFR: NFR-003, NFR-004
- Issues: #API-001, #PRIV-001

## Ownership & Dates
Owner: Backend Team  
Reviewers: DevSecOps Team, Legal/Compliance  
Date created: 2024-06-15  
Last updated: 2024-06-15

## Open Questions
- What specific fields constitute PII beyond email, IP, and tokens?
- How to handle debugging scenarios that require PII access?
- Retention policy for audit logs vs application logs?