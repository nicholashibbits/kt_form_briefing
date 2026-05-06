# 674 — Main Flow

Source: `src/applications/dependents/686c-674/config/chapters/formConfig674.js` and `config/chapters/674/addStudentsArrayPages.js`. Built with `arrayBuilderPages` — Veteran can add multiple students. The chapter only renders when `view:addOrRemoveDependents.add` is true and the user picked the "add student" task.

For failure modes, see [686c-offramp.md](686c-offramp.md) — 674 inherits 686c's submission and downtime story.

```mermaid
flowchart TB
  Note["Form-wide pending change: #120876 reopened<br/>Q4 2026 PDF redesign — schema regen, deprecated field removal, new fields"]:::reopened
  Entry["Entered from 686 task wizard<br/>'Add a child 18-23 attending school'"]:::frontend

  subgraph Students["Add students 18-23 — array builder"]
    direction TB
    Intro[Intro page]:::frontend
    Summary[Students summary]:::frontend

    subgraph Row1["For each student — part 1"]
      direction LR
      Identity["Identity & relationship<br/>vaDependentsNoSsn hideIf #124457"]:::toggle
      Personal[Personal info & marital]:::frontend
      Address[Address]:::frontend
      Education[Education benefits]:::frontend
      Federal[Federally funded program?]:::frontend
      Program[Program info & start date]:::frontend
      Attendance[Attendance & accreditation]:::frontend
      Identity --> Personal --> Address --> Education --> Federal --> Program --> Attendance
    end

    subgraph Row2["For each student — part 2"]
      direction LR
      Terms[Term dates + previous term]:::frontend
      Pension[Claims/receives pension?]:::backend
      Income[Earnings, future earnings, assets]:::frontend
      Stopped[Stopped attending date — conditional]:::frontend
      Marriage[Marriage date — conditional]:::frontend
      Remarks[Remarks]:::frontend
      Terms --> Pension --> Income --> Stopped --> Marriage --> Remarks
    end

    Intro --> Summary
    Summary --> Identity
    Attendance --> Terms
    Remarks -.loop next student.-> Summary
  end

  Note -.applies to entire form.-> Students
  Entry --> Intro
  Summary --> Done["Returns to 686 main flow"]:::frontend

  classDef frontend fill:#ffffff,stroke:#333333,stroke-width:1px;
  classDef backend fill:#dbeafe,stroke:#1e40af,stroke-width:1px,color:#1e3a8a;
  classDef toggle fill:#e9d5ff,stroke:#7e22ce,stroke-width:1px,stroke-dasharray:4 2,color:#581c87;
  classDef inProgress fill:#fef3c7,stroke:#d97706,stroke-width:3px,color:#78350f;
  classDef reopened fill:#fed7aa,stroke:#ea580c,stroke-width:4px,color:#7c2d12;
```

## Reading notes

- **`Note` (orange-bordered)** flags the form-wide reopened scope: #120876 PDF redesign will churn schemas, prefill transformers, and probably page modules.
- **`Identity`** is purple-dotted because of the `vaDependentsNoSsn` hideIf gate (`config/chapters/674/studentIdentityPages.js:49`).
- **`Pension`** is blue because the "claims/receives pension" answer participates in the pension-API fork that flows through the 686.
- **The loop edge from `Remarks` back to `Summary`** is the array-builder pattern — Veteran can add another student or move on.
