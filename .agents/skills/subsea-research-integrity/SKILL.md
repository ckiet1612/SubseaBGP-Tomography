---
name: subsea-research-integrity
description: Use when adding or changing scientific claims, citations, source data, experiment reports, baselines, or reproducibility records in SubseaBGP-Tomography.
---

# Research Integrity

Keep the project distinguishable between measured results, semi-synthetic data,
simulation output, and assumptions. Read `AGENTS.md`, `docs/REQUIREMENTS.md`,
`docs/RESEARCH.md`, and `docs/ARCHITECTURE.md` before changing a scientific
statement or evaluation protocol.

For every external source or dataset, record the primary publisher, exact URL or
DOI, license, version or retrieval date, time zone, preprocessing, filters,
schema, and any rate or access constraint. Prefer primary sources. Do not copy
old reported numbers into a new result without rerunning the protocol.

When documents, code, and data disagree, record the discrepancy and its impact;
do not silently choose the interpretation that changes scope or conclusions.
Label every result as observed, semi-synthetic, simulated, or assumed. Keep
claims proportional to the evidence: a passing parser, HTTP response, or local
run does not establish scientific validity or browser readiness.

Before handoff, require a compact provenance record containing the source
manifest, commit, dependency lock, command, configuration, seed, split, output
hashes, and unresolved limitations. Never invent citations, values, licenses,
or replication claims.
