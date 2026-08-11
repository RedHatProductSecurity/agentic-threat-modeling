# THREAT_MODEL.md Output Template

This template defines the structure for all threat model reports. The report should read as a narrative — not a data dump. Each section builds on the previous one to tell the story of what this system is, who wants to attack it, what could go wrong, and what to do about it.

## Section Structure

### 1. Executive Summary
2-3 paragraphs, written for a non-technical audience. Cover:
- What this system does and why it matters to the business
- The top 3 threats that should concern leadership (in plain language)
- The single most important action to take first
- Overall risk posture: is this system in good shape, or is it a house of cards?

**Tone:** Direct, clear, no jargon. A VP who reads only this section should walk away knowing whether to be worried and what to fund.

### 2. System Context and Scope
- What was modeled (system name, version, deployment environment)
- What was explicitly excluded and why
- Key assumptions made during the analysis
- Date of analysis and methodology used

### 3. Architecture Overview
Describe the system architecture in enough detail to understand the threat landscape:
- Major components and their responsibilities
- How data flows between components (describe flows, not just list components)
- External integrations and third-party dependencies
- Deployment topology (cloud regions, network segments, container orchestration)

Use text-based descriptions that are diagram-ready — a reader should be able to sketch the architecture from this section.

### 4. Threat Actor Landscape
Who is likely targeting this system and why:
- List 2-4 most relevant threat actors (from threat-actors.md profiles)
- For each actor: why THIS system is attractive to them, what they'd be after, and how they'd likely approach it
- Explain the reasoning: "This system stores 2M customer payment records, making it a high-value target for financially motivated organized crime groups"

### 5. Assets and Crown Jewels
What's worth protecting, ranked by value:

| Asset | Description | Sensitivity | Impact if Compromised | Owner |
|---|---|---|---|---|
| Customer PII | Names, emails, addresses of 2M users | High | Regulatory fine + brand damage | Data Engineering |

For each high-value asset, briefly explain *why* it's valuable — to the business AND to an attacker.

### 5.5 Data Classification
This table connects what data exists, how sensitive it is, where it lives, where it moves, and what regulations govern it. Threats in section 7 reference this table to justify impact ratings.

| Data Type | Classification | Where Stored | Where It Flows | Regulatory Scope | Example Records |
|---|---|---|---|---|---|
| Customer PII | Confidential | PostgreSQL users table | API → DB, DB → backup S3 | GDPR, CCPA | name, email, address |

Classification levels:
- **Public** — no impact if disclosed
- **Internal** — low impact, internal use only
- **Confidential** — significant impact if disclosed (PII, financial, customer data)
- **Restricted** — severe impact if disclosed (credentials, encryption keys, trade secrets)

When a threat references a data type, its impact rating should be consistent with the classification here. A reader should be able to trace from a threat → the data it exposes → the classification and regulatory consequences.

### 6. Attack Surface
Entry points and trust boundaries:

