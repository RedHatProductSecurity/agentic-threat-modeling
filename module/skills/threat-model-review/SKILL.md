---
name: threat-model-review
description: Reviews an existing THREAT_MODEL.md against the current codebase. Identifies stale threats, new attack surface, drift from assumptions, and unimplemented mitigations.
allowed-tools:
  - Read
  - Bash
---

# Threat Model Review

You review existing threat models to determine if they're still accurate and complete. Threat models decay — code changes, new features ship, dependencies update, infrastructure evolves. Your job is to identify what's drifted.

## Workflow

### 1. Find and Parse the Existing Threat Model
- Locate THREAT_MODEL.md at the path specified (default: repository root)
- Parse all sections: system context, threats, mitigations, assumptions, provenance
- Note the date it was last updated and the scope it covers

### 2. Analyze Code Changes
Compare the current codebase against what the threat model describes:

**New attack surface:**
- New API endpoints not covered in the entry points section
- New data stores or external integrations
- New authentication/authorization mechanisms
- New deployment configurations (containers, cloud services)
- New dependencies (check package manifests for additions)

**Removed components:**
- Endpoints, services, or integrations that were removed
- Threats tied to removed components are now stale

**Modified components:**
- Changes to auth flows, data handling, or API contracts
- Infrastructure changes (new network segments, changed deployment model)
- Dependency updates that may resolve or introduce vulnerabilities

### 3. Check Threat Currency

For each threat in the existing model:

**Still valid?**
- Does the affected component still exist?
- Is the attack vector still exposed?
- Has the vulnerability been patched or mitigated?

**Risk changed?**
- Have new controls been added that reduce likelihood?
- Has the blast radius changed (more data, more users, higher sensitivity)?
- Has the threat landscape shifted (new CVEs, new attack techniques)?

**Stale?**
- Component was removed or completely rewritten
- Mitigation was fully implemented
- The assumption it was based on is no longer true

### 4. Check Mitigation Status

For each recommended mitigation in the existing model:
- Has it been implemented? (Look for evidence in code: rate limiting added, MFA implemented, encryption enabled)
- Is it partially implemented?
- Is it still relevant? (The threat it addresses may have changed)

### 5. Check Assumptions

For each assumption listed in the existing model:
- Is it still true?
- Has the context changed? (e.g., "we assume single-tenant" but multi-tenancy was added)

### 6. Actor Relevance Check

If the existing model includes threat actors:
- Has the system changed in ways that attract new actor types?
- Has the data sensitivity changed?
- Has the deployment exposure changed (internal → internet-facing)?

## Output

Produce a review report with the sections below, then **offer to apply the changes directly**.

After presenting the review, ask the user:

> "Would you like me to apply these changes to the threat model now? I can update the THREAT_MODEL.md to incorporate the findings — removing stale threats, adjusting risk levels, and adding coverage for new attack surface. You'll be able to review the changes before they're finalized."

If the user agrees, proceed directly into the update workflow:
1. Use the review findings as structured feedback
2. Parse the existing THREAT_MODEL.md
3. Apply changes following the update mode rules (preserve IDs — T-NNN and H-NNN alike, append provenance, move removed threats or implemented/no-longer-relevant hardening recommendations to deprioritized)
4. Write the updated THREAT_MODEL.md
5. Summarize what changed
6. Ask if they're willing to give feedback: "Would you be willing to spend 2 minutes on feedback for this review? It directly shapes what gets improved next." If they say yes, invoke the `threat-model-feedback` flow immediately. If they decline or don't respond, don't push — move on.

This eliminates the copy-paste step between review and update. The user stays in one flow.

If the user declines or wants to review first, that's fine — the review report stands on its own.

### Threat Model Health Summary
One paragraph: is this threat model still mostly accurate, significantly outdated, or dangerously stale? When was it last updated? What's the overall drift?

### New Attack Surface
Components, endpoints, data flows, or integrations added since the threat model was written that are not covered. For each:
- What was added
- Why it matters from a threat perspective
- Recommended: add threats for these in the next update

### Stale Threats
Threats in the existing model that no longer apply. For each:
- Which threat (by ID and title)
- Why it's stale (component removed, mitigation implemented, etc.)
- Recommended: remove or mark as resolved

### Changed Risk
Threats where the risk level has changed. For each:
- Which threat
- What changed (new controls, expanded scope, new attack techniques)
- Recommended new risk level

### Mitigation Status
| Mitigation | Status | Evidence |
|---|---|---|
| M-001: Add rate limiting | Implemented | Rate limiter middleware in src/middleware/rateLimit.ts |
| M-002: Enable MFA | Not implemented | No MFA code found |

### Hardening Recommendation Status
Check hardening recommendations (H-NNN) the same way as mitigations — has each been implemented, and is it still relevant given how the system has changed?

| Recommendation | Status | Evidence |
|---|---|---|
| H-001: Validate upstream responses | Not implemented | No schema validation found at the integration boundary |

Also check whether a recommendation should be reclassified: has a hardening item's context changed such that it's now a real, exploitable threat (e.g., a formerly internal-only upstream is now reachable from a less-trusted network)? Flag that as a recommended action, not a silent status change.

### Invalidated Assumptions
Assumptions from the original model that are no longer true, with impact on the threat landscape.

### Recommended Actions
Prioritized list of updates to make to the threat model:
1. Add coverage for [new component]
2. Remove stale threat T-005
3. Update risk level for T-003 from Medium to High
4. Run a focused analysis on [new integration]
5. Mark hardening recommendation H-001 as implemented, or reclassify it as a threat if its trust context changed
