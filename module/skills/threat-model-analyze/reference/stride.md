# STRIDE Threat Modeling Framework

STRIDE is Microsoft's threat classification model. It provides six categories of threats, each representing a violation of a specific security property. Apply STRIDE per-element: for every component, data flow, and data store in the system, systematically consider each category.

## Threat Categories

### Spoofing (violates Authentication)
Pretending to be something or someone other than yourself.
- Can an attacker impersonate a user, service, or system component?
- Are authentication mechanisms bypassable, weak, or missing?
- Can tokens, cookies, or credentials be forged, replayed, or stolen?
- Are there unauthenticated endpoints that should require identity?

**Common attack patterns:** credential stuffing, session hijacking, token forgery, certificate spoofing, ARP/DNS spoofing, OAuth redirect manipulation.

**Mitigations:** multi-factor authentication, mutual TLS, signed tokens with expiration, certificate pinning, strong session management.

### Tampering (violates Integrity)
Modifying data or code without authorization.
- Can data in transit be modified (MITM)?
- Can data at rest be altered (database, config files, logs)?
- Can an attacker modify binaries, packages, or deployment artifacts?
- Are there missing integrity checks on inputs, uploads, or API payloads?

**Common attack patterns:** parameter tampering, SQL injection, XML/JSON injection, binary patching, config file manipulation, log injection, supply chain tampering.

**Mitigations:** input validation, parameterized queries, signed artifacts, checksums/HMAC, immutable infrastructure, write-ahead logging, content security policies.

### Repudiation (violates Non-repudiation)
Performing an action that cannot be traced back to the actor.
- Can a user deny having performed a transaction or action?
- Are audit logs comprehensive, tamper-proof, and centralized?
- Can an attacker delete or modify logs to cover their tracks?
- Are critical actions (admin operations, financial transactions, data access) logged with actor identity?

**Common attack patterns:** log deletion, log injection, timestamp manipulation, exploiting gaps in audit coverage, acting through shared/service accounts.

**Mitigations:** append-only logging, centralized log aggregation (SIEM), digital signatures on log entries, per-user identity for all actions, audit trail for admin operations.

### Information Disclosure (violates Confidentiality)
Exposing information to unauthorized parties.
- Can sensitive data leak through error messages, logs, or debug endpoints?
- Is data encrypted in transit and at rest?
- Are there overly permissive API responses (returning more data than needed)?
- Can an attacker enumerate users, resources, or internal structure?
- Are secrets (API keys, passwords, tokens) hardcoded or exposed?

**Common attack patterns:** directory traversal, IDOR, verbose error messages, side-channel attacks, memory dumps, exposed .env files, git history leaking secrets, timing attacks.

**Mitigations:** encryption (TLS, AES), least-privilege data access, response filtering, generic error messages, secret management (vault), data classification, DLP.

### Denial of Service (violates Availability)
Making a system or resource unavailable to legitimate users.
- Can an attacker exhaust resources (CPU, memory, disk, connections, API quotas)?
- Are there unbounded operations (queries without limits, file uploads without size caps)?
- Can a single user/tenant monopolize shared resources?
- Are there single points of failure?

**Common attack patterns:** volumetric DDoS, application-layer DoS (slowloris, regex DoS), resource exhaustion, lock contention, queue flooding, algorithmic complexity attacks.

**Mitigations:** rate limiting, request size limits, pagination, circuit breakers, auto-scaling, CDN/WAF, resource quotas, graceful degradation, bulkhead pattern.

### Elevation of Privilege (violates Authorization)
Gaining capabilities beyond what is authorized.
- Can a regular user access admin functionality?
- Are authorization checks enforced at every layer (not just the UI)?
- Can an attacker manipulate roles, permissions, or access tokens?
- Are there privilege escalation paths through misconfigurations?
- Can a compromised low-privilege component access high-privilege resources?

**Common attack patterns:** IDOR, JWT manipulation, role parameter tampering, path traversal to admin endpoints, container escape, SSRF to metadata services, dependency confusion.

**Mitigations:** role-based access control (RBAC), principle of least privilege, server-side authorization checks, input validation on role/permission fields, sandboxing, network segmentation.

## How to Apply STRIDE

1. **Enumerate elements**: List every component, data flow, data store, and external entity in the system.
2. **Per-element analysis**: For each element, walk through all six STRIDE categories. Not every category applies to every element type:
   - **External entities & processes**: all six categories
   - **Data flows**: Tampering, Information Disclosure, Denial of Service
   - **Data stores**: Tampering, Information Disclosure, Denial of Service, Repudiation
3. **Rate each threat**: Assess likelihood (how easy is this to exploit?) and impact (what's the blast radius?).
4. **Map to mitigations**: For each identified threat, specify existing controls and recommended mitigations.

## STRIDE-per-Interaction Variant

For complex systems, apply STRIDE to each interaction (data flow) rather than each element. This focuses on trust boundary crossings where threats are most likely to materialize.

## Risk Rating Guidance

| Likelihood | Description |
|---|---|
| High | Known attack pattern, easy to exploit, attacker-facing surface |
| Medium | Requires specific conditions or moderate skill |
| Low | Theoretical, requires insider access or chained exploits |

| Impact | Description |
|---|---|
| Critical | Full system compromise, mass data breach, safety risk |
| High | Significant data loss, service outage, compliance violation |
| Medium | Limited data exposure, degraded service, reputational damage |
| Low | Minimal impact, easily recoverable |
