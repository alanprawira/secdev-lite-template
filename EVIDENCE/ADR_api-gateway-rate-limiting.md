# ADR: API Gateway Rate Limiting & Request Validation
Status: Proposed

## Context
Risk: R-01 "Public endpoint DoS/abuse" (L=4, I=4, Score=16)
DFD: Edge Internet → API (/register, /verify-email, /login)
NFR: NFR-001, NFR-002
Assumptions:
- API Gateway is already in place (Kong/NGINX/Cloud)
- Authentication endpoints are public-facing
- Need to maintain user onboarding experience

## Decision
Implement comprehensive rate limiting and request validation at the API Gateway layer to protect public authentication endpoints from abuse and DoS attacks.
- **Param/Policy:** Rate limit: 3 requests/minute per IP for `/api/auth/verify-email` (scope: public endpoint)
- **Param/Policy:** Rate limit: 10 requests/minute per IP for `/api/auth/login` (scope: public endpoint)  
- **Param/Policy:** Request size limit: 64 KiB for all POST endpoints (scope: all public endpoints)
- **Param/Policy:** Response: 429 + Retry-After for rate limits; 413 for oversized requests (scope: error responses)
- **Notes:** Applied at API Gateway layer before requests reach application servers.

## Alternatives
- **Application-level Rate Limiting** - Rejected because it allows abusive traffic to reach application servers, consuming resources.
- **Cloud WAF with DDoS Protection** - Rejected due to high cost, complexity, and external dependency for our current scale.

## Consequences
+ **Positive:** Immediate protection at the edge; reduced load on application servers; compliance with NFR-001 and NFR-002
+ **Positive:** Clear abuse detection through gateway logs and metrics
- **Negative:** Potential false positives blocking legitimate users; requires careful tuning of limits
- **Negative:** Additional configuration and maintenance of gateway rules

## DoD / Acceptance
**Given** IP address makes 5 requests to `/api/auth/verify-email` in 60 seconds
**When** requests are processed by API Gateway
**Then** requests 4-5 receive 429 status with Retry-After header

**Given** client sends 128 KiB payload to `/api/auth/register`
**When** request reaches API Gateway  
**Then** request is rejected with 413 status and RFC7807 body

**Checks:**
- test: E2E tests verify rate limiting behavior for all protected endpoints
- log: Gateway logs show 429/413 events with client IP and endpoint
- metric: Rate limit hit rate < 1% of total traffic during normal operation
- scan: Security scan confirms no oversized requests bypass validation

## Rollback / Fallback
- Rollback: Disable rate limiting rules in gateway configuration
- Fallback: Implement application-level fallback limits if gateway fails
- Monitoring: Watch for sudden traffic spikes or increased error rates during rollback

## Trace
- DFD: Edge Internet → API (all auth endpoints)
- STRIDE: Rows for /register and /verify-email DoS threats
- Risk scoring: R-01 (Top-1, Score=16)
- NFR: NFR-001, NFR-002
- Issues: #SEC-001, #SEC-002

## Ownership & Dates
Owner: DevSecOps Team  
Reviewers: Backend Team, Product Owner  
Date created: 2024-06-15  
Last updated: 2024-06-15

## Open Questions
- Should rate limits be different for mobile vs web clients?
- How to handle legitimate bulk operations from single IPs?
