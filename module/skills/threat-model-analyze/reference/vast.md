# VAST (Visual, Agile, and Simple Threat) Modeling Framework

VAST is a threat modeling methodology designed for enterprise scale and Agile/DevOps integration. Unlike STRIDE (which is thorough but manual) or PASTA (which is deep but heavyweight), VAST is built to be fast, repeatable, and embeddable in CI/CD workflows. It produces two complementary model types.

## Two Model Types

### Application Threat Model (ATM)
Built from the developer's perspective using process flow diagrams.

**Focus:** How data moves through application logic — request handling, business logic, data processing, external calls.

**How to build an ATM:**
1. Map the application's process flow as the user/data experiences it
2. Identify each processing step: input handling, validation, business logic, data access, output formatting
3. At each step, ask:
   - What input does this step accept? Can it be malformed or malicious?
   - What privileges does this step run with? Are they minimal?
   - What data does this step access? Is it the minimum needed?
   - What output does this step produce? Can it leak information?
   - What happens if this step fails? Is the failure safe?
4. Mark trust boundary crossings (client→server, service→database, internal→external)
5. For each boundary crossing, identify the controls that should exist

**ATM threat categories:**
- Input validation failures
- Authentication/authorization gaps
- Data exposure at rest or in transit
- Error handling information leaks
- Business logic abuse
- Race conditions and state management
- Dependency/supply chain risks

### Operational Threat Model (OTM)
Built from the infrastructure/ops perspective using data flow diagrams.

**Focus:** How the system is deployed, networked, and operated — infrastructure, network topology, deployment pipelines, operational processes.

**How to build an OTM:**
1. Map the infrastructure: servers, containers, cloud services, networks, load balancers, CDN
2. Draw network segments and firewall rules
3. Identify operational flows: deployment pipelines, monitoring, backup, incident response
4. At each infrastructure component, ask:
   - Is this component hardened (minimal attack surface, patched, secure defaults)?
   - Is network access to this component restricted to what's necessary?
   - Are credentials for this component rotated and stored securely?
   - What happens if this component is compromised? What's the blast radius?
   - Is this component monitored for anomalies?
5. For each operational flow, identify where human error or malicious action could cause harm

**OTM threat categories:**
- Network segmentation failures
- Infrastructure misconfiguration
- Credential exposure and management gaps
- Deployment pipeline compromise (supply chain)
- Monitoring and alerting blind spots
- Backup and recovery failures
- Insider access abuse
- Cloud service misconfiguration (IAM, storage, networking)

## Agile Integration

VAST is designed to fit into sprint cycles:

**Per-story threat assessment:**
- When a new user story touches a trust boundary, security-relevant data, or a new integration — do a lightweight ATM for that story
- Takes 15-30 minutes, produces 3-5 actionable findings
- Findings become acceptance criteria or security tasks in the sprint backlog

**Per-release threat review:**
- Before each release, review the cumulative ATM and OTM for new attack surface
- Focus on what changed since the last release
- Update the threat model incrementally rather than rewriting from scratch

**CI/CD integration points:**
- Pre-merge: automated checks for known-vulnerable patterns
- Pre-deploy: verify security controls are in place for new components
- Post-deploy: validate that operational controls (monitoring, network rules) match the OTM

## Applying VAST

1. **Choose the model type**: ATM for application-level analysis, OTM for infrastructure-level, or both for comprehensive coverage.
2. **Draw the diagram**: Process flow (ATM) or data flow (OTM) — keep it simple and accurate.
3. **Walk each element**: Apply the relevant threat categories at each step/component.
4. **Score by attacker ROI**: Prioritize threats where the attacker's effort-to-reward ratio is most favorable.
5. **Output as actionable items**: Each finding should be specific enough to become a task in a sprint backlog.

## When to Use VAST

- Large organizations with many teams and applications
- Agile/DevOps environments where heavyweight processes don't fit
- When you need both application and infrastructure threat perspectives
- When threat modeling needs to scale across dozens of services
- When findings need to feed directly into sprint backlogs

## VAST vs. Other Frameworks

| Aspect | VAST | STRIDE | PASTA |
|---|---|---|---|
| Speed | Fast (sprint-compatible) | Moderate | Slow (7 stages) |
| Scale | Enterprise (many apps) | Per-application | Per-application |
| Perspective | App + Infrastructure | App elements | Business + Technical |
| Output style | Actionable backlog items | Threat classification | Risk quantification |
| Best audience | Dev + Ops teams | Security engineers | Business + Security |
