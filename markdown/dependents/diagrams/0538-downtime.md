# 0538 — Downtime & Off-ramps

The 0538 has a **suspicious** downtime story: `downtime.dependencies` is fully commented out at `config/form.js:55-64`. This is a prefill-required form that depends on BGS for dependent records, but the platform's downtime detection doesn't know about it.

```mermaid
flowchart TB
  Render[Form attempts to render]:::frontend
  Render --> Downtime{"downtime.dependencies<br/>COMMENTED OUT<br/>(config/form.js:55-64)"}:::offramp
  Downtime -->|"BGS up — works as intended"| Prefill[Prefill from BGS]:::backend
  Downtime -->|"BGS down — no platform alert"| ConfusingError["Veteran sees generic error<br/>or partial form<br/>FOOTGUN"]:::downtime

  Prefill --> Veteran{Veteran reviews dependents}:::frontend

  Veteran -->|"hasDependentsStatusChanged = 'N'"| Submit[Submit 0538]:::frontend --> Confirmation[Confirmation]:::frontend
  Veteran -->|"hasDependentsStatusChanged = 'Y'"| ExitForm["ExitForm component<br/>redirects to 686C-674"]:::offramp
  ExitForm --> NoAuditTrail["No 0538 backend record<br/>that change was reported"]:::offramp

  classDef downtime fill:#fecaca,stroke:#dc2626,stroke-width:2px,color:#7f1d1d;
  classDef offramp fill:#fed7aa,stroke:#ea580c,stroke-width:2px,color:#7c2d12;
  classDef recovery fill:#bbf7d0,stroke:#16a34a,stroke-width:1px,color:#14532d;
  classDef frontend fill:#ffffff,stroke:#333333,stroke-width:1px;
  classDef backend fill:#dbeafe,stroke:#1e40af,stroke-width:1px,color:#1e3a8a;
```

## Reading notes

- **The orange `downtime.dependencies COMMENTED OUT` decision node** is the most important thing to fix or document. The original commented block included BGS, MVI, VA Profile, VBMS — re-enable with the team's blessing if no specific reason exists for the disable.
- **The red `Veteran sees generic error / FOOTGUN` node** is what Veterans see today during a real BGS outage: no platform downtime alert, just whatever the prefill request happens to throw.
- **The orange `ExitForm component — redirects to 686C-674` node** is the silent off-ramp. Veterans intend to update dependents, hit "yes things changed," and end up on a new form. The 0538 has no record they ever started — not great for analytics or follow-up.
- **The `No 0538 backend record` callout** is connected to the warning-alert / 8-year-diary-date issue documented in the view-dependents README — there's no API for the BGS diary date, so the alert is time-based rather than intelligent.
