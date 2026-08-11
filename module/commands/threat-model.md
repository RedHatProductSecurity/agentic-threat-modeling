---
description: Generate a threat model for the current project or a specific target. Supports multiple frameworks and threat actor perspectives.
argument-hint: "[framework] [mode] [actors:...] [target]"
---

Invoke the threat-model skill to generate a threat model.

Parse the user's arguments to determine:

**Framework** (optional — if not specified, recommend one):
- `stride` — STRIDE (Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, Elevation of Privilege)
- `pasta` — PASTA (Process for Attack Simulation and Threat Analysis)
- `linddun` — LINDDUN (privacy-focused threat modeling)
- `vast` — VAST (Visual, Agile, Simple Threat modeling)
- `attack-trees` — Attack Trees (goal-decomposition analysis)
- `octave` — OCTAVE (organizational risk assessment)

**Mode** (optional — default: guided):
- `quick` — fast, non-interactive scan using STRIDE
- `guided` — interactive session with auto-discovery + interview
- `full` — comprehensive multi-framework analysis
- `pr` — threat-model the current branch's diff against main
- `pr:123` — threat-model a specific PR by number
- `update` — refine an existing THREAT_MODEL.md with feedback
- `update "remove T-003, upgrade T-005 to Critical"` — apply specific feedback inline

**Actor personas** (optional — default: auto-detect):
- `actors:insider` — single actor focus
- `actors:insider,supply-chain,apt` — multiple actors
- `actors:all` — all actor perspectives

**External context** (optional — for docs the code scan won't find: architecture write-ups, compliance assessments, prior threat models, SBOMs):
- `context:docs/ess_controls.md` — a file path (relative to repo root)
- `context:docs/` — a directory (ingests all supported files)
- `context:https://example.com/arch` — a public URL (authenticated pages require exporting first)
- Comma-separated for multiple: `context:docs/ess_controls.md,docs/sar_report.md`

**Target** (optional — default: current repository):
- A file or directory path to focus on

**Always-on features** (required in every mode — quick, guided, full):
- **Code citations**: every threat must include `file:line` references grounding findings in the codebase. For absence-of-control threats, note where the control should exist.
- **Data classification table**: Section 5.5 maps data types to sensitivity, storage, flows, and regulatory scope. Use this table to calibrate threat severity.

Examples:
  - `/threat-model context:docs/openapi.yaml` — ingest API spec before analysis
  - `/threat-model stride context:https://docs.google.com/doc/d/... context:sbom.json` — STRIDE with external arch doc and SBOM

Arguments from user: $ARGUMENTS

If no arguments provided, start an interactive guided session. Ask the user to choose framework, external context, and scope BEFORE scanning the codebase — don't start analysis until they've made their selections.
