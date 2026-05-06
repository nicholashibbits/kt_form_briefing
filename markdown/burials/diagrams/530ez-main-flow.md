# 530EZ — Main Flow

Source: `src/applications/burials-ez/config/form.js`. Five chapters; many `depends:` gates driven by `view:claimedBenefits.*`, `relationshipToVeteran`, and the `showPdfFormAlignment` toggle.

`showPdfFormAlignment` is a session-storage check (`utils/helpers.jsx:166`) populated from the `burialPdfFormAlignment` Redux feature toggle in `BurialsApp.jsx:46-56` — i.e. the toggle is mirrored into sessionStorage so synchronous `depends:` functions can read it. **The toggle is absent from `featureFlagNames.json`.** (TBD: confirm with team.)

```mermaid
flowchart TB
  Note["Form-wide pending change: #114166<br/>OMB version alignment — milestone 2026-05-31<br/>toggle: burialPdfFormAlignment (NOT in registry)<br/>version: 3 → 4 migration TODO at config/form.js:100"]:::inProgress
  Intro[IntroductionPage]:::frontend

  subgraph C1["1. Claimant information"]
    direction LR
    C1a[relationship-to-veteran] --> C1b[personal-information] --> C1c[mailing-address] --> C1d[contact-information]
  end

  subgraph C2["2. Deceased Veteran information"]
    direction LR
    C2a[veteran-information] --> C2b[burial dates] --> C2c[location-of-death] --> C2d["home-hospice-care<br/>depends locationOfDeath + flag"] --> C2e["home-hospice-care-after-discharge<br/>depends hospice answer"] --> C2f["nursing-home variant<br/>unpaid/paid/vaMedical/state"]
  end

  subgraph C3["3. Military history"]
    direction LR
    C3a[separation-documents] --> C3b["upload DD214<br/>depends hasDD214"] --> C3c["service-number<br/>depends no DD214"] --> C3d["service-period vs service-periods<br/>SPLIT by showPdfFormAlignment"]:::toggle --> C3e[previous-names + add — conditional] --> C3f[POW pages]
  end

  subgraph C4["4. Benefits selection"]
    direction LR
    C4a[benefits/selection] --> C4b["burialAllowance pages<br/>depends benefit"] --> C4c["burialAllowance statement-of-truth<br/>depends funeralDirector + unclaimed"] --> C4d["finalRestingPlace + cemetery<br/>depends plotAllowance benefit"] --> C4e[plotAllowance pages] --> C4f[transportationExpenses]
  end

  subgraph C5["5. Additional information"]
    direction LR
    C5a[directDeposit] --> C5b[supportingDocuments] --> C5c["deathCertificate vs deathCertificateRequired<br/>SPLIT by relationship"] --> C5d["transportationReceipts<br/>depends transportation chosen"] --> C5e[additionalEvidence] --> C5f["fasterClaimProcessing<br/>HIDDEN when showPdfFormAlignment"]:::toggle
  end

  Note -.applies to entire form.-> Intro
  Intro --> C1 --> C2 --> C3 --> C4 --> C5
  C5 --> Review["Review & Submit<br/>#121603 BPDS direct integration in-progress"]:::inProgress
  Review --> Confirmation[Confirmation]:::frontend

  classDef frontend fill:#ffffff,stroke:#333333,stroke-width:1px;
  classDef backend fill:#dbeafe,stroke:#1e40af,stroke-width:1px,color:#1e3a8a;
  classDef toggle fill:#e9d5ff,stroke:#7e22ce,stroke-width:1px,stroke-dasharray:4 2,color:#581c87;
  classDef inProgress fill:#fef3c7,stroke:#d97706,stroke-width:3px,color:#78350f;

  class C1,C2,C3,C4,C5 frontend
```

## Reading notes

- **`Note` (yellow-orange)** is the form-wide pending change for #114166 plus the lurking `version: 3 → 4` SIP migration foot-gun at `config/form.js:100`.
- **`C3d` and `C5f` are purple-dotted** — both are direct `showPdfFormAlignment` forks. C3d picks between `service-period` and `service-periods`; C5f hides `fasterClaimProcessing` entirely when the toggle is on.
- **`C5c` is the relationship-driven split** — funeral director sees `deathCertificateRequired`; everyone else sees `deathCertificate`. Easy to break by editing one and forgetting the other.
- **`Review` is yellow-orange** because #121603 (BPDS integration) will eventually rewire the submission path; today it still goes JSON → PDF → CentralMail → OCR → BPDS.
