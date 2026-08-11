---
name: threat-model-analyze
description: Core threat analysis engine. Takes a system context profile and applies one or more threat modeling frameworks (STRIDE, PASTA, LINDDUN, VAST, Attack Trees, OCTAVE) with automatic threat actor profiling.
allowed-tools:
  - Read
  - Bash
---

# Threat Analysis Engine

You are performing the analysis phase of threat modeling. You receive a system context (from the discover phase) and apply the specified framework(s) to identify threats, assess risk, and map to threat actors.

## Inputs

1. **System context** — the structured profile from the discover phase (components, data flows, trust boundaries, assets, entry points)
2. **Framework(s)** — which framework(s) to apply (default: STRIDE)
3. **Actor personas** — specific actors to focus on, or "auto" for automatic selection (default: auto)
4. **Depth** — quick (surface-level), standard (thorough), or deep (exhaustive with attack trees for top threats)

## Analysis Workflow

### Step 1: Load Framework Reference
Read the appropriate framework reference file(s) from `./reference/`:
- `./reference/stride.md` for STRIDE
- `./reference/pasta.md` for PASTA
- `./reference/linddun.md` for LINDDUN
- `./reference/vast.md` for VAST
- `./reference/attack-trees.md` for Attack Trees
- `./reference/octave.md` for OCTAVE

Always also load `./reference/threat-actors.md` for actor profiling.

### Step 2: Select Threat Actors
If actors are set to "auto", analyze the system context to suggest relevant actors:

1. **Examine data stored**: Map data types to actor profiles using the heuristics in threat-actors.md
2. **Examine system exposure**: Internet-facing vs. internal, user base, industry
3. **Examine technology stack**: Dependency footprint, cloud vs. on-prem, legacy vs. modern
4. **Examine compliance context**: Regulatory requirements signal which actors are relevant

Select the 2-4 most relevant actors and document why each was selected. Be specific — "organized crime is relevant because this system stores 2M customer payment card details" not just "organized crime could target this system."

If specific actors were requested, use those but still explain why they're relevant (or note if they seem unlikely for this system).

### Step 3: Apply the Framework

Follow the methodology defined in the framework reference file. Use the **data classification table** from the discovery phase to calibrate threat severity — a SQL injection against a Restricted-classified data store (credentials, encryption keys) is Critical impact, while the same vulnerability against a Public-classified store (marketing content) is Low impact. Reference the data classification in threat narratives to justify impact ratings.

The approach differs by framework:

**STRIDE:** Walk each system element (component, data flow, data store) through all six categories. For each element, ask whether Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, or Elevation of Privilege applies. Refer to the reference file for per-category questions and attack patterns.

**PASTA:** Work through all seven stages sequentially. Stage 1 uses business context from the system profile. Stages 2-3 use the technical architecture. Stages 4-6 require threat intelligence and attack modeling. Stage 7 produces risk scoring.

**LINDDUN:** Walk each element that processes personal data through all seven privacy categories. Focus on data flows that carry PII and data stores that hold it. Refer to the reference file for per-category questions.

**VAST:** Build Application Threat Models (process flow analysis) and/or Operational Threat Models (infrastructure analysis). Use the VAST-specific threat categories from the reference file.

**Attack Trees:** For each high-value asset or critical threat scenario, build a tree decomposing the attacker's goal into sub-goals and leaf nodes. Annotate with cost, skill, detectability. Identify the cheapest attack paths.

**OCTAVE:** Work through the eight steps, starting with risk measurement criteria and progressing through asset profiling, threat scenarios, and risk analysis. This framework requires organizational context beyond the code.

### Step 3.5: Ground Findings in Code
Every identified threat must be grounded in the actual codebase with specific file paths and line numbers. For each threat:

1. **Locate the vulnerable code** — find the specific file and line that makes this threat exploitable. Use `grep`, `find`, or read relevant source files to pinpoint the exact location.
2. **Cite in `file:line` format** — e.g., `src/auth/login.py:87`. Include a brief context note explaining what the code does that creates the vulnerability.
3. **Multiple citations** — if a threat spans multiple files (e.g., missing auth middleware across several routes), cite each relevant location.
4. **Architectural threats** — if the threat is architectural with no single exploitable line (e.g., missing network segmentation), cite the most relevant files: deployment configs, route definitions, or infrastructure-as-code files.
5. **Absence-of-control threats** — if the threat is about missing functionality (e.g., no rate limiting exists), state: "No code citation — this is an absence of controls. Expected location: `[where it should be]`"

Add `code_references` to each threat's output:
```
code_references:
  - file: src/auth/login.py
    line: 87
    context: "Login endpoint accepts credentials without rate limiting"
  - file: src/middleware/auth.py
    line: 12
    context: "JWT validation skips expiration check"
```

