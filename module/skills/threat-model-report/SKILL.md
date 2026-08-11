---
name: threat-model-report
description: Generates the final threat model deliverable. Produces a narrative THREAT_MODEL.md report.
allowed-tools:
  - Read
  - Bash
---

# Threat Model Report Generator

You are producing the final threat model deliverable. You receive structured findings from the analysis phase and transform them into a report that tells the story of this system's threat landscape.

## Core Principle: Tell the Story

This is not a data dump. Every section should be readable by someone encountering this system for the first time. The report should answer these questions in order:

1. What is this system and why does it matter?
2. Who wants to attack it and why?
3. What are the most dangerous things that could happen?
4. What should we do about it, and in what order?

## Report Generation

### Load the Template
Read `./templates/threat-model-template.md` for the full section structure and writing guidelines. Follow it closely — it defines the output contract.

### Write Each Section

**Executive Summary:**
Write this LAST (after all other sections are complete) but place it FIRST in the document. Distill the entire analysis into 2-3 paragraphs. Name the top 3 threats in plain language. State the single most important action. Give an honest overall risk assessment.

Do not use security jargon in the executive summary. "An attacker could steal customer payment details through a flaw in the search feature" — not "STRIDE-identified information disclosure vulnerability via SQL injection in the user search endpoint."

**Architecture Overview:**
Describe the system so someone could sketch it on a whiteboard. Use clear component names and describe how data actually flows — "When a user logs in, the React frontend sends credentials to the API gateway, which forwards to the auth service, which checks against the PostgreSQL user database and returns a JWT."

**Threat Actor Landscape:**
For each relevant actor, write a paragraph explaining why they'd target THIS system specifically. Connect it to what the system stores, processes, or exposes. "This system processes payment card data for 50,000 transactions per month, making it a high-value target for organized crime groups who monetize stolen card data through dark web marketplaces."

