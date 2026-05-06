# 686c — Off-ramps & Downtime

Failure modes for the 686. Three classes:

1. **Pre-load downtime** — external systems are down before the form even renders.
2. **In-flow off-ramps** — the form renders but Veteran has to leave (e.g., adopted child branch).
3. **Submission off-ramps to RBPS manual processing** — form submits successfully, but RBPS rejects the case to a human adjudicator.

```mermaid
flowchart TB
  subgraph Preload["Pre-load — downtime gates (config/form.js:109-118)"]
    direction LR
    BGS[(BGS)]:::downtime
    Profile[(VA Profile)]:::downtime
    MVI[(MVI)]:::downtime
    VBMS[(VBMS)]:::downtime
    Pension[(Pension API)]:::downtime
  end

  Preload -->|any down| Block["Form blocks rendering<br/>downtime alert shown"]:::downtime
  Preload -->|all up| Render[Form renders normally]:::recovery
  Render -->|"BGS returns no VA file #"| FileBlock["Pre-form alert: no VA file #<br/>Veteran cannot submit"]:::downtime
  Render -->|"Pension API fails"| BackupPath["showPensionBackupPath() = true<br/>manual check-veteran-pension page"]:::recovery

  subgraph InFlow["In-flow off-ramps"]
    direction LR
    AdoptedExit["Picklist: child adopted<br/>EXIT — different form"]:::offramp
    ParentOtherExit["Picklist: parent reason 'Other'<br/>EXIT — different form"]:::offramp
  end

  subgraph RBPS["Post-submit — RBPS off-ramps to manual processing"]
    direction TB
    RBPS_Add["Dependent Additions #113855 reopened<br/>• duplicate dependents (~7,400 FY25 — done)<br/>• 674 for non-benefits dep (~2k → #130754)<br/>• spouse add without remove (~750 → #112364)"]:::offramp
    RBPS_Other["Other Causes #124480 in-progress<br/>• suspended/terminated awards<br/>• earlier effective dates<br/>• RAD-date conflicts<br/>• event-date < 18th birthday<br/>• military retired pay withholding"]:::offramp
  end

  Render --> InFlow
  Render --> RBPS

  classDef downtime fill:#fecaca,stroke:#dc2626,stroke-width:2px,color:#7f1d1d;
  classDef offramp fill:#fed7aa,stroke:#ea580c,stroke-width:2px,color:#7c2d12;
  classDef recovery fill:#bbf7d0,stroke:#16a34a,stroke-width:1px,color:#14532d;
```

## Reading notes

- **Red = full downtime.** Form won't render, Veteran sees the platform downtime alert.
- **Orange = off-ramp.** Form does work, but the Veteran's case gets routed away from automated processing.
- **Green = recovery path.** The form has a graceful fallback (e.g., manual pension question when the API is down).
- **RBPS sits behind BGS.** A BGS outage looks identical to "rules-based processing rejected your case" from the Veteran's perspective; ops should distinguish.
- **The `#113855` and `#124480` boxes** are intentionally listed as a checklist — those are the scenarios the team is choosing whether to pre-emptively block in VA.gov vs allow-and-let-RBPS-reject.
