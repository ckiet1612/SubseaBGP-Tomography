---
name: subsea-anti-leakage
description: Use when changing dataset splits, features, labels, calibration, thresholds, replay windows, or evaluation in the SubseaBGP-Tomography pipeline.
---

# Anti-Leakage Protocol

Treat `V-LEAKAGE` as a release gate. Keep the five gates from
`docs/ARCHITECTURE.md` explicit and machine-checkable:

1. Split time-forward into `train`, `cal`, and `test` with a plus/minus 72-hour
   buffer; group independent semi-synthetic incidents.
2. Use Leave-One-Basin-Out for the four basins. Do not put raw latitude or
   longitude into the segment feature vector.
3. Apply the AIS 96-hour and SAR 120-hour masks before the event time and remove
   cable-repair vessels.
4. Freeze RFS lifecycle and every data snapshot strictly before the prediction
   time; keep the `2023-02` and `2026-08-25` checks.
5. Fit model parameters and `b_m` only on `train_fit`, estimate the 0.90
   conformal quantile on the locked 200-record calibration set, and never use
   `D_test` to select runtime or thresholds.

Scan for hidden future timestamps, duplicated incidents, shared entities across
splits, post-event metadata, and validation-driven feature selection. Missing
data stays explicit `NA`; never impute a value that changes observability or
turns an unobservable segment into a safe result. Disable physics-derived
features when their required approved data is absent.

Every run must emit the split manifest, masks, snapshot cutoffs, seed, selected
thresholds, and a zero-violation result. If a gate cannot be verified, mark the
run blocked rather than reporting a model score.
