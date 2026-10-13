# Agentic Threat Modeling

A [Lola](https://github.com/LobsterTrap/lola) module for AI-assisted threat modeling. Works across Claude Code, Cursor, Gemini CLI, and other supported AI assistants.

Generate threat models that tell a story — not just tables of threats, but narratives that explain who is attacking, how, and why it matters.

## Features

- **Six frameworks**: STRIDE, PASTA, LINDDUN, VAST, Attack Trees, OCTAVE
- **Automatic threat actor profiling**: suggests likely attackers based on what your system stores, serves, and how it's built
- **Five modes**: quick scan, guided session, comprehensive analysis, PR-scoped review, and iterative update
- **Code citations**: every threat grounded in the codebase with `file:line` references
- **Data classification**: structured table linking data types to sensitivity levels, storage, flows, and regulatory scope
- **External context ingestion**: feed in architecture docs, OpenAPI specs, SBOMs, deployment configs, prior threat models, and SAR reports
- **Narrative output**: every threat includes an attack story, not just a risk rating
- **Kill-chain scenarios**: top threats get full attack narratives from reconnaissance to impact
- **Threat model review**: check if an existing threat model is still current, with option to apply changes directly
- **Iterative updates**: refine an existing threat model with feedback without regenerating from scratch

## Quick Start

### Install via Lola

```bash
lola mod add https://github.com/RedHatProductSecurity/agentic-threat-modeling.git
lola install agentic-threat-modeling -a claude-code
```

### Install Locally (Development)

```bash
git clone git@github.com:RedHatProductSecurity/agentic-threat-modeling.git
cd agentic-threat-modeling
lola mod add ./module
lola install agentic-threat-modeling -a claude-code
```

## Usage

### Commands

| Command | Description |
|---|---|
| `/threat-model` | Interactive threat modeling session. Asks scoping questions, recommends a framework, walks through discovery and analysis. |
| `/threat-model stride` | STRIDE analysis of the current repo |
| `/threat-model pasta src/auth/` | PASTA analysis focused on the auth module |
| `/threat-model quick` | Fast non-interactive scan using STRIDE |
| `/threat-model quick context:docs/openapi.yaml` | Quick scan with external context ingestion |
| `/threat-model full actors:all` | Comprehensive multi-framework analysis with all actor perspectives |
| `/threat-model pr` | Threat model the current branch's diff against main |
| `/threat-model pr:123` | Threat model a specific PR by number |
| `/threat-model update` | Refine an existing THREAT_MODEL.md interactively |
| `/threat-model update "remove T-003, upgrade T-005"` | Apply specific feedback to an existing model |
| `/threat-model-quick` | Shortcut for quick non-interactive assessment |
| `/threat-model-review` | Review an existing THREAT_MODEL.md against current code, with option to apply changes |
| `/threat-model-feedback` | Share feedback on a generated threat model by filing a GitHub issue |

### Framework Selection

| Framework | Best For |
|---|---|
| **STRIDE** | General-purpose threat identification. Default for most projects. |
| **PASTA** | Risk-centric analysis with business impact. Good for compliance-driven work. |
| **LINDDUN** | Privacy threat analysis. Use when PII, GDPR, or CCPA are concerns. |
| **VAST** | Agile/DevOps integration. Produces sprint-ready findings. |
| **Attack Trees** | Deep-dive on specific high-value attack scenarios. |
| **OCTAVE** | Organizational risk assessment across systems. |

### Threat Actor Personas

The skill automatically suggests relevant threat actors based on your system. You can also specify actors explicitly:

```
/threat-model stride actors:insider,supply-chain
/threat-model actors:all
```

Available actors: `apt` (nation-state), `crime` (organized crime), `insider` (malicious insider), `negligent` (negligent insider), `hacktivist`, `opportunistic`, `competitor`, `supply-chain`.

## Output

The primary output is a `THREAT_MODEL.md` file with:

1. **Executive summary** — plain-language overview for any audience
2. **Architecture overview** — how the system works
3. **Threat actor landscape** — who's targeting this system and why
4. **Assets & crown jewels** — what's worth protecting
5. **Data classification** — table linking data types to sensitivity, storage, flows, and regulatory scope
6. **Attack surface** — entry points and trust boundaries
7. **Threats** — each with a narrative attack scenario, `file:line` code citations, risk rating, and recommended mitigations
8. **Attack scenarios** — full kill-chain narratives for the top threats
9. **Mitigation roadmap** — prioritized by risk reduction per effort
10. **Open questions** — unknowns that could change the picture

PR mode produces a lighter **PR Threat Assessment** with change summary, new threats, existing threats affected, mitigations impact, and an approve/conditions/request-changes recommendation.

## Module Structure

```
module/
├── skills/
│   ├── threat-model/              # Main dispatcher
│   ├── threat-model-discover/     # System context auto-discovery + interview
│   ├── threat-model-analyze/      # Framework-based threat analysis
│   │   └── reference/             # Framework methodology guides
│   ├── threat-model-report/       # Narrative report generation
│   │   └── templates/             # Output template
│   └── threat-model-review/       # Review existing threat models
├── commands/                      # Slash commands (/threat-model, etc.)
└── agents/                        # Threat modeler subagent
```

## Feedback

Run `/threat-model-feedback` after a threat model is generated, or [file feedback directly](https://github.com/RedHatProductSecurity/agentic-threat-modeling/issues/new?template=feedback.yml).

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines on adding frameworks, improving prompts, or extending the module.

## License

Apache License 2.0 — see [LICENSE](LICENSE).
