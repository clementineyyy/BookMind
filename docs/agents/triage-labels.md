# Triage Labels

The skills speak in terms of five canonical triage roles. These are the corresponding label strings used in this repository's GitHub Issues.

| Canonical role | GitHub label | Meaning |
| --- | --- | --- |
| `needs-triage` | `needs-triage` | Maintainer needs to evaluate this issue |
| `needs-info` | `needs-info` | Waiting on the reporter for more information |
| `ready-for-agent` | `ready-for-agent` | Fully specified and ready for an AFK agent |
| `ready-for-human` | `ready-for-human` | Requires human implementation |
| `wontfix` | `wontfix` | Will not be actioned |

When a skill mentions a role (e.g. "apply the AFK-ready triage label"), use the matching GitHub label from this table. Update the GitHub label column if the repository adopts different vocabulary.

Judge every issue against [`SCOPE.md`](../../SCOPE.md) before picking a state. Rejections close as "not planned".