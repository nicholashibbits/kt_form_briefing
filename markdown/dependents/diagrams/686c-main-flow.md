# 686c — Main Flow

Source: `src/applications/dependents/686c-674/config/form.js` plus chapter modules under `config/chapters/`.

Edges are labeled with the gate that controls the transition. Issue badges in node labels reference the relevant epic; full links are in the [main briefing's 686c issue index](../kt_form_briefing.md#vaf21-686c--application-request-to-add-andor-remove-dependents).

```mermaid
flowchart TB
  Intro["Introduction page<br/>VA file # check"]:::backend

  subgraph VI["Chapter: Veteran Information"]
    direction LR
    VI1[veteran-information]:::frontend --> VI2[veteran-address]:::frontend --> VI3[veteran-contact-information]:::frontend
  end

  subgraph OS["Chapter: Option Selection — Manage Dependents"]
    direction TB
    OS1["review-dependents<br/>v3 picklist intro"]:::toggle
    OS2[options-selection]:::frontend
    OS3a["options-selection/<br/>add-dependents"]:::frontend
    OS3b["options-selection/<br/>remove-dependents"]:::frontend
    OS4["check-veteran-pension<br/>backup path"]:::backend
    OS5["Picklist remove options<br/>v3 only — #120229"]:::toggle
  end

  PicklistFollowups["Picklist Removal Followups<br/>per-dependent custom routing<br/>#108392 Phase 2 pending"]:::inProgress
  AddSpouse["Add Spouse — addSpouse<br/>~24 pages, marriage histories<br/>#124457 — #113855 reopened"]:::reopened
  AddChild["Add Child — report-add-child<br/>~13 pages, array builder<br/>#124457"]:::inProgress
  Add674["Add Student 18-23 — formConfig674<br/>#124457 — #114340 — #120876 reopened"]:::reopened

  subgraph RemoveV2["Remove flows — v2 only legacy (gated by noV3Picklist)"]
    direction LR
    RD[Report divorce]:::frontend --> SC[Stepchild left household]:::frontend --> DD[Deceased dependents]:::frontend --> CM[Child marriage under 18]:::frontend --> CS[Child stopped school 18-23]:::frontend
  end

  HHI["Household Income — net-worth"]:::backend
  AE1["add-spouse-evidence<br/>spouseEvidence() helper"]:::frontend
  AE2["add-child-evidence<br/>childEvidence() helper"]:::frontend

  Review["Review & Submit<br/>SubmitForm686cJob queues SubmitForm674Job<br/>#119718 FDF Pilot — #124480"]:::inProgress
  Conf["Confirmation page<br/>FDF viewer link<br/>toggle: dependentsEnableFormViewerMfe"]:::toggle

  Intro -->|"hasValidVAFileNumber (BGS)"| VI1
  VI3 --> OS1
  OS1 -->|"showV3Picklist() && hasAwardedDependents()"| OS5
  OS1 -->|"!showV3Picklist() OR no awarded deps"| OS2
  OS2 -->|"isAddingDependents()"| OS3a
  OS2 -->|"noV3Picklist() && isRemovingDependents()"| OS3b
  OS2 -->|"showPensionBackupPath()"| OS4
  OS5 -->|"hasSelectedPicklistItems()"| PicklistFollowups
  PicklistFollowups -.continues to add flows.-> AddSpouse
  OS3a --> AddSpouse
  AddSpouse --> AddChild
  AddChild --> Add674
  Add674 -->|"noV3Picklist()"| RemoveV2
  Add674 -->|"showV3Picklist()"| HHI
  RemoveV2 --> HHI
  HHI -->|"showPensionRelatedQuestions()"| AE1
  AE1 --> AE2
  AE2 --> Review
  Review --> Conf

  classDef frontend fill:#ffffff,stroke:#333333,stroke-width:1px;
  classDef backend fill:#dbeafe,stroke:#1e40af,stroke-width:1px,color:#1e3a8a;
  classDef toggle fill:#e9d5ff,stroke:#7e22ce,stroke-width:1px,stroke-dasharray:4 2,color:#581c87;
  classDef inProgress fill:#fef3c7,stroke:#d97706,stroke-width:3px,color:#78350f;
  classDef reopened fill:#fed7aa,stroke:#ea580c,stroke-width:4px,color:#7c2d12;
  classDef builtNotEnabled fill:#bbf7d0,stroke:#16a34a,stroke-width:1px,stroke-dasharray:6 3,color:#14532d;
```

## Reading notes

- **`review-dependents` → `Picklist remove options` vs `review-dependents` → `options-selection`** is the v2/v3 fork. `vaDependentV2Flow !== true` is the third state that decides which side a Veteran lands on; see the picklist deep-dive in the main briefing.
- **`Add Spouse` and `Add Student 18-23`** are marked reopened because they each carry an open reopened issue (#113855 and #120876 respectively).
- **The `check-veteran-pension` backup-path node** only appears when the pension API call fails — it's the manual fallback for `dependents_pension_check`.
- **`Picklist Removal Followups`** is one logical node here but in code it's a router; see [686c-picklist-subflow.md](686c-picklist-subflow.md).
