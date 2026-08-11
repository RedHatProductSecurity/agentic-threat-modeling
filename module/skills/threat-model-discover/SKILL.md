---
name: threat-model-discover
description: Discovers system context for threat modeling through code analysis and/or user interview. Produces a structured system profile covering components, data flows, trust boundaries, assets, and entry points.
allowed-tools:
  - Read
  - Bash
  - WebFetch
---

# System Context Discovery

You are performing the discovery phase of threat modeling. Your job is to build a complete picture of the system being modeled — its components, how data flows, where trust boundaries exist, what assets matter, and where attackers can enter.

You operate in two modes. The dispatcher skill will tell you which to use.

## Step 0: External Context Ingestion

If `context:` arguments were provided, split on commas and ingest each source. In guided or full mode, ask if there's anything else:

> "Do you have any other external documentation I should ingest? Architecture docs, prior threat models, API specs, deployment diagrams, SBOMs, or SAR reports?"

If no `context:` arguments were provided and the mode is guided or full, ask the same question without "other." In quick mode, skip prompting entirely — just ingest any `context:` arguments and move on.

**Supported external sources:**

| Source Type | What's Extracted | How to Provide |
|---|---|---|
| Google Docs / Confluence pages | Architecture descriptions, component relationships, data flows | Public URL, or paste/export content. Authenticated pages (behind SSO, Google login) require exporting first. |
| Mermaid / PlantUML diagrams | Component topology, data flows, trust boundaries | Paste content or provide file path |
| Prior threat models (THREAT_MODEL.md) | Previously identified threats, accepted risks, implemented mitigations | File path or URL |
| SAR reports (Security Assessment Reports) | Previous findings, compliance status, remediation state | File path or URL |
| OpenAPI / Swagger specs (YAML/JSON) | Endpoints, auth schemes, data models, request/response shapes | File path in repo or external |
| SBOM (CycloneDX or SPDX) | Dependency inventory, known vulnerabilities, license risks | File path |
| Product definition JSON | Service catalog, ownership, dependencies | File path |
| Deployment topology YAML | Network segments, trust zones, cloud services, regions | File path |
| Terraform / CloudFormation templates | Infrastructure components, network rules, IAM policies, storage configs | File path in repo or external |
| Kubernetes manifests / Helm values | Pod security, network policies, RBAC, secrets management | File path in repo or external |

**Ingestion workflow:**
1. For URLs: fetch the content, then extract security-relevant information
2. For file paths: read the file and parse according to format
3. For directory paths: list the directory and ingest all supported files (`.yaml`, `.yml`, `.json`, `.md`, `.tf`, `.hcl`, `.xml`) — skip binaries, images, and other non-parseable formats
4. For pasted content: parse directly

**If a source cannot be ingested:**
- URL returns an error or requires authentication → tell the user and suggest they paste the content or provide a local file path instead
- File path does not exist → tell the user with the exact path attempted
- Content is too large or in an unparseable binary format → tell the user what was attempted and skip gracefully
- Never silently skip a source the user explicitly provided

**After ingestion, confirm what was extracted:**
For each ingested source, briefly summarize what security-relevant information was found — e.g., "From `docs/ess_controls.md`: 15 security controls covering authentication, encryption, logging, and patching." This lets the user verify the right content was captured and course-correct early.

**Merging with auto-discovery:**
When both external docs and code are available, merge them with this priority:
- **Architectural intent** (how it *should* work): prefer external docs — they capture design decisions and context that code alone can't convey
- **Implementation reality** (how it *actually* works): prefer code analysis — it shows what's deployed, not what was planned
- **Conflicts**: note discrepancies as findings — a gap between documented architecture and actual implementation is itself a security concern

If the user has no external sources, proceed directly to auto-discovery. Don't block on this step.

## Mode: Auto-Discovery (from code)

Analyze the repository to build a system context profile. Work through these steps:

