# 0538 — Main Flow

Source: `src/applications/dependents/dependents-verification/config/form.js`. Two real outcomes per the in-app README: (1) confirm dependents are up-to-date and submit, or (2) report status changed → ExitForm redirects to the 686C-674.

```mermaid
flowchart TB
  Intro[IntroductionPage]:::frontend
  Ch1[1. Veteran personal information — veteran-information]:::frontend

  subgraph Ch2["2. Veteran contact information"]
    direction LR
    B1["veteran-contact-information<br/>CustomPage"]:::frontend --> B2[editAddressPage]:::frontend --> B3[editEmailPage]:::frontend --> B4[editPhonePage]:::frontend --> B5[editInternationalPhonePage]:::frontend
  end

  subgraph Ch3["3. Review dependents"]
    direction LR
    C1["dependents — DependentsInformation custom page<br/>#108392 DV Phase 2 add removal capability"]:::inProgress --> C2["exit-form<br/>depends hasDependentsStatusChanged === 'Y'"]:::frontend
  end

  Intro --> Ch1 --> Ch2 --> Ch3
  Ch3 --> Review[Review & Submit]:::frontend --> Confirmation[Confirmation]:::frontend
  C2 -.->|"redirects to 686C-674"| Form686[Form 21-686C-674]:::backend

  classDef frontend fill:#ffffff,stroke:#333333,stroke-width:1px;
  classDef backend fill:#dbeafe,stroke:#1e40af,stroke-width:1px,color:#1e3a8a;
  classDef inProgress fill:#fef3c7,stroke:#d97706,stroke-width:3px,color:#78350f;
```

## Reading notes

- **The yellow-orange `dependents — DependentsInformation custom page` node** is where #108392 Phase 2 lands if removal capability gets built into the 0538 (rather than picklist in 686). Path TBD.
- **The `exit-form` node is a redirect, not a submission.** Veterans who say "yes, things changed" never submit a 0538 — they're sent to the 686. There's no audit trail of that intent in the 0538 backend.
- **No `defaultDefinitions`** — schemas live per-chapter; same as 0969.
- **The legacy embedded 0538** in `view-dependents/manage-dependents/` is a separate, dormant code path gated by `manageDependents`. Not shown here. Don't accidentally revive it.
