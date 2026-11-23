---
name: subsea-routing-verification
description: Use when implementing, changing, or evaluating the SR-TE/SRv6 LP solver, failure scenarios, capacities, or flow metrics in SubseaBGP-Tomography.
---

# Routing Verification

Treat the routing model as a research simulation, not a production network
controller. Keep the three lexicographic objectives explicit: minimize dropped
demand, then maximum link utilization, then weighted latency. Record solver
status, objective values, tolerances, infeasibility, and fallback behavior.

Validate every input at the boundary with units: `demand_tbps`, `capacity_tbps`,
`latency_ms`, and `utilization_pct`. Check non-negative flow, flow conservation
at every transit node, demand satisfaction or explicit dropped-flow variables,
capacity limits, SRLG constraints, and the selected failed segments. A solver
returning a status is not evidence that the solution is feasible.

Compare a scenario only with a shortest-path baseline using the same frozen
topology, demand matrix, capacities, latency data, and failure set. Report
dropped `Tbps`, maximum link utilization (`MLU`, `%`), weighted latency `ms`,
feasibility, and the exact topology/snapshot hash. Test no-failure, single-segment, shared-SRLG,
disconnected, zero-demand, and insufficient-capacity cases.

Keep LP core logic independent from data loading and UI. Do not silently repair
infeasible input, clamp a negative value, or substitute a default capacity;
return a typed error or an explicit degraded result that the caller can display.
