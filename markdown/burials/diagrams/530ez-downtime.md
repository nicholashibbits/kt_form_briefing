# 530EZ — Downtime & Off-ramps

The 530's submission pipeline is mid-migration. Today: JSON → PDF → CentralMail → OCR → BPDS. Eventually (#121603): JSON → BPDS directly.

```mermaid
flowchart TB
  subgraph Preload["Pre-load — declared downtime"]
    icmhs[(icmhs)]:::downtime
  end

  Preload -->|down| Block[Form blocks rendering]:::downtime
  Preload -->|up| Render[Form renders]:::recovery

  Render --> Submit[Veteran submits]:::frontend

  subgraph SubmitToday["Today's submission path (legacy)"]
    direction LR
    JSON1[JSON] --> PDF1[PDF render] --> CMail[CentralMail intake] --> OCR1[OCR] --> BPDS1[(BPDS)]
  end

  subgraph SubmitFuture["#121603 future path (no assignee)"]
    direction LR
    JSON2[JSON] --> BPDS2[(BPDS direct)]
  end

  Submit -->|"current production"| SubmitToday
  Submit -.->|"after #121603 ships<br/>frontend impact TBD"| SubmitFuture

  Render --> SessionFork{"showPdfFormAlignment()<br/>reads sessionStorage"}:::toggle
  SessionFork -->|"sessionStorage cleared mid-flow"| SilentChange["Conditional pages silently change<br/>FOOTGUN"]:::offramp
  SessionFork -->|"sessionStorage stable"| Normal[Normal flow]:::recovery

  classDef downtime fill:#fecaca,stroke:#dc2626,stroke-width:2px,color:#7f1d1d;
  classDef offramp fill:#fed7aa,stroke:#ea580c,stroke-width:2px,color:#7c2d12;
  classDef recovery fill:#bbf7d0,stroke:#16a34a,stroke-width:1px,color:#14532d;
  classDef toggle fill:#e9d5ff,stroke:#7e22ce,stroke-width:1px,stroke-dasharray:4 2,color:#581c87;
  classDef frontend fill:#ffffff,stroke:#333333,stroke-width:1px;
  classDef backend fill:#dbeafe,stroke:#1e40af,stroke-width:1px,color:#1e3a8a;

  class JSON1,PDF1,CMail,OCR1,BPDS1,JSON2,BPDS2 backend
```

## Reading notes

- **icmhs is again the only declared dependency.** Same caveat as 527EZ.
- **The "Today's submission path (legacy)" lane** is the slow path — PDF rendering and OCR add latency and OCR error rates. POI (Pension Optimization Initiative) is funding the move to direct BPDS.
- **The "#121603 future path" lane is dotted** — not yet wired in production. #121603 has no assignee on the epic; ownership unclear.
- **The orange `Conditional pages silently change / FOOTGUN` node** is the sessionStorage foot-gun. Tests that mock `burialPdfFormAlignment` via Redux only will not match runtime behavior; if anything clears `sessionStorage` mid-flow (e.g., navigating across tabs, plugin behavior), the toggle effectively flips for that user.
- **`version: 3 → 4` migration** at `config/form.js:100` ships in lockstep with the toggle going live. SIP records started under v3 must migrate cleanly.
