---
name: subsea-evaluation
description: Use when adding models, baselines, ablations, calibration, historical replay, or routing metrics for SubseaBGP-Tomography.
---

# Evaluation Protocol

Use the same frozen snapshots, split manifest, seed, and scoring code for every
baseline, ablation, quantized model, and full system. Do not tune a model or
threshold on the test set. Read `docs/REQUIREMENTS.md` and
`docs/ARCHITECTURE.md` for the active acceptance gate before running.

For the MVP, preserve the 24-segment/48-channel semi-synthetic baseline and its
independent calibration and test records. Report sample counts, event grouping,
confidence intervals, PR-AUC or the required localization metric, conformal
coverage and set size, false alarms, misses, and all `NA`/unobservable cases.
Never claim 90% coverage before calibration has been run and checked.

For routing, report dropped demand in `Tbps`, maximum link utilization in `%`,
weighted latency in `ms`, solver status, feasibility, and flow-conservation
violations. Compare against the shortest-path baseline on the identical frozen
topology, demand, capacities, and failed segments. State whether a number is
observed, semi-synthetic, simulated, or assumed.

Keep ablations named and reproducible (`B1`, `B2`, `A1`, `A2`, `Full` where
applicable). A result is not complete until the command, configuration, seed,
commit, artifact hashes, and known limitations are recorded.