### 1. Project Structure Scan
- Read the project README, architecture docs, and any design documents
- Examine the directory structure to identify service boundaries
- Look for deployment manifests (Dockerfile, docker-compose.yml, k8s manifests, Helm charts, Terraform)
- Check CI/CD configuration (.github/workflows, .gitlab-ci.yml, Jenkinsfile)
- If you find files matching supported external context types (OpenAPI specs, Kubernetes manifests, Terraform configs, SBOMs) that were not already provided via `context:`, note them as available sources in the system context output so the user knows they can be ingested

### 2. API and Interface Discovery
- Find API definitions: OpenAPI/Swagger specs, GraphQL schemas, protobuf definitions
- Scan route definitions and controller files for endpoints
- Identify WebSocket, gRPC, or message queue consumers
- Look for admin interfaces, health check endpoints, debug endpoints

### 3. Data Store and State Discovery
- Identify databases from connection strings, ORM configs, migration files
- Find cache layers (Redis, Memcached configs)
- Identify message queues (Kafka, RabbitMQ, SQS configs)
- Check for file storage (S3, blob storage, local filesystem usage)

### 4. Data Classification
Cross-reference discovered data stores, data flows, and code patterns to build a data classification table. For each data type found in the system, determine:

- **Classification level:**
  - **Public** — no impact if disclosed (marketing content, public docs)
  - **Internal** — low impact, for internal use (internal metrics, non-sensitive configs)
  - **Confidential** — significant impact if disclosed (PII, customer data, financial records)
  - **Restricted** — severe impact if disclosed (credentials, encryption keys, trade secrets)

- **Where it's stored** — which data stores hold this data type
- **Where it flows** — which data flows carry this data type
- **Regulatory scope** — which regulations govern this data (GDPR, CCPA, HIPAA, PCI-DSS, SOX, etc.)
- **Example records** — concrete field names or record types

To classify data, examine:
- ORM models and migration files for column names suggesting sensitive data (email, password, ssn, card_number, dob, address, etc.)
- Data validation and masking patterns in the code
- Encryption-at-rest configurations (which tables/columns are encrypted?)
- Access control patterns (who can query what tables, column-level permissions)
- Compliance-related comments or annotations in code
- Environment variable names referencing secrets or keys

| Data Type | Classification | Where Stored | Where It Flows | Regulatory Scope | Example Records |
|---|---|---|---|---|---|
| Customer PII | Confidential | PostgreSQL users table | API → DB, DB → backup S3 | GDPR, CCPA | name, email, address |
| Audit logs | Internal | Elasticsearch | App → ES, ES → SIEM | SOX | user actions, timestamps |
| API keys | Restricted | HashiCorp Vault | Vault → App (runtime) | Internal policy | third-party service keys |

This table feeds directly into the analyze phase to calibrate threat severity — a vulnerability exposing Restricted data is Critical; the same vulnerability exposing Public data is Low.

### 5. Authentication and Authorization Discovery
- Find auth middleware, JWT handling, OAuth configs, session management
- Identify user roles and permission models
- Check for API key usage, service-to-service auth (mTLS, service accounts)
- Look for secrets management (vault configs, env var patterns, secret references)

### 6. External Dependency Discovery
- Examine package manifests (package.json, requirements.txt, go.mod, pom.xml, Gemfile)
- Identify third-party API integrations from code (HTTP clients, SDKs)
- Find external service configurations (payment providers, email services, analytics)

### 7. Trust Boundary Mapping
Based on what you found, identify trust boundaries:
- Network boundaries (internet → DMZ → app tier → data tier)
- Process boundaries (container isolation, service mesh)
- Authentication boundaries (anonymous → authenticated → admin)
- Organizational boundaries (our code → third-party services)

While mapping boundaries, note any place where code comments, READMEs, or architecture docs explicitly state that a security-relevant choice is deliberate (e.g., "auth is intentionally skipped here — internal-only endpoint"). Record these in Assumptions and Design Decisions with the source (file:line or doc) — don't rely on inferring intent from the code alone.

