# Engineering Traceability

This directory is the traceability control plane for the APMS engineering record.

It connects:

Requirement
→ Decision
→ Implementation
→ Test
→ Verification
→ Evidence
→ Claim

## Sources of Truth

| Domain | Authority |
|---|---|
| Requirement / Process | SA-1 |
| Architecture / Decision | SA-1 |
| Implementation | 26-05adf |
| Test | 26-05adf |
| Verification | 26-05adf / PR-1A1 |
| Evidence | PR-1A1 |
| Portfolio Claim | PR-1A1 |

## Repositories

- SA-1 — engineering process, requirements and decisions
- 26-05adf — actual system implementation and tests
- PR-1A1 — evidence, verification and portfolio projection

## Registry

`inventory.yaml` records discovered engineering artifacts.

`traceability.yaml` connects those artifacts into feature-level
requirement-to-claim traceability.

This directory does not create a parallel source of truth.
It references artifacts owned by the repositories above.