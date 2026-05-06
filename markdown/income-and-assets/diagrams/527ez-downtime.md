# 527EZ — Downtime & Off-ramps

Pension's external surface is unusually thin. The downtime declaration in `config/form.js:36` lists only `externalServices.icmhs` (TBD: confirm with team — 527EZ has no BGS/MVI prefill dependency declared, which is suspicious for a benefits form).

```mermaid
flowchart TB
  subgraph Preload["Pre-load — declared downtime gates"]
    icmhs[(icmhs)]:::downtime
  end

  Preload -->|down| Block["Form blocks rendering<br/>(only icmhs declared)"]:::downtime
  Preload -->|up| Render["Form renders normally"]:::recovery

  Render --> RatingAPI[/"GET /v0/rated_disabilities<br/>DisabilityRatingAlert"/]:::backend
  RatingAPI -->|"100% rating"| Alert["Informational alert<br/>do NOT block submit (VBA directive)"]:::recovery
  RatingAPI -->|"API error"| LoggedOnly["Datadog logged<br/>no user-facing alert"]:::recovery
  RatingAPI -->|"<100% rating"| NoAlert["No alert"]:::recovery

  Render --> Submit["Veteran submits<br/>(no in-flow off-ramps declared)"]:::frontend
  Submit --> Disposition[/"VBA disposition — outside FE scope"/]:::backend

  classDef downtime fill:#fecaca,stroke:#dc2626,stroke-width:2px,color:#7f1d1d;
  classDef offramp fill:#fed7aa,stroke:#ea580c,stroke-width:2px,color:#7c2d12;
  classDef recovery fill:#bbf7d0,stroke:#16a34a,stroke-width:1px,color:#14532d;
  classDef backend fill:#dbeafe,stroke:#1e40af,stroke-width:1px,color:#1e3a8a;
  classDef frontend fill:#ffffff,stroke:#333333,stroke-width:1px;
```

## Reading notes

- **icmhs is the only declared dependency.** If pension actually depends on BGS/MVI for prefill, the downtime list is stale and Veterans hitting an outage will see confusing errors. Open question for the team.
- **The disability-rating alert is the only conditional UI**, and it's never a blocker — VBA directive in #121731. Logging-only mode is via `pensionRatingAlertLoggingEnabled`.
- **No 527EZ-specific in-flow off-ramps.** This is unlike 686c, which has multiple pre-emptive RBPS-rejection branches.