### 8. Asset Identification
Identify what's worth protecting:
- User data (PII, credentials, payment info)
- Business data (financial records, trade secrets, config)
- System assets (admin access, encryption keys, certificates)
- Availability-critical components (what breaks if this goes down?)

## Mode: PR / Diff Analysis

Threat-model a specific change rather than the full system. This mode produces a focused, lightweight assessment of what a PR or diff introduces.

### 1. Get the Diff
- If a PR number is provided: use `glab mr diff <number>` or `gh pr diff <number>` to get the changes
- If no PR number: use `git diff main...HEAD` (or the appropriate base branch) to get the current branch's changes
- If a patch file is provided: read it directly

### 2. Identify Security-Relevant Changes
Categorize what changed:
- **New/modified endpoints or routes** — new attack surface
- **Changes to auth/authz logic** — potential bypass or weakening
- **New data stores or schema changes** — new data at risk
- **New external integrations or dependencies** — new trust relationships
- **Changes to trust boundaries** — new network configs, modified Dockerfiles, updated k8s manifests
- **New secrets or credential handling** — exposure risk
- **Removed security controls** — regression risk

Skip changes that have no security relevance (formatting, comments, test data, documentation-only).

### 3. Build Scoped System Context
Create a focused context covering:
- The changed components and what they do
- Their immediate neighbors (components that call or are called by the changed code)
- Trust boundaries the changes touch
- Data flows affected by the changes

### 4. Cross-reference Existing Threat Model
If a THREAT_MODEL.md exists in the repo:
- Check which existing threats are affected by this change
- Note if this change resolves, worsens, or invalidates any existing threats
- Check if existing mitigations are impacted

### Output Format

```
## PR System Context: [PR title or branch name]

### Changes Summary
| File | Change Type | Security Relevance |
|---|---|---|
| src/auth/oauth.py | Modified | Auth flow change — high relevance |

### Changed Components
| Component | What Changed | Trust Boundary Impact |
|---|---|---|

### Affected Data Flows
| Flow | What Changed | Data Sensitivity |
|---|---|---|

### Existing Threats Affected
| Threat ID | Impact of This Change |
|---|---|

### New Attack Surface
- New endpoints, integrations, or data stores introduced
```

## Mode: Bootstrap-then-Interview

Run auto-discovery first, then present findings to the user and ask targeted questions about gaps — things you couldn't determine from code alone:
- What data is considered most sensitive?
- Who are the users and what are their roles?
- What compliance requirements apply?
- What's the deployment environment look like in production?
- Are there any known threats or past incidents?
- Are there any security-relevant design choices you know are intentional rather than oversights (e.g., relaxed auth defaults on an internal-only service, no encryption on a boundary considered trusted)? Record these with their source — the analyze phase treats a documented, sourced design decision differently from an apparent misconfiguration.

## Output Format

Produce a structured system context document with these sections:

```
## System Context: [System Name]

### Components
| Component | Type | Technology | Exposure | Trust Zone |
|---|---|---|---|---|
| API Gateway | gateway | nginx | internet-facing | dmz |

### Data Flows
| From | To | Data | Protocol | Auth | Crosses Trust Boundary |
|---|---|---|---|---|---|

### Trust Boundaries
| Boundary | From Zone | To Zone | Controls |
|---|---|---|---|

### Data Classification
| Data Type | Classification | Where Stored | Where It Flows | Regulatory Scope | Example Records |
|---|---|---|---|---|---|

### Assets
| Asset | Sensitivity | C/I/A Priority | Owner |
|---|---|---|---|

### Entry Points
| Entry Point | Type | Trust Boundary | Authentication |
|---|---|---|---|

### Assumptions and Design Decisions
- Security-relevant choices confirmed to be intentional, with source — e.g., "Empty DRF permission defaults are deliberate: the API is internal-only behind Kerberos ingress (confirmed via interview with service owner)."

### Open Questions
- Things you couldn't determine that need human input
```

This output feeds directly into the analyze phase.
