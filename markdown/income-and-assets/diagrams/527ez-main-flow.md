# 527EZ — Main Flow

Source: `src/applications/pensions/config/form.js` plus six chapter folders under `config/chapters/`. Page-level detail in chapters 3-5 is large (~25 pages each); collapsed to chapter-level here. See `config/chapters/` for the full per-page tree.

```mermaid
flowchart TB
  Note["Form-wide pending change: #102564<br/>OMB version alignment — field-by-field audit<br/>toggle: pensionPdfFormAlignment"]:::inProgress
  Intro["IntroductionPage<br/>renders DisabilityRatingAlert (#121731 closed)<br/>toggle: pensionRatingAlertLoggingEnabled"]:::toggle

  subgraph Ch1["1. Applicant information"]
    direction LR
    A1[applicantInformation] --> A2[mailingAddress] --> A3[contactInformation]
  end

  subgraph Ch2["2. Military history"]
    direction LR
    B1[servicePeriod] --> B2["hasOtherNames + otherNames pages"] --> B3[POW status]
  end

  subgraph Ch3["3. Health & employment"]
    direction LR
    C1[Over 65 / Soc Sec supplemental] --> C2[Medical / nursing / Medicaid] --> C3[Special Monthly Pension] --> C4[VA & federal treatment history] --> C5[Employment history] --> C6[Additional evidence]
  end

  subgraph Ch4["4. Household information"]
    direction LR
    D1[Marital status / marriage info] --> D2[Marriage & spouse-marriage history] --> D3[Spouse address & monthly support] --> D4[Dependent children + per-child pages]
  end

  subgraph Ch5["5. Financial information"]
    direction LR
    E1[Net worth / transferred assets] --> E2[Home ownership & acreage] --> E3[Income sources array] --> E4[Care expenses array] --> E5[Medical expenses array]
  end

  subgraph Ch6["6. Additional information"]
    direction LR
    F1[Direct deposit + account info] --> F2[Other payment options] --> F3[Supporting documents — A&A] --> F4[Upload documents] --> F5[Faster claim processing]
  end

  Note -.applies to entire form.-> Intro
  Intro --> Ch1 --> Ch2 --> Ch3 --> Ch4 --> Ch5 --> Ch6
  Ch6 --> Review[Review & Submit]:::frontend --> Confirmation[Confirmation]:::frontend

  classDef frontend fill:#ffffff,stroke:#333333,stroke-width:1px;
  classDef backend fill:#dbeafe,stroke:#1e40af,stroke-width:1px,color:#1e3a8a;
  classDef toggle fill:#e9d5ff,stroke:#7e22ce,stroke-width:1px,stroke-dasharray:4 2,color:#581c87;
  classDef inProgress fill:#fef3c7,stroke:#d97706,stroke-width:3px,color:#78350f;

  class Ch1,Ch2,Ch3,Ch4,Ch5,Ch6 frontend
```

## Reading notes

- **The "Form-wide pending change: #102564" banner** signals that #102564 (OMB version alignment) touches every chapter — `pensionPdfFormAlignment` gates content everywhere.
- **`IntroductionPage`** is purple because the disability-rating alert (#121731) is gated behind `pensionRatingAlertLoggingEnabled`. Per VBA, the alert is informational — never block submission.
- **Chapter 5 sub-flows are heavy with array builders** (income, care, medical). Per-row schema changes need ID stability across save-in-progress.
- **0969 is a required supplemental** for some 527EZ applicants — content changes here may need parallel edits in `src/applications/income-and-asset-statement/`.
