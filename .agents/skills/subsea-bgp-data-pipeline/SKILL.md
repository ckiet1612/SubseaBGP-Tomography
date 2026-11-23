---
name: subsea-bgp-data-pipeline
description: Use when ingesting, parsing, normalizing, or joining BGP, minRTT, outage, or measurement data for SubseaBGP-Tomography.
---

# BGP Data Pipeline

Keep ingestion, normalization, feature construction, and model input separate.
Use only declared read-only public sources and preserve source provenance. Store
timestamps in UTC with the source timestamp and retrieval timestamp distinct.
Carry collector, peer, prefix, path, protocol, source, snapshot, and parser
version fields far enough to reproduce a derived record.

Normalize BGP updates and MRT archives into explicit five-minute windows without
silently dropping withdrawals, duplicate announcements, malformed records, or
collector gaps. Tag known hijack or route-leak evidence separately from cable
fault evidence. Do not treat a BGP change alone as physical confirmation.

Compute minRTT from permitted passive or already-published measurements. Never
add active probing or network-control traffic. Preserve the measurement count,
invalid count, units, window bounds, and quality status. Missing or incompatible
channels remain `NA` and must flow into the contract status; do not replace them
with zero, a median, or an implicit healthy value.

Every transformation needs a schema, unit, timezone, filter, source manifest,
and deterministic test fixture. Hash snapshots and name windows with stable
`snapshot_id` and `window_id` values so replay and leakage checks can refer to
the exact input.