**Threats Section:**
For each threat, use this format — no details table, the narrative carries the information:
- Title with inline severity and likelihood: `**Severity:** Critical | **Likelihood:** High`
- A narrative scenario (3-5 sentences) written in present tense, describing the attack as it unfolds. Weave in `file:line` code references naturally — e.g., "The login endpoint (`src/auth/routes.py:45`) accepts credentials without rate limiting..."
- Relevant actors (who would exploit this and why)
- Existing controls (what's already in place) — write each as the citation the analysis phase verified, e.g. `Per-IP rate limiting (src/middleware/rate_limit.py:12)`. If analysis marked a claimed control `Unverified`, say so explicitly rather than dropping the caveat — e.g. `GENERAL_TRACKING_ENABLED=False — Unverified, no evidence found in codebase`. Never restate an unverified control as settled fact.
- Recommended mitigations (bullet list, specific and actionable)
- Affected files (list of `file:line` references)

Order threats by risk level (critical first), then by likelihood within the same risk level.

Severity/likelihood must be justified against the trust-boundary and exposure data from the system context, not asserted. Don't carry forward a Critical/High rating from the analysis phase without checking it against what boundary actually sits in front of the affected component — an internal-only, authenticated service is not equivalent to an internet-facing one at the same nominal vulnerability.

**Attack Scenarios:**
Expand the top 3-5 threats into full kill-chain narratives. Each should read like a brief incident report from the future — "here's how this attack played out." Include:
- How the attacker discovers the opportunity
- What they do to exploit it (specific steps)
- What the impact looks like in concrete terms
- Why current controls don't stop it (or do stop it, if they would)
- A "why this matters" closing that connects to business impact

**Mitigation Roadmap:**
Group mitigations by urgency (immediate / short-term / long-term) and present as a checklist. For each mitigation:
- What to do (specific and actionable)
- Which threats it addresses (by ID)
- Effort estimate (small / medium / large)
- Expected risk reduction

Highlight mitigations that address multiple threats — these are the highest-ROI investments.

**Hardening Recommendations:**
List findings the analysis phase classified as hardening recommendations rather than threats — generic best-practice suggestions with no specific exploitable weakness tied to this system's actual trust boundaries or data. Open with a sentence framing the distinction: these are good practice, not specific weaknesses in this system. For each: title, one-line rationale, suggested action. Do not give these a severity or likelihood rating — that would misrepresent them as threats. Omit this section if the analysis produced none.

**Deprioritized Threats:**
List threats that were identified but deemed acceptable. For each, explain the reasoning so future reviewers know it was a conscious decision, not an oversight.

## Output Format

Write the full narrative report as a Markdown file named `THREAT_MODEL.md` in the repository root (or the target directory if specified).

## PR Report Format

When generating a PR threat assessment (not a full THREAT_MODEL.md), use this lighter format:

```markdown
## PR Threat Assessment: [PR title or branch name]

### Change Summary
What changed and why it matters from a security perspective (2-3 sentences).

### New Threats Introduced
Threats that did not exist before this change. Each gets:
- Title, narrative (1-2 sentences), risk level, code references to the diff

### Existing Threats Affected
How this change impacts threats in the existing THREAT_MODEL.md (if one exists):
- Risk increased / decreased / resolved, with explanation

### Mitigations Impact
Existing mitigations that are broken, weakened, or invalidated by this change.

### Recommendation
**Approve** / **Approve with conditions** / **Request changes**
- Specific conditions or required changes listed

### Code References
All cited file:line locations from the diff.
```

Do NOT generate a full THREAT_MODEL.md for PR analysis — keep it focused on the delta.

## Update Mode

When updating an existing THREAT_MODEL.md rather than generating from scratch, follow these rules:

**Preserve continuity:**
- Keep existing threat IDs — never renumber. If T-003 is removed, the next new threat is still the next available ID, not T-003 reused. The same rule applies to hardening recommendation IDs (H-NNN) — they're a separate sequence from T-NNN, but follow identical preservation rules.
- Preserve human-written edits, annotations, and custom sections that aren't contradicted by the feedback.
- Keep the original provenance entry and append a new update entry.

**Handle changes:**
- **Removed threats**: Move to the "Deprioritized Threats" section with a "Removed" subsection. Include the removal reason and date. Don't delete — future reviewers need to know it was consciously removed.
- **Added threats**: Assign the next available threat ID. Write full narrative and details as for a new analysis.
- **Modified threats**: Update the specific fields that changed (risk level, mitigations, narrative). Note what changed in the threat's details.
- **Risk level changes**: Update the risk level and add a note explaining why it changed.
- **Hardening recommendations**: Handle with the same rigor as threats — added ones get the next available H-NNN, removed ones move to "Deprioritized Threats" with a reason (don't just delete), implemented ones get noted as resolved rather than silently dropped.
- **Reclassification**: If re-analysis determines a finding should move between Threats and Hardening Recommendations (e.g., new evidence shows an exposure is internet-facing after all), retire the old ID in its original section with a note explaining the reclassification, and assign a fresh ID in the new section — don't reuse the old ID across sections.

**Provenance update:**
Append an update history entry to the provenance section:

```
### Update History
| Date | Changes | Reason |
|---|---|---|
| YYYY-MM-DD | Removed T-003, upgraded T-005 to Critical, added T-012 | Review feedback from security team |
```

**Targeted re-analysis:**
Only re-analyze the parts that need updating. If feedback says "add a threat about the new Redis cache," run discovery and analysis only for the Redis-related components — don't regenerate the entire report.

## Quality Checks

Before finalizing the report:
- Every threat has a narrative scenario (not just a title and table)
- Every code-grounded threat has at least one `file:line` citation (absence-of-control threats note where the control should exist)
- Every threat maps to at least one actor with motivation
- Every threat has at least one recommended mitigation
- Every existing-control entry has a `file:line` citation or is explicitly marked `Unverified` — no bare assertions
- Every Critical/High severity rating is justified against the trust-boundary/exposure data, not just the nominal vulnerability
- Generic best-practice suggestions live in Hardening Recommendations, not mixed into Threats
- The executive summary mentions no jargon and is readable by a non-technical VP
- The mitigation roadmap is actionable (specific enough to become tasks)
- Deprioritized threats have documented rationale
- Open questions are listed (don't pretend you know everything)
- Provenance section is complete (date, frameworks, actors, scope, mode)