| Entry Point | Type | Trust Boundary | Authentication | Notes |
|---|---|---|---|---|
| /api/v2/* | REST API | Internet → App tier | JWT Bearer | Public-facing |

Describe trust boundaries as transitions: "Data crosses from the untrusted internet into the application tier at the API gateway. The API gateway terminates TLS and validates JWTs before forwarding to backend services."

### 7. Threats
The core of the report. Each threat gets a title with inline severity, a narrative scenario, then mitigations and code references. No details table — the narrative carries the information.

Severity must be justified against the trust boundary/exposure data from Section 6 — don't rate an internal-only, authenticated component the same as an internet-facing one without explaining why.

**Example:**

#### T-001: Credential stuffing against user login endpoint

**Severity:** Critical | **Likelihood:** High

An attacker obtains a credential dump from an unrelated breach — 50M email/password pairs available for $50 on dark web markets. They run an automated credential stuffing tool against our `/api/auth/login` endpoint (`src/auth/routes.py:45`), testing thousands of combinations per minute. Our rate limiting is per-IP (`src/middleware/rate_limit.py:12`) and they're using a rotating proxy service. Within hours, they've identified 12,000 accounts where users reused passwords. They drain stored payment methods and sell the remaining valid credentials.

**Actors:** Organized crime (financial gain from stolen payment methods), Opportunistic (commodity automation tools)

**Existing Controls:** Per-IP rate limiting — insufficient (`src/middleware/rate_limit.py:12`)

**Recommended Mitigations:**
- Implement MFA for all user accounts
- Add credential breach monitoring (check passwords against known dumps)
- Add device fingerprinting to detect automated tools

**Affected Files:** `src/auth/routes.py:45`, `src/middleware/rate_limit.py:12`

Every existing control needs a citation like the one above, or an explicit `Unverified` marker if it couldn't be confirmed in the codebase — e.g. `GENERAL_TRACKING_ENABLED=False — Unverified, no evidence found in codebase`. Never state a mitigation as fact without one or the other.

### 8. Attack Scenarios (Top 3-5)
For the highest-risk threats, expand into full kill-chain narratives:

**Scenario: "The Credential Stuffing Campaign"**
1. **Reconnaissance:** Attacker identifies our login endpoint from public documentation. They note we use email/password authentication without visible MFA requirement.
2. **Resource Development:** They purchase a recent credential dump (500M records, $200) and rent a residential proxy service ($50/month for rotating IPs).
3. **Initial Access:** Automated tool tests credentials at 100/second across 1000 rotating IPs, staying under our per-IP rate limit of 10/second.
4. **Account Takeover:** 0.5% hit rate yields 12,000 compromised accounts within 6 hours.
5. **Impact:** Attacker accesses stored payment methods, makes unauthorized purchases, and sells remaining valid credentials on dark web forums.

**Why this matters:** This attack requires minimal skill and investment (~$250) but yields potentially millions in fraud losses plus regulatory notification requirements under state breach laws.

### 9. Mitigation Roadmap
Grouped by urgency and effort:

**Immediate (this sprint):**
- [ ] Mitigation description — addresses T-001, T-003 — effort: small

**Short-term (this quarter):**
- [ ] Mitigation description — addresses T-002, T-005 — effort: medium

**Long-term (next quarter+):**
- [ ] Mitigation description — addresses T-004 — effort: large

For each mitigation, explain what risk it reduces and roughly how much.

### 9.5 Hardening Recommendations
Good practice, not specific weaknesses in this system. These are findings that didn't tie to a concrete exploitable weakness given this system's actual trust boundaries, data, and actors — generic defense-in-depth, redundant with a platform guarantee already in place, or constrained by a third party's own model. No severity or likelihood rating — assigning one would misrepresent these as threats.

Each gets an ID in its own sequence (H-001, H-002, ...) so it can be tracked and preserved across updates the same way threats are.

- **H-001: Validate responses from the upstream tracker service** — Defense-in-depth against a trusted, internally-operated upstream. Suggested action: add response schema validation at the integration boundary.

Omit this section if none were identified.

### 10. Deprioritized Threats
Threats that were identified but deemed acceptable risk. Document them so future reviewers know they weren't overlooked:

| Threat | Why Deprioritized |
|---|---|
| T-008: Physical access to data center | Managed by cloud provider (AWS); covered by their SOC2 Type II |

### 11. Open Questions and Assumptions
Unknowns that could change the risk picture:
- "We assumed the API gateway handles TLS termination — if this changes, T-003 severity increases to Critical"
- "We could not determine whether the database backup encryption key is rotated — this needs verification"

### 11.5 External Context Discrepancies
When external documentation was ingested, flag any gaps between documented architecture/controls and actual implementation. Each discrepancy is itself a finding worth tracking — it means either the docs are stale or the implementation drifted.

| Source | Documented Claim | Actual Implementation | Impact |
|---|---|---|---|
| `docs/architecture.md` | "All traffic encrypted via mTLS" | No mTLS configured; TLS terminates at load balancer | Data-in-transit gap for east-west traffic |

Omit this section if no external context was provided.

### 12. Provenance
| Field | Value |
|---|---|
| Date | YYYY-MM-DD |
| Framework(s) | STRIDE, PASTA, etc. |
| Scope | Repository/service/system name |
| Actor Personas Applied | organized-crime, insider, supply-chain |
| Mode | quick / guided / full |
| External Context | List of ingested sources with brief description, or "None" |
| Participants | Names or roles involved |

**Feedback:** Was this threat model useful? Run `/threat-model-feedback` or [file feedback directly](https://github.com/RedHatProductSecurity/agentic-threat-modeling/issues/new?template=feedback.yml).

## Writing Guidelines

- **Tell stories, not just list findings.** Every threat should answer: "What happens, who does it, why, and what does it cost us?"
- **Be specific.** "An attacker could access sensitive data" is useless. "An attacker exploits the IDOR in /api/users/{id}/billing to access other users' payment card details" is actionable.
- **Quantify where possible.** "2M records at risk" beats "data could be exposed." "$50K estimated remediation cost" beats "significant financial impact."
- **Use plain language.** The executive summary should be readable by anyone. Technical details belong in the threat details, not the summary.
- **Acknowledge uncertainty.** If likelihood is hard to assess, say so. An honest "we don't know" is better than false precision.
- **Make it actionable.** Every threat should map to a mitigation. Every mitigation should be concrete enough to become a task.