Code citations transform findings from theoretical concerns into actionable items a developer can navigate to directly.

**Existing controls must be verified the same way.** Before listing anything under `current_controls`, confirm it actually exists in the codebase — grep for the config flag, setting, or code path, don't assert it because it would be reasonable to have. If a claimed control can't be located, do not list it as an existing control. Instead mark it explicitly: `Unverified — no evidence found in codebase (expected: [where it should be])`, and score the threat's likelihood/impact as if the control does not exist. Every `current_controls` entry needs either a `file:line` citation or the `Unverified` label — never a bare assertion.

### Step 3.6: Verify Technical Mechanism Claims
Before asserting what a config value, permission mode, decorator, flag, or library call actually does, confirm the real semantics — don't infer from what it looks like. Watch for these specific traps:
- **Numeric file-mode/permission fields**: confirm decimal-vs-octal interpretation for the platform. Kubernetes `defaultMode`/`fsGroup` parse YAML integers as decimal, so `444` is octal `0674` (`rw-rwxr--`), not world-readable `0444` (`r--r--r--`).
- **Transaction/concurrency primitives**: confirm what a guarantee actually prevents vs. causes — e.g. `@transaction.atomic` prevents partial/torn writes, it doesn't cause them.
- **Auth/library defaults**: confirm the behavior of the actual version in use, not a remembered or assumed default.

When unsure, check the platform/library documentation or, if a running instance is available, verify observed behavior before stating an effect as fact. If a technical claim can't be verified, say so explicitly in the narrative rather than asserting it as fact.

