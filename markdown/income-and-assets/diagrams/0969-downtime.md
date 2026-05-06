# 0969 — Downtime & Off-ramps

The 0969's external surface is essentially nil today on the FE side — submission goes through the same legacy CentralMail-via-PDF route as 530EZ, scheduled to be replaced by direct BPDS via #121601.

```mermaid
flowchart TB
  Render[Form renders]:::recovery
  Render --> SaveCopy{"saveInProgress.messages<br/>COMMENTED OUT (config/form.js:44-50)"}:::offramp
  SaveCopy -->|expired SIP| DefaultCopy["Default platform copy<br/>not customized"]:::offramp
  SaveCopy -->|normal| Submit[Veteran submits]:::frontend

  subgraph SubmitToday["Today's submission path (legacy)"]
    direction LR
    JSON1[JSON] --> PDF1[PDF render] --> CMail[CentralMail intake] --> OCR1[OCR] --> BPDS1[(BPDS)]
  end

  subgraph SubmitFuture["#121601 future path (no assignee, milestone 2026-03-06)"]
    direction LR
    JSON2[JSON] --> BPDS2[(BPDS direct)]
  end

  Submit -->|"current production"| SubmitToday
  Submit -.->|"after #121601 ships<br/>frontend impact TBD"| SubmitFuture

  classDef downtime fill:#fecaca,stroke:#dc2626,stroke-width:2px,color:#7f1d1d;
  classDef offramp fill:#fed7aa,stroke:#ea580c,stroke-width:2px,color:#7c2d12;
  classDef recovery fill:#bbf7d0,stroke:#16a34a,stroke-width:1px,color:#14532d;
  classDef toggle fill:#e9d5ff,stroke:#7e22ce,stroke-width:1px,stroke-dasharray:4 2,color:#581c87;
  classDef frontend fill:#ffffff,stroke:#333333,stroke-width:1px;
  classDef backend fill:#dbeafe,stroke:#1e40af,stroke-width:1px,color:#1e3a8a;

  class JSON1,PDF1,CMail,OCR1,BPDS1,JSON2,BPDS2 backend
```

## Reading notes

- **No declared `downtime.dependencies`.** The 0969 doesn't declare external dependencies — different from 527EZ (icmhs only) and 686 (BGS/Profile/MVI/VBMS). If a Veteran hits a vets-api outage during submission, they'll see generic platform errors.
- **The `SaveCopy` orange node** is the commented-out `saveInProgress.messages` foot-gun. Veterans get the platform default copy when SIP expires; if you ever need to customize that copy, this is where to do it.
- **`SubmitFuture` is dotted** — same status as 530's BPDS work. No assignee on the epic; FE side TBD.
- **Email-notification toggles** (per the in-app README) suggest there's a notification path on submit/error/persistent-attachment-error, but the four toggles are not yet found in `featureFlagNames.json`. Not mapped here; ask the team.
