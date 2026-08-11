---
description: Quick non-interactive threat assessment using STRIDE. Scans the current repo and produces a full threat model without asking questions.
argument-hint: "[target path] [actors:...] [context:...]"
---

Invoke the threat-model skill in quick mode.

Run a full, non-interactive threat assessment — auto-picks all options so the user doesn't have to answer questions:
1. If `context:` arguments are provided, ingest them first — these are docs the code scan won't find (architecture write-ups, compliance assessments, prior threat models, SBOMs). Merge with code analysis: prefer external docs for architectural intent, prefer code for implementation reality, flag discrepancies as findings.
2. Auto-discover system context from code (no interview questions)
3. Build a **data classification table** mapping each data type to sensitivity, storage, flows, and regulatory scope
4. Apply STRIDE framework
5. **Ground every threat in code** with `file:line` citations. For each threat, locate the specific file and line that makes it exploitable. For absence-of-control threats, note where the control should exist.
6. Auto-detect 2-3 most relevant threat actor personas
7. Find as many threats as the codebase warrants — do not cap or limit the count
8. Produce a full THREAT_MODEL.md following the report template, including all sections: executive summary, architecture overview, actor profiles, assets, data classification, attack surface, threats with narratives and code citations, mitigation roadmap, open questions, and provenance

This is the "autopilot" mode — same depth as guided, without the interview. Useful when you want a complete threat model without back-and-forth.

Optional arguments:
- A file or directory path to focus on (default: entire repo)
- `actors:insider,apt` to override auto-detected actor personas
- `context:file,dir/,url` for docs the code scan won't find (comma-separated; paths relative to repo root)

Arguments from user: $ARGUMENTS