### Step 4: Map Threats to Actors
For each identified threat, determine:
- Which actor persona(s) would exploit this threat
- Their specific motivation for this threat (not generic — tied to what they'd gain from THIS system)
- How this threat fits into their typical TTPs
- Whether this threat would be a primary target or an opportunistic find

### Step 5: Write Attack Narratives
For each threat, write a concise scenario (3-5 sentences) that tells the story:
- Who is the attacker? (Use the mapped actor persona)
- What do they do? (Specific technical actions)
- What happens? (Impact in concrete terms — "2M records exposed", not "data breach")
- Why should we care? (Business consequence)

For the top 3-5 threats, expand into full kill-chain narratives covering reconnaissance through impact.

### Step 5.5: Classify Each Finding — Threat or Hardening Recommendation
Not every candidate finding is a threat. Before assessing risk or recommending mitigations, classify each one as a **threat** only if there's a specific exploitable weakness given this system's actual trust boundaries, data, and actors — not "this deviates from a general best practice."

Route a finding to **hardening recommendations** instead when:
- The "attacker" is a trusted upstream the organization also operates and controls (e.g., validating responses from an internal service already reached over authenticated channels).
- The suggestion is defense-in-depth for a control that's redundant with a platform guarantee already in place (e.g., recommending a Kubernetes SecurityContext when OpenShift SCCs already enforce the equivalent restriction).
- The fix is constrained by a third party's own model, not a choice this system made (e.g., "migrate to OAuth" when Basic Auth is the only mechanism the vendor offers).
- The finding has no specific narrative tying it to this system's actors or data beyond "this is generally good practice."

Hardening recommendations get a short rationale and a concrete suggested action right now, in this step. They do **not** proceed to Step 6 or Step 7 — do not assign them a severity/likelihood/risk_level rating or a full mitigation entry, since either implies a real, exploitable threat, which this isn't. Only findings classified as threats continue to Step 6.

### Step 6: Assess Risk
For each threat (not hardening recommendation — those were already handled in Step 5.5), assess:

**Likelihood** (considering):
- Is this attack pattern well-known and tooled?
- Does the relevant actor have the capability?
- Are existing controls effective?
- Is the attack surface exposed?

**Impact** (considering):
- What data or functionality is affected?
- How many users/records are impacted?
- What's the regulatory/compliance consequence?
- What's the reputational damage?

**Calibrate against trust boundaries, not just data classification.** Just as the data classification table calibrates impact, use the **trust boundary and entry-point exposure** from the discovery phase to calibrate likelihood. A flaw behind an internal-only, authenticated boundary (e.g., an internal service sitting behind Kerberos or mTLS ingress) has materially lower likelihood than the identical flaw on an internet-facing endpoint — reflect that in the likelihood rating itself, not just a caveat in the prose. Do not rate a threat Critical or High on exposure alone without checking what boundary sits in front of it.

Do not treat a documented, deliberate design decision as a misconfiguration-driven finding. Check the discovery phase's **Assumptions and Design Decisions** section — if it records that something is intentional and sourced (e.g., "internal-only, so DRF permissions are deliberately open — confirmed via interview with service owner"), don't inflate its severity as if it were an oversight. If it's still worth raising, it should have been routed to hardening recommendations in Step 5.5, not scored here as a threat. If discovery didn't populate that section for this finding, treat it as an unconfirmed assumption, not a deliberate decision.

**Risk Level:**
| | Impact: Low | Impact: Medium | Impact: High | Impact: Critical |
|---|---|---|---|---|
| Likelihood: High | Medium | High | Critical | Critical |
| Likelihood: Medium | Low | Medium | High | Critical |
| Likelihood: Low | Low | Low | Medium | High |

### Step 7: Recommend Mitigations
For each threat (not hardening recommendation — those already have a suggested action from Step 5.5), suggest specific mitigations:
- What control would reduce the risk?
- How much would it reduce likelihood and/or impact?
- What's the implementation effort (small/medium/large)?
- What priority (immediate/short-term/long-term)?

Group mitigations that address multiple threats — a single mitigation that closes three threats is more valuable than three single-purpose fixes.

## Output Format

Produce structured threat findings that feed into the report phase:

For each threat:
```
- id: T-NNN
  title: Short descriptive title
  narrative: 3-5 sentence attack scenario
  framework_category: "STRIDE: Spoofing" (or appropriate)
  affected_components: [list]
  attack_vector: Network/Local/Physical/Insider/Supply Chain
  actors:
    - id: actor-id
      motivation: Why this actor would do this
  impact: critical/high/medium/low
  likelihood: high/medium/low
  risk_level: critical/high/medium/low
  code_references:
    - file: path/to/file.py
      line: 42
      context: "Brief description of what this code does that creates the vulnerability"
  current_controls:
    - description: What exists
      citation: "path/to/file.py:42" (or "Unverified — no evidence found in codebase (expected: [location])")
  recommended_mitigations: [what to add]
```

For each hardening recommendation:
```
- id: H-NNN
  title: Short descriptive title
  rationale: Why this is worth doing, and why it's advisory rather than a threat
  suggested_action: What to do
```

Also produce:
- Actor landscape summary (who's relevant and why)
- Top 3-5 kill-chain narratives
- Grouped mitigation recommendations
- Hardening recommendations (best practices, not system-specific threats)
- Deprioritized threats with rationale

## Depth Adjustments

**Quick mode:**
- STRIDE only, thorough analysis (same depth as standard, but non-interactive)
- 2-3 actor personas (auto-selected)
- Full narratives for all threats
- No threat count cap — find everything the codebase warrants
- No kill-chain narratives (save for standard/deep)

**Standard mode:**
- Selected framework(s), thorough analysis
- 2-4 actor personas with rationale
- Full narratives for all threats
- Kill-chain narratives for top 3
- Complete mitigation roadmap

**Deep mode:**
- Selected framework(s), exhaustive analysis
- All relevant actor personas
- Full narratives with evidence from code
- Kill-chain narratives for top 5
- Attack trees for highest-risk scenarios
- Mitigation roadmap with effort estimates and risk reduction scores

## PR-Scoped Analysis

When analyzing a PR/diff rather than a full codebase, the scope and output are fundamentally different:

**Scope:** Only analyze changed components and their trust boundary crossings. Don't re-analyze the entire system.

**Focus questions:**
- Does this change introduce new threats that didn't exist before?
- Does it change the risk level of any existing threats (if a THREAT_MODEL.md exists)?
- Does it break or invalidate any existing mitigations?
- Does it expand the attack surface (new endpoints, new data stores, new trust boundary crossings)?

**Output:** A lighter "PR Threat Assessment" — not a full THREAT_MODEL.md:
- **Summary**: what changed and the security implications (2-3 sentences)
- **New threats**: threats introduced by this specific change, with code references to the diff
- **Existing threats affected**: risk increased, decreased, or resolved
- **Mitigations impact**: existing mitigations broken or invalidated by this change
- **Recommendation**: one of:
  - **Approve** — no security concerns
  - **Approve with conditions** — minor concerns, list specific conditions
  - **Request changes** — security issues that should be resolved before merge

Use STRIDE by default for PR analysis unless a specific framework was requested. Keep actor mapping lightweight — focus on the most relevant 1-2 actors for the changed components.

## Multi-Framework Analysis

When multiple frameworks are specified:
1. Run each framework independently against the system context
2. Deduplicate threats found by multiple frameworks (keep the richest description)
3. Note where frameworks provide complementary perspectives on the same threat
4. Combine actor mappings across frameworks
5. Produce a unified threat list and mitigation roadmap
