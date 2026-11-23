---
name: subsea-geospatial-srlg
description: Use when building cable geometries, landfall links, basin or strait features, physical/logical graphs, or SRLG scenarios in SubseaBGP-Tomography.
---

# Geospatial and SRLG Modeling

Keep the physical cable graph and logical routing graph separate, with stable
IDs and an explicit mapping between them. Preserve the source geometry, license,
retrieval date, coordinate reference system, geometry validity result, and
topology hash. Use names such as `length_km`, `depth_m`, `lat_deg`, and
`lon_deg`; do not hide units in generic fields.

Construct segments deterministically from approved cable and landing-point
sources. Record antimeridian handling, simplification tolerance, snapping,
overlap rules, and the treatment of missing or disputed geometry. Do not infer a
physical route from a map image or invent a segment to fill a gap.

Apply the project SRLG corridor and bottleneck rules consistently, including the
25 km segment/risk resolution where the requirement calls for it. A shared
physical corridor is not automatically a shared logical path; document every
SRLG membership and its evidence. Freeze geometry and risk snapshots before a
historical prediction time.

Keep map rendering concerns separate from graph and solver data. Validate graph
connectivity, segment lengths, duplicate IDs, coordinate bounds, CRS conversion,
SRLG membership, and topology hashes before model or routing evaluation.
