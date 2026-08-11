---
description: Review an existing THREAT_MODEL.md against the current code state. Identifies drift, stale threats, and new attack surface.
argument-hint: "[path to THREAT_MODEL.md]"
---

Invoke the threat-model-review skill.

Find and review an existing threat model:
1. Locate THREAT_MODEL.md (in the repo root, or at the path specified by the user)
2. Parse its contents — threats, mitigations, assumptions, scope
3. Analyze what has changed in the codebase since the threat model was written
4. Identify:
   - **Stale threats**: threats that no longer apply due to code changes or mitigations implemented
   - **New attack surface**: new endpoints, components, data flows, or integrations not covered by the existing threat model
   - **Drift**: assumptions that are no longer valid
   - **Open mitigations**: recommended mitigations that haven't been implemented yet
5. Produce an update report with recommended changes to the threat model
6. After presenting the review findings, offer to apply the changes directly:
   > "Would you like me to apply these changes to the threat model now? I can update the THREAT_MODEL.md to incorporate the findings — removing stale threats, adjusting risk levels, and adding coverage for new attack surface. You'll be able to review the changes before they're finalized."
   If the user agrees, apply changes following update mode rules: preserve existing threat and hardening-recommendation IDs (T-NNN and H-NNN alike, never renumber), move removed threats or resolved/no-longer-relevant hardening recommendations to Deprioritized Threats with removal reason and date, preserve human-written edits, and append an Update History entry to the provenance section.

Arguments from user: $ARGUMENTS

If no THREAT_MODEL.md is found, suggest running `/threat-model` to create one.
