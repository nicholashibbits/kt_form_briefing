# KT — Form Briefing

Knowledge transfer docs for the DMC / PBB / LSS form portfolio in `vets-website`. Written for the engineer taking over. All file paths and issue links were verified against `vets-website` `main` as of 2026-04-30.

> **Diagrams** are written in Mermaid and render inline on GitHub — no extra tooling needed.

---

## Start here

| Document | What it is |
|---|---|
| [KT Form Briefing](markdown/kt_form_briefing.md) | Master document. Per-form breakdowns covering issue index, feature toggles, in-flight work, known gotchas, and external dependencies. Read this first. |
| [Open Questions](markdown/kt-questions.md) | Unresolved questions to bring to the outgoing team — one section per form, with pointers to where the READMEs partially answer each one. |

---

## Forms

### Dependents

| Document | What it is |
|---|---|
| [Team Overview](markdown/dependents/original-readmes/team-overview.md) | Dependents Management team scope: apps owned, shared infrastructure, and how 686c, 674, 0538, and view-dependents relate to each other. |
| [686c — Add/Remove Dependents](markdown/dependents/original-readmes/686c-674.md) | In-app README for VAF21-686c. Covers app structure, picklist (v2 vs v3), feature toggles, async BGS/RBPS path, open epics, and known issues. |
| [Picklist (v2 → v3)](markdown/dependents/original-readmes/picklist.md) | Standalone README for the 686c picklist component. Documents the v2/v3 architecture split and the migration in progress under #120229. |
| [674 — School Attendance](markdown/dependents/original-readmes/686c-674.md) | 674 lives inside the 686c-674 app. See the 686c README above; 674-specific context is in its own section there. |
| [674 Async VetsAPI Doc](markdown/dependents/original-readmes/674-async-vetsapi-doc.md) | Technical reference for the 674 async submission path through vets-api. |
| [686c Async VetsAPI Doc](markdown/dependents/original-readmes/686c-async-vetsapi-doc.md) | Technical reference for the 686c async submission path through vets-api. |
| [0538 — Dependents Verification](markdown/dependents/original-readmes/dependents-verification.md) | In-app README for VAF21-0538. Covers the prefill-required flow, the suspicious commented-out downtime config, and the relationship to 686c for removals. |
| [View Dependents](markdown/dependents/original-readmes/view-dependents.md) | In-app README for the view-dependents app. Covers the diary-date localStorage pattern, the never-released embedded 0538, and the `dependents_module_enabled` endpoint toggle. |

**Diagrams**

| Diagram | What it shows |
|---|---|
| [686c — Main Flow](markdown/dependents/diagrams/686c-main-flow.md) | Chapter-by-chapter flowchart of the 686c form, including picklist routing and `depends:` gates. |
| [686c — Off-ramps & Downtime](markdown/dependents/diagrams/686c-offramp.md) | Three failure-mode classes for the 686: ITF errors, async submission failures, and platform downtime. |
| [674 — Main Flow](markdown/dependents/diagrams/674-main-flow.md) | Array-builder flow for adding students; only renders when the "add student" task is selected in the picklist. |
| [0538 — Main Flow](markdown/dependents/diagrams/0538-main-flow.md) | Two-outcome flow: confirm dependents are current and submit, or report a change and redirect to 686c. |
| [0538 — Downtime & Off-ramps](markdown/dependents/diagrams/0538-downtime.md) | Documents the missing downtime config and what that means for BGS-dependent prefill. |

---

### Burials

| Document | What it is |
|---|---|
| [530EZ — Burial Benefits](markdown/burials/original-readmes/530ez.md) | In-app README for VAF21P-530EZ. Covers the mid-migration submission pipeline (CentralMail → BPDS), the `burialPdfFormAlignment` toggle, the session-storage mirroring pattern in `BurialsApp.jsx`, and open epics. |

**Diagrams**

| Diagram | What it shows |
|---|---|
| [530EZ — Main Flow](markdown/burials/diagrams/530ez-main-flow.md) | Five-chapter flowchart with `depends:` gates driven by `claimedBenefits`, `relationshipToVeteran`, and the PDF alignment toggle. |
| [530EZ — Downtime & Off-ramps](markdown/burials/diagrams/530ez-downtime.md) | Current CentralMail path and the in-progress BPDS migration under #121603. |

---

### Income & Assets

| Document | What it is |
|---|---|
| [527EZ — Pension](markdown/income-and-assets/original-readmes/527ez-pensions.md) | In-app README for VAF21P-527EZ. Covers the unusually thin downtime declaration, the disability-rating alert toggle, and open epics around the PDF revision. |
| [0969 — Income and Asset Statement](markdown/income-and-assets/original-readmes/0969.md) | In-app README for VAF21P-0969. Covers the four email-notification toggles missing from the central registry, the commented-out JSON schema import, and the BPDS migration under #121601. |

**Diagrams**

| Diagram | What it shows |
|---|---|
| [527EZ — Main Flow](markdown/income-and-assets/diagrams/527ez-main-flow.md) | Chapter-level flowchart (6 chapters, ~25 pages each); page-level detail lives in `config/chapters/`. |
| [527EZ — Downtime & Off-ramps](markdown/income-and-assets/diagrams/527ez-downtime.md) | Single external dependency (`icmhs`) and open question about whether BGS/MVI are actually in the prefill path. |
| [0969 — Main Flow](markdown/income-and-assets/diagrams/0969-main-flow.md) | Heavy array-builder flow across sections 2–11. |
| [0969 — Downtime & Off-ramps](markdown/income-and-assets/diagrams/0969-downtime.md) | Minimal current external surface; what changes when BPDS takes over submission. |

---

## Folder structure

```
markdown/
├── kt_form_briefing.md              ← master KT doc
├── kt-questions.md                  ← open questions for the outgoing team
├── burials/
│   ├── diagrams/
│   │   ├── 530ez-main-flow.md
│   │   └── 530ez-downtime.md
│   └── original-readmes/
│       └── 530ez.md
├── dependents/
│   ├── diagrams/
│   │   ├── 686c-main-flow.md
│   │   ├── 686c-offramp.md
│   │   ├── 674-main-flow.md
│   │   ├── 0538-main-flow.md
│   │   └── 0538-downtime.md
│   └── original-readmes/
│       ├── team-overview.md
│       ├── 686c-674.md
│       ├── picklist.md
│       ├── 674-async-vetsapi-doc.md
│       ├── 686c-async-vetsapi-doc.md
│       ├── dependents-verification.md
│       └── view-dependents.md
└── income-and-assets/
    ├── diagrams/
    │   ├── 527ez-main-flow.md
    │   ├── 527ez-downtime.md
    │   ├── 0969-main-flow.md
    │   └── 0969-downtime.md
    └── original-readmes/
        ├── 527ez-pensions.md
        └── 0969.md
```
