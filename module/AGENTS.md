# Threat Model Module

AI-assisted threat modeling across six frameworks (STRIDE, PASTA, LINDDUN, VAST, Attack Trees, OCTAVE) with automatic threat actor profiling.

## Skills

- **threat-model**: Main dispatcher — routes to discover, analyze, and report sub-skills based on mode and arguments
- **threat-model-discover**: System context auto-discovery from code, external artifact ingestion, and optional user interview
- **threat-model-analyze**: Framework-based threat analysis with actor profiling and code-grounded citations
- **threat-model-report**: Narrative report generation following the THREAT_MODEL.md template
- **threat-model-review**: Review an existing THREAT_MODEL.md against current code state, with option to apply changes

## Commands

- `/threat-model` — Full threat modeling session (modes: quick, guided, full, pr, update)
- `/threat-model-quick` — Fast non-interactive STRIDE assessment
- `/threat-model-review` — Review and optionally update an existing threat model

## Agents

- `@threat-modeler` — Threat analysis subagent for parallel component or framework analysis

## Key Conventions

- Every threat must include `file:line` code citations grounding findings in the codebase
- Every report must include a Data Classification table (Section 5.5)
- External context (`context:` argument) is supported in quick, guided, and full modes
- PR mode produces a lightweight PR Threat Assessment, not a full THREAT_MODEL.md
- Update mode preserves existing threat IDs and appends to provenance history
