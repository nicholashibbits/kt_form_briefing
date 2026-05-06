# Knowledge Transfer — Form Briefing

General overview to guide initial exploration in `vets-website` relevant to work assigned to MMS. It is generated with AI. It is NOT a definitive guide.

## Table of Contents

1. [How to read this document](#how-to-read-this-document)
2. [VAF21-686c — Application Request to Add and/or Remove Dependents](#vaf21-686c--application-request-to-add-andor-remove-dependents)
   - [How 686 and 674 are connected](#how-686-and-674-are-connected)
   - [Picklist (v2 vs v3) — 686c-674](#picklist-v2-vs-v3--686c-674)
3. [VAF21-674 — Request for Approval of School Attendance](#vaf21-674--request-for-approval-of-school-attendance)
4. [VAF21P-527EZ — Pension](#vaf21p-527ez--pension)
5. [VAF21P-530EZ — Burial Benefits](#vaf21p-530ez--burial-benefits)
6. [VAF21P-0969 — Income and Asset Statement](#vaf21p-0969--income-and-asset-statement)
7. [VAF21-0538 — Status of Dependents (Dependents Verification)](#vaf21-0538--status-of-dependents-dependents-verification)
8. [Cross-Cutting Initiatives](#cross-cutting-initiatives)
9. [Glossary](#glossary)

---

## How to read this document

### Per-form template
Every form section follows the same layout so you can scan quickly. Each numbered form section opens with a **Primary directory** bullet pointing at the app's root in `vets-website`, then the rest of the layout below:

1. **Issue index** — every open/closed epic affecting the form, each linked. Primary directory is shown as a bullet right under the form heading, before this table.
2. **Diagrams** — links to standalone `.md` files in the program-area `diagrams/` folder (current-state flowchart and, where applicable, off-ramp / downtime flowchart)
3. **Feature toggles in scope** — every toggle that gates code in this app, with registry presence and call sites
4. **In-flight work** — tables grouped by issue, status, and code surface
5. **Gotchas, factors, and watch-outs** — the things that will burn you
6. **Open questions for the old team** — moved to [kt-questions.md](kt-questions.md). One section per form for the takeover meeting; items the in-app README already answers are removed or annotated `(README — answered: …)`.

For per-page / per-chapter source-file mappings, refer to the form's primary directory in `vets-website` and the original README in the appropriate `original-readmes/` subfolder.

### Issue index ordering convention

Open / in-progress / reopened issues are listed first and have their **Status** column bolded. A separator row separates them from closed-but-still-relevant issues at the bottom of the table.

### Diagram legend

Diagrams are split into individual files under each program area's `diagrams/` folder (e.g., `dependents/diagrams/`, `income-and-assets/diagrams/`, `burials/diagrams/`) so you can iterate on one without scrolling past the rest. Every flowchart uses the same color and edge conventions:

| Node style | Meaning |
|---|---|
| White fill, thin black border | Frontend-only logic |
| Light blue fill | Backend-driven (BGS, BPDS, pension API, MVI, VBMS, CentralMail, Forms API) — flow depends on an external system |
| Light purple fill, dotted border | Feature-toggle-gated — flow only fires when a Flipper toggle is on |
| Yellow fill, thick orange border | In-progress (open issue, code on `main`) — issue badge is in the node label |
| Light orange fill, very thick orange border | Reopened issue (was closed, scope or regression brought it back) |
| Light green fill, dashed border | Built but the toggle is not yet fully on in production |

Edges are labeled with the gate that controls the transition (e.g., `showV3Picklist() && hasAwardedDependents`) wherever the next page is conditional. Off-ramp diagrams use a separate red/amber palette for downtime and rejection states; the legend is repeated at the top of each off-ramp file.

### Issue linking convention

Every GitHub issue is linked the **first time it appears in a form's section** and **every time it appears inside a table cell**. Subsequent in-prose mentions in the same section stay as plain `#119718` text to avoid visual noise.

### File-reference convention

`file/path.js:42-58` — clickable in most IDEs. Line numbers were captured on `main` as of 2026-04-30; if you find drift, the file path is still authoritative.

### Knowledge gaps

Items I could not confirm without team input are flagged inline as **(TBD: confirm with team)** rather than buried at the end. Two known gaps as of writing:

- `dependentsPensionCheck` toggle is read by the 686c app but absent from `featureFlagNames.json` — could be passthrough convention or a registry oversight.
- `burialPdfFormAlignment` has the same shape — read by code, missing from the registry, and additionally mirrored into `sessionStorage` for sync `depends:` access.

---

# VAF21-686c — Application Request to Add and/or Remove Dependents

- **Primary directory:** `src/applications/dependents/686c-674/` ([original README](dependents/original-readmes/686c-674.md))

The 686c (Add/Remove Dependents) and 674 (School Attendance) live in the **same** vets-website application. The 674 is a workflow inside the 686c form. One POST submits both; on the backend two separate Sidekiq jobs run, with 674 queued conditionally on 686 success — see [How 686 and 674 are connected](#how-686-and-674-are-connected) below.

## Issue index

| Issue | Title | Status |
|---|---|---|
| [#119718](https://github.com/department-of-veterans-affairs/va.gov-team/issues/119718) | Integrate 686c submission with Forms API (FDF Pilot) | **In-progress (96% — 49/51). Milestone 2026-04-27 already past.** |
| [#114340](https://github.com/department-of-veterans-affairs/va.gov-team/issues/114340) | 21-686c and 21-674 Separation, Submission Orchestration, Modularization (Engines) | **In-progress (89% — 66/74)** |
| [#124480](https://github.com/department-of-veterans-affairs/va.gov-team/issues/124480) | Reduce 686 RBPS off-ramps to manual processing (Other Causes) | **In-progress (71% — 5/7)** |
| [#113855](https://github.com/department-of-veterans-affairs/va.gov-team/issues/113855) | Reduce 686 RBPS off-ramps to manual processing (Dependent Additions) | **Reopened — In-progress** |
| [#108392](https://github.com/department-of-veterans-affairs/va.gov-team/issues/108392) | Remove dependents during DV process (Phase 2) | **Open (no registered work)** |
| [#124457](https://github.com/department-of-veterans-affairs/va.gov-team/issues/124457) | 686/674 "Add dependent" flow improvements | **In-progress (83% — 35/42)** |
| — | *— closed below —* | |
| [#112364](https://github.com/department-of-veterans-affairs/va.gov-team/issues/112364) | Simplify previous marriage section in 686 | Closed (Nov 2025) |
| [#120229](https://github.com/department-of-veterans-affairs/va.gov-team/issues/120229) | 686c Removal Picklist MVP Implementation | Closed (Apr 2026) |
| [#113850](https://github.com/department-of-veterans-affairs/va.gov-team/issues/113850) | Integrate pension income questions into 686 | Closed (Mar 2026) |

## Directory detail

- `src/applications/dependents/686c-674/` — single, co-mingled application. See the in-app README ([local copy](dependents/original-readmes/686c-674.md)) for the form-system overview, valid-VA-file-number check, and toggle map.
- `src/applications/dependents/686c-674/config/chapters/674/` — 674 sub-chapters; the only frontend evidence of the modular split called for in #114340. The "Engines" work is primarily a vets-api Rails-engine effort; the frontend side is the chapter-level isolation already started here.
- `src/applications/dependents/shared/` — actions/components/reducers shared across 686/674 and view-dependents/0538.

No legacy/deprecated co-resident copy on `main`.

## Diagrams

- **Main flow:** [dependents/diagrams/686c-main-flow.md](dependents/diagrams/686c-main-flow.md)
- **Off-ramp / downtime:** [dependents/diagrams/686c-offramp.md](dependents/diagrams/686c-offramp.md)

## How 686 and 674 are connected

**They are one app on the frontend, one POST on submission, two Sidekiq jobs in the backend. There is no redirect between them.**

### Code-level coupling

The 674 is a chapter (`report674`) inside the same vets-website application. The user opts in to it from the 686 task wizard by checking *"Add a child 18 to 23 years old who'll be attending school"*. From the in-app README:

> "You will notice that the last workflow is technically a totally separate form (the form 21-674). The stakeholders wanted this form added as a workflow to this form since it deals with making changes to a dependent. That workflow actually submits using a separate job than the 686c does." — `dependents/original-readmes/686c-674.md`

Every page in `config/chapters/formConfig674.js` (lines 41–267) is gated by:

```js
depends: formData =>
  isChapterFieldRequired(TASK_KEYS.report674, formData) && isAddingDependents(formData)
```

Same `isChapterFieldRequired` helper drives the add-spouse, add-child, and remove flows — they all consult `formData["view:selectable686Options"]` to decide whether their chapter renders. So at the chapter level, the 674 is just one of several optional task chapters; it is not architecturally privileged.

### Single submission, multiple async jobs

Both forms POST together to `/v0/dependents_applications` (in vets-api: `dependents_applications_controller.rb#create`). On accept, vets-api:

1. Builds a `DependencyClaim`.
2. **Always** kicks off a PDF→VBMS job (PDFtk renders the form to a PDF, uploads to the Veteran's eFolder).
3. Conditionally enqueues `BGS::SubmitForm686cJob` and/or `BGS::SubmitForm674Job` based on which workflows the payload contains.

If the synchronous step fails (validation, PDF gen) the entire claim **takes a Lighthouse backup path** — form + attachments are generated and uploaded into Lighthouse for evaluation. (Per `dependents/original-readmes/686c-674.md`.)

### Async ordering rule (the load-bearing detail)

From the in-app docs ([686c async copy](dependents/original-readmes/686c-async-vetsapi-doc.md), [674 async copy](dependents/original-readmes/674-async-vetsapi-doc.md)):

- `SubmitForm686cJob` runs first.
- **If 686 succeeds**, it queues `SubmitForm674Job`, deletes the in-progress form, and sends `VANotify::ConfirmationEmail`.
- **If 686 fails** — i.e., raises `Invalid686cClaim` (blank SSN / dependent app / veteran address) or hits a `BGS_686c_SERVICE_403` from `BGS::Service#create_note` — the in-progress form is restored and `DependentsApplicationFailureMailer` notifies the user. **The 674 job never runs.**
- The 674 job, when it runs, follows the same shape: success deletes SIP + sends a confirmation email; failure restores SIP + sends the failure mailer. There is no compensating action that resubmits the 686 if the 674 fails after the 686 succeeded.

This asymmetry is the bug-hunting starting point: a Veteran who selected both flows can have their 686 land in BGS while the 674 silently never queued (or queued and failed independently). When debugging missing-674 reports, **always start with the 686 BGS job log first, then the 674 log second.**

When the Engines work [#114340](https://github.com/department-of-veterans-affairs/va.gov-team/issues/114340) lands on vets-api, this ordering may change (the open question below tracks it).

### Where a redirect *does* exist

The only real redirect in this portfolio is `0538 → 686`. On the Dependents Verification form, when the Veteran answers "Yes, my dependents have changed", `dependents-verification/components/ExitForm.jsx` navigates them to `/manage-dependents/add-remove-form-21-686c-674/introduction`. The 0538 backend has no record that the Veteran intended to update — only the destination 686 submission would (if they finish it). See the [0538 section](#vaf21-0538--status-of-dependents-dependents-verification).

## Feature toggles in scope

| Toggle | Registry presence | Where it's read |
|---|---|---|
| `vaDependentsV3` (`va_dependents_v3`) | `featureFlagNames.json:348` | `config/utilities/formHelpers.js:71-72` (drives `showV3Picklist` / `noV3Picklist`) |
| `vaDependentsNoSsn` | `featureFlagNames.json:352` | `config/chapters/674/studentIdentityPages.js:49`; `config/chapters/report-add-a-spouse/spouse-information/spouseInformation.js:65`; `config/chapters/report-add-child/information.js:45` |
| `dependentsPensionCheck` | **Absent** (TBD: confirm with team) | gates `showPensionRelatedQuestions` and `showPensionBackupPath` in `config/utilities/api.js:55-87` |
| `vaDependentsDuplicateModals` | `featureFlagNames.json:351` | `config/utilities/formHelpers.js:163-164` (`showDupeModalIfEnabled`) |
| `dependentsEnableFormViewerMfe` | `featureFlagNames.json:84` | `containers/ConfirmationPage.jsx:94` (FDF viewer link) — owned by **FDF team**, not this team |
| `vaDependentsBrowserMonitoringEnabled` | `featureFlagNames.json:350` | Datadog RUM init |

## In-flight work

### [#119718](https://github.com/department-of-veterans-affairs/va.gov-team/issues/119718) — Forms API / FDF Pilot
First end-to-end Fully Digital Forms use case: vets-api submits structured JSON to the new Forms API while still hitting BGS/RBPS and uploading to eFolder via Claim Evidence API. Pattern reused from BPDS work. Milestone 2026-04-27 has passed; verify go-live state with the team.

| Surface | File |
|---|---|
| FDF viewer link on confirmation | `containers/ConfirmationPage.jsx:94` |
| Submission entry | `config/form.js:64` + `submit.js` |
| Toggle | `dependentsEnableFormViewerMfe` ([#119718](https://github.com/department-of-veterans-affairs/va.gov-team/issues/119718)) |

### [#114340](https://github.com/department-of-veterans-affairs/va.gov-team/issues/114340) — Engines (Separation + Orchestration)
Split co-mingled 686/674 codebases into separate vets-api Rails engines and vets-website chapter modules, plus orchestrate multi-system submissions (BGS/VBMS/Lighthouse/Forms API) with failure recovery.

| Surface | File |
|---|---|
| Chapter modularity (FE side already started) | `config/chapters/674/`, `formConfigAdd.js`, `formConfigRemoveV2.js`, `formConfigRemovePicklist.js` |
| App-level split | **Not done.** Still one app on the FE. |
| Submission ordering | `documentation/686c.md` and `documentation/674.md` describe today's `SubmitForm686cJob` → `SubmitForm674Job` chain. |

### [#124480](https://github.com/department-of-veterans-affairs/va.gov-team/issues/124480) — RBPS off-ramps (Other Causes)
Pre-submission alerts for off-ramps caused by suspended/terminated awards, earlier effective dates, RAD-date conflicts, event-date-before-18th-birthday, and military retired pay withholding. Team must decide which to handle in VA.gov vs leave to RBPS/VBA.

Likely lives in alert components in chapters that gate on awarded/RAD/effective-date data — not yet visible on `main`. **(TBD: ask team which scenarios survived scope cuts.)**

### [#113855](https://github.com/department-of-veterans-affairs/va.gov-team/issues/113855) — RBPS off-ramps (Dependent Additions, **REOPENED**)
Three off-ramp scenarios: duplicate dependents (~7,400 FY25 — done), 674 submitted for non-benefits dependent (~2k — rolled into [#130754](https://github.com/department-of-veterans-affairs/va.gov-team/issues/130754)), spouse add without removing old (~750 — covered by [#112364](https://github.com/department-of-veterans-affairs/va.gov-team/issues/112364)). Was closed once already; find out what regression or scope expansion brought it back.

| Surface | File |
|---|---|
| Duplicate-spouse modal | `config/utilities/formHelpers.js:163-164` (`showDupeModalIfEnabled`) — toggle `vaDependentsDuplicateModals` |
| Spouse-add-without-remove | shipped under [#112364](https://github.com/department-of-veterans-affairs/va.gov-team/issues/112364) (closed Nov 2025) — `config/chapters/formConfigAdd.js` veteran-marriage-history pages |

### [#124457](https://github.com/department-of-veterans-affairs/va.gov-team/issues/124457) — Add-flow improvements
Add-flow content/copy updates, allow no-SSN dependents (today users enter dummy SSNs that break processing), clarify school-attendance section.

| Surface | File |
|---|---|
| `vaDependentsNoSsn` hideIf gates | `config/chapters/674/studentIdentityPages.js:49`; `config/chapters/report-add-a-spouse/spouse-information/spouseInformation.js:65`; `config/chapters/report-add-child/information.js:45` |

### [#108392](https://github.com/department-of-veterans-affairs/va.gov-team/issues/108392) — Remove dependents during DV process (Phase 2)
Phase 2 of the DV Tool. Originally planned via digital 0538; team pivoted to picklist in 686c first ([#120229](https://github.com/department-of-veterans-affairs/va.gov-team/issues/120229)). Whether 0538-based removal still happens is unclear. **(TBD: confirm long-term home of removal flow.)**

### Closed but worth knowing
- [#120229](https://github.com/department-of-veterans-affairs/va.gov-team/issues/120229) Removal Picklist MVP — closed Apr 2026. Lives in `config/chapters/formConfigRemovePicklist.js` and `components/picklist/*`. See the [Picklist deep-dive below](#picklist-v2-vs-v3--686c-674).
- [#113850](https://github.com/department-of-veterans-affairs/va.gov-team/issues/113850) Pension income questions — closed Mar 2026. Two conditional questions driven by `showPensionRelatedQuestions` and `showPensionBackupPath`.
- [#112364](https://github.com/department-of-veterans-affairs/va.gov-team/issues/112364) Simplify previous marriage — closed Nov 2025. Prefill prior-spouse data and alert when an active prior spouse must be removed before adding a new one.

## Gotchas, factors, and watch-outs

- **The 686 and 674 are one app on the frontend.** Releases couple them. The "Engines" effort ([#114340](https://github.com/department-of-veterans-affairs/va.gov-team/issues/114340)) is fixing this on the backend first. Frontend chapter isolation already exists at `config/chapters/674/` and per-flow `formConfig*.js` files, but there is no separate vets-website application. Don't expect a clean split before backend lands.
- **Schema is pinned to a private GitHub Enterprise commit:** `package.json:340` — `"vets-json-schema": "https://va.ghe.com/software/vets-json-schema.git#ea08303608c47dc35ada003546bb43d7b8ba794b"`. Bumping the commit is how you pull in new fields. ([#119026](https://github.com/department-of-veterans-affairs/va.gov-team/issues/119026) originally proposed swapping this for a build-time GET from an endpoint — descoped.)
- **Feature-toggle map** (read by Redux `state.featureToggles`):
  - `va_dependents_v3` (`vaDependentsV3`) — picklist removal flow.
  - `dependents_pension_check` — pension API integration that decides the conditional 686 pension questions and the backup path. **Absent from `featureFlagNames.json` (TBD).**
  - `va_dependents_no_ssn` — allows adding dependents without an SSN (#124457 deliverable).
  - `va_dependents_duplicate_modals` — duplicate-spouse warning modal.
  - `dependents_enable_form_viewer_mfe` — owned by the FDF team, not this team. Drives the JSON-rendered viewer linked from the confirmation page.
  - `va_dependents_browser_monitoring_enabled` — Datadog RUM.
- **External system surface** (from `config/form.js:109-118` downtime list): BGS, VA Profile, MVI, VBMS — any of those down means prefill blocked. RBPS sits behind BGS — that's what off-ramps to manual processing.
- **VA file number gate.** The form makes a pre-load API call and refuses to render the form unless the user has a valid VA file number (BGS requirement). New users without a file number get an alert instead of the form.
- **Reopened issue #113855.** Was closed once already — find out what regression or scope expansion brought it back.
- **#119718 milestone already passed (4/27/2026).** Ask for the actual go-live state, not the plan.
- **`useV2: true` forced in initial data** (`config/form.js:131-135`) with a comment that they're "seeing v2 submissions without it." Past data-shape bug. Be cautious when changing migrations or initial data.
- **Three picklist user states coexist** — see the [picklist deep-dive](#picklist-v2-vs-v3--686c-674) for `vaDependentV2Flow`. A v3-toggle-on user with an in-flight v2 SIP is forced down v2.

---

## Picklist (v2 vs v3) — 686c-674

The picklist is the dependent-removal UI in the 686c. It replaces the original "type the dependent's full legal name into a text box" remove flow, sourcing dependents from BGS via `state.dependents.awarded`. The user checks the dependents they want to remove; the form then loops through each checked item and asks relationship-specific follow-up questions (why removing, when, supporting facts). The design did not fit any built-in form-system pattern, so it was built as a **CustomPage with custom routing** living entirely outside the standard `arrayBuilderPages` machinery.

Source-of-truth doc inside the app: `src/applications/dependents/686c-674/components/picklist/README.md`. The "why custom code" section there explains why the built-in `followup` array pattern and `itemFilter` approaches were both rejected.

### The two versions

| Version | What it is | Toggle / gate | Where it lives |
|---|---|---|---|
| **v2** (legacy) | Five separate remove flows: report-divorce, stepchild-left-household (array), deceased-dependents (array), child-marriage-under-18 (array), child-stopped-school-18-23 (array). Each is a chapter with its own pages; user re-types dependent name. | `noV3Picklist(formData)` | `config/chapters/formConfigRemoveV2.js` (lines 57–457) |
| **v3** (picklist) | Two-chapter flow: a single picklist of awarded dependents, then a per-dependent followup chapter that branches by relationship type and reason. | `showV3Picklist(formData)` = `vaDependentsV3 && vaDependentV2Flow !== true` | `config/chapters/formConfigRemovePicklist.js` (50 lines, all delegated to `components/picklist/*`) |

### The crucial third state — `vaDependentV2Flow`

Defined at `config/utilities/formHelpers.js:71-72`. If a Veteran has a *v2 form already in progress* (save-in-progress was started before v3 launched), `vaDependentV2Flow` is set true and v2 wins even when the toggle says v3. So in production today three users coexist:

1. v3 toggle off → v2
2. v3 toggle on, no in-flight v2 SIP → v3
3. v3 toggle on, has in-flight v2 SIP → v2 (forced)

The `vaDependentV2Flow !== true` clause is what prevents in-progress submissions from breaking when the toggle flipped on. Don't remove it without a migration plan for users in state 3.

### How v3 is wired

```
config/form.js (chapters)
  ├── reviewDependents (custom intro page) — config/form.js:158-165
  ├── optionsSelection chapter
  │     └── removeDependentsPicklistOptions  ← shows the checkbox list
  │         CustomPage: components/picklist/PicklistRemoveDependents.jsx
  │         depends: showV3Picklist && hasAwardedDependents && isRemovingDependents
  └── removeDependentsPicklistFollowupPages chapter (one synthetic "page")
        └── removeDependentFollowup
            CustomPage: components/picklist/PicklistRemoveDependentFollowup.jsx
            path: 'remove-dependent'
            returnUrl: '/options-selection/remove-active-dependents'
            depends: …+ hasSelectedPicklistItems
```

The single `removeDependentFollowup` page is a **router**, not a real page. It mounts `PicklistRemoveDependentFollowup`, which reads from `components/picklist/routes.js`. That file maps each dependent type → an ordered array of sub-page modules:

- **Spouse:** `spouseReasonToRemove` → `spouseDeath` OR `spouseMarriageEnded`
- **Child:** `childIsStepchild` → `childReasonToRemove` → one of `childDeath`, `childMarriage`, `childLeftSchool`, `stepchildFinancialSupport` (→ `stepchildCurrentAddress` → `stepchildLivesWith` → `stepchildLeftHousehold`), or `childAdoptedExit` (exits form)
- **Parent:** `parentReasonToRemove` → `parentDeath` OR `parentOtherExit`

Each sub-page is a JSX module exporting a `Component`, an `onSubmit` handler, and a `goForward(itemData, index, fullData)` returning either the next path or `'DONE'`. The custom router walks selected dependents one-at-a-time; when a dependent's `goForward` returns `'DONE'`, the router advances to the next selected dependent and replays the relationship-specific subflow until all selected items are exhausted, then control returns to the form-system flow.

The in-app picklist README is mirrored at [dependents/original-readmes/picklist.md](dependents/original-readmes/picklist.md), including its embedded mermaid sub-flow diagram.

### Picklist data shape

`formData[PICKLIST_DATA]` (the constant `picklistData`) is an array of objects with at least `{ id, relationshipToVeteran, selected, ...prefillFields }`. Helpers in `formHelpers.js` read it:

| Helper | Line | Purpose |
|---|---|---|
| `hasAwardedDependents` | 194 | Gates picklist visibility |
| `hasSelectedPicklistItems` | 234 | Gates followup chapter |
| `isVisiblePicklistPage(formData, relationship)` | 217 | Per-relationship visibility (kept for future per-relationship pages) |
| `onFormLoaded` | 258 | Date hygiene for `endDate` on resume from SIP |

### Open issues against the picklist

- [#108392](https://github.com/department-of-veterans-affairs/va.gov-team/issues/108392) Phase 2 DV is the next picklist deliverable in flight. Whether it extends the picklist (continue building in 686c) or revives the digital 0538 is the unresolved question.
- [#120229](https://github.com/department-of-veterans-affairs/va.gov-team/issues/120229) MVP is closed (Apr 2026), so the picklist itself is in production behind `vaDependentsV3`. Confirm rollout %.
- The in-app README at `components/picklist/README.md` lists one known unresolved bug: a selected dependent missing DOB or relationship will be redirected back to the picklist page instead of into the followup. Worth noting if Veterans complain about silent redirects.

---

# VAF21-674 — Request for Approval of School Attendance

- **Primary directory:** `src/applications/dependents/686c-674/config/chapters/674/` (sub-form inside the 686c-674 app — there is **no separate `21-674/` application directory**). Original 686/674 README mirrored at [dependents/original-readmes/686c-674.md](dependents/original-readmes/686c-674.md).

## Issue index

| Issue | Title | Status |
|---|---|---|
| [#120876](https://github.com/department-of-veterans-affairs/va.gov-team/issues/120876) | 674 Q4 2026 PDF Updates | **In-progress, state_reason: reopened (80% — 4/5)** |
| [#114340](https://github.com/department-of-veterans-affairs/va.gov-team/issues/114340) | 686/674 Separation (Engines) | **In-progress** (cross-listed; see 686c) |
| [#113855](https://github.com/department-of-veterans-affairs/va.gov-team/issues/113855) | RBPS off-ramps — Dependent Additions | **Reopened — In-progress** (cross-listed; 674-without-686 case rolled into [#130754](https://github.com/department-of-veterans-affairs/va.gov-team/issues/130754)) |
| [#124457](https://github.com/department-of-veterans-affairs/va.gov-team/issues/124457) | 686/674 Add-flow improvements | **In-progress** (cross-listed) |
| [#124480](https://github.com/department-of-veterans-affairs/va.gov-team/issues/124480) | RBPS off-ramps — Other Causes | **In-progress** (cross-listed) |

(No closed-but-relevant 674 issues; all 674 work that is closed lives under the 686c index.)

## Directory detail

- `src/applications/dependents/686c-674/config/chapters/674/` — 674 sub-form lives inside the 686c-674 app.
- Entry from the 686 task wizard via the `report674` chapter export from `config/chapters/formConfig674.js`.
- Backend submission is `BGS::SubmitForm674Job` queued **after** `SubmitForm686cJob` succeeds — see [How 686 and 674 are connected](#how-686-and-674-are-connected) and [dependents/original-readmes/674-async-vetsapi-doc.md](dependents/original-readmes/674-async-vetsapi-doc.md).

## Diagrams

- **Main flow:** [dependents/diagrams/674-main-flow.md](dependents/diagrams/674-main-flow.md)

(674 inherits 686c's submission and downtime story; no separate off-ramp diagram. See [dependents/diagrams/686c-offramp.md](dependents/diagrams/686c-offramp.md) for failure modes.)

## Feature toggles in scope

| Toggle | Registry presence | Where it's read |
|---|---|---|
| `vaDependentsNoSsn` | `featureFlagNames.json:352` | `config/chapters/674/studentIdentityPages.js:49` |

(All other 686c-674 toggles apply transitively; see the [686c table](#feature-toggles-in-scope).)

## In-flight work

### [#120876](https://github.com/department-of-veterans-affairs/va.gov-team/issues/120876) — 674 Q4 2026 PDF Updates (Reopened)

Pension and Fiduciary Services revising the 21-674 PDF (final form due Q4 2026; current PDF expires 2027-11-30). VA.gov must regenerate, drop deprecated fields, add new ones. IPT led by Kayce White, kicked off 2025-09-30. Reopened state suggests scope shifted at least once.

| Surface | File |
|---|---|
| Schema pin | `package.json:340` (`vets-json-schema`) — bump commit hash to pull new fields |
| 674 schema import | `vets-json-schema/dist/686C-674-V2-schema.json` (loaded by 686c-674 app) |
| Page modules likely to churn | `config/chapters/formConfig674.js` + `config/chapters/674/*.js` |

## Gotchas, factors, and watch-outs

- **The 674 has no app of its own,** but does have its own JSON schema (loaded from `vets-json-schema/dist/686C-674-V2-schema.json`). Schema changes require bumping the package pin in `package.json:340`.
- **Async submission ordering** (`documentation/674.md`): if 686 succeeds, then 674 is queued; if 686 fails, 674 never runs. Bug hunting submissions means starting with the 686 BGS job log.
- **Q4 2026 PDF redesign** is the dominant near-term piece of work. Expect schema churn, `formData` shape changes, prefill transformer changes, and a backend PDF regeneration on vets-api. Reopened state suggests scope shifted at least once.
- **The "rolled into [#130754](https://github.com/department-of-veterans-affairs/va.gov-team/issues/130754)" reference in #113855** points to a picklist add-flow effort — confirm that issue is in scope before assuming the 674-without-686 case is solved.

# VAF21P-527EZ — Pension

- **Primary directory:** `src/applications/pensions/` ([original README](income-and-assets/original-readmes/527ez-pensions.md)). No legacy or duplicate copy on `main`.

## Issue index

| Issue | Title | Status |
|---|---|---|
| [#102564](https://github.com/department-of-veterans-affairs/va.gov-team/issues/102564) | Update 527EZ to latest OMB version | **In-progress (69% — 11/16)** |
| — | *— closed below —* | |
| [#121731](https://github.com/department-of-veterans-affairs/va.gov-team/issues/121731) | Access Disability Ratings to Drive 527EZ Recommendations | Closed (Feb 2026) |

## Diagrams

- **Main flow:** [income-and-assets/diagrams/527ez-main-flow.md](income-and-assets/diagrams/527ez-main-flow.md)
- **Downtime / off-ramp:** [income-and-assets/diagrams/527ez-downtime.md](income-and-assets/diagrams/527ez-downtime.md)

## Feature toggles in scope

| Toggle | Registry presence | Where it's read |
|---|---|---|
| `pensionFormEnabled` | `featureFlagNames.json:247` | `PensionsApp.jsx:25` |
| `pensionMultiplePageResponse` | `featureFlagNames.json:249` | `PensionsApp.jsx:26-27` |
| `pensionPdfFormAlignment` | `featureFlagNames.json:250` | `PensionsApp.jsx:29-30` |
| `pensionsBrowserMonitoringEnabled` | `featureFlagNames.json:251` | `PensionsApp.jsx:48` |
| `pensionRatingAlertLoggingEnabled` | `featureFlagNames.json:252` | `components/IntroductionPage.jsx:19-20` |
| `pensionIncomeAndAssetsClarification` | `featureFlagNames.json:248` | (registered, call sites TBD — check with team) |

## In-flight work

### [#102564](https://github.com/department-of-veterans-affairs/va.gov-team/issues/102564) — Update 527EZ to latest OMB version
VBA updating the paper 527EZ; the online form must be audited and aligned. Source-of-truth deck on SharePoint. Tracked under milestone 1508 (PBB Pension PDF form alignment).

| Surface | File |
|---|---|
| Toggle | `pensionPdfFormAlignment` (`featureFlagNames.json:250`) |
| Pages with `depends:` reading the toggle | scattered across `config/chapters/` per page; review every `depends:` that reads it |

### [#121731](https://github.com/department-of-veterans-affairs/va.gov-team/issues/121731) — Disability rating alert (Closed Feb 2026)
Veterans at 100% service-connected rating are nearly always ineligible for pension. Surface an alert via existing rating API. VBA explicitly directed **not to block submission** because of edge cases (10+ dependents, etc.).

| Surface | File |
|---|---|
| Component | `components/DisabilityRatingAlert.jsx` |
| API call | `DisabilityRatingAlert.jsx:23-25` (`apiRequest(${environment.API_URL}/v0/rated_disabilities)`) |
| Datadog logging (error/hidden/visible) | `DisabilityRatingAlert.jsx:54-61, 80-88, 92-99` |
| Rendered when | `components/IntroductionPage.jsx:41` (gated by `pensionRatingAlertLoggingEnabled`) |

## Gotchas, factors, and watch-outs

- **PDF alignment toggle**: `pensionPdfFormAlignment` flag exists in `featureFlagNames.json:250`. Mirror of the burials pattern — same architecture, same risk that conditional flow logic depends on a toggle that may flip without notice.
- **Don't add submission blocks to the disability-rating alert ([#121731](https://github.com/department-of-veterans-affairs/va.gov-team/issues/121731)).** This was an explicit VBA directive. If you ever expand that flow, the alert pattern is the right precedent — informational only.
- **Chapter 5 (Financial) is heavy with array builders.** Income sources, care expenses, medical expenses are all `arrayBuilderPages`. Updates that affect a per-row schema must respect the pattern's idiosyncrasies (summary page + per-item pages + ID stability across save-in-progress).
- **Form integrates closely with 0969 (Income & Asset Statement).** The 0969 is a *required supplemental* for some 527EZ applicants. Keep transitions in mind — when content changes here, the 0969 may need parallel edits.
- **Toggles to know:** `pensionFormEnabled`, `pensionIncomeAndAssetsClarification`, `pensionMultiplePageResponse`, `pensionPdfFormAlignment`, `pensionsBrowserMonitoringEnabled`, `pensionRatingAlertLoggingEnabled`.

# VAF21P-530EZ — Burial Benefits

- **Primary directory:** `src/applications/burials-ez/` ([original README](burials/original-readmes/530ez.md)). No legacy or duplicate copy on `main`.

## Issue index

| Issue | Title | Status |
|---|---|---|
| [#114166](https://github.com/department-of-veterans-affairs/va.gov-team/issues/114166) | Update 530EZ to latest OMB version | **In-progress (93% — 42/45). Milestone due 2026-05-31.** |
| [#121603](https://github.com/department-of-veterans-affairs/va.gov-team/issues/121603) | PBB \| 530 integration with BPDS | **In-progress (50% — 3/6). No assignee on the epic.** |

(No closed-but-relevant 530EZ issues at this snapshot.)

## Diagrams

- **Main flow:** [burials/diagrams/530ez-main-flow.md](burials/diagrams/530ez-main-flow.md)
- **Downtime / off-ramp:** [burials/diagrams/530ez-downtime.md](burials/diagrams/530ez-downtime.md)

## Feature toggles in scope

| Toggle | Registry presence | Where it's read |
|---|---|---|
| `burialPdfFormAlignment` | **Absent (TBD: confirm with team)** | Read from Redux at `BurialsApp.jsx:25`; mirrored to `sessionStorage` at `BurialsApp.jsx:49-51`; helper `showPdfFormAlignment()` at `utils/helpers.jsx:166-167`; `depends:` calls at `config/form.js:338, 349, 569`; chapter-level reads at `config/chapters/03-military-history/powPages.js:10,114,121` and `config/chapters/05-additional-information/supportingDocuments.js:4,30`; intro page at `components/IntroductionPage.jsx:12,159` |

## In-flight work

### [#114166](https://github.com/department-of-veterans-affairs/va.gov-team/issues/114166) — Update 530EZ to latest OMB version
VBA updating the paper 530EZ; the online form must be audited and aligned to the new BIM Change Document. Milestone due 2026-05-31.

| Surface | File |
|---|---|
| Toggle | `burialPdfFormAlignment` (Redux passthrough; not in registry) |
| `depends:` gates | `config/form.js:338, 349, 569`; `powPages.js:10, 114, 121`; `supportingDocuments.js:4, 30` |
| Migration TODO | `config/form.js:100` — "Change to 4 when PDF alignment feature toggle is enabled and enable migration" |

### [#121603](https://github.com/department-of-veterans-affairs/va.gov-team/issues/121603) — 530 integration with BPDS
Today: 530 collects JSON → PDF → CentralMail → OCR → BPDS. This wires 530 directly to BPDS so the JSON skips the PDF/OCR detour, accelerating Pension Optimization Initiative automation. **No assignee on the epic.** Mostly vets-api work.

No frontend artifacts found yet on `main`. **(TBD: ask team if FE work is in scope, e.g., confirmation messaging or submission tracking.)**

## Gotchas, factors, and watch-outs

- **The session-storage feature-toggle pattern is unusual.** `BurialsApp.jsx:46-56` writes `burialPdfFormAlignment` into `sessionStorage`, then `showPdfFormAlignment()` (`utils/helpers.jsx:166`) reads it. If you ever clear sessionStorage during a flow, conditional pages will silently change behavior. Tests that mock the toggle via Redux only will not match runtime.
- **`burialPdfFormAlignment` is not in `featureFlagNames.json`.** The Redux store still holds it (it's a passthrough of whatever vets-api returns), but the central registry is missing the entry. Trace the value end-to-end before assuming it's correctly wired.
- **`version: 3` with a comment** "// Change to 4 when PDF alignment feature toggle is enabled and enable migration" (`config/form.js:100`) — when the toggle goes live, a saved-in-progress migration must ship in lockstep. This is a foot-gun.
- **#121603 has no assignee** on the super epic; ownership unclear. BPDS integration is primarily vets-api work but downstream effects on confirmation messaging and submission tracking will hit the frontend.
- **External system surface:** CentralMail (legacy submission path), BPDS (new path), `externalServices.icmhs` (the only declared downtime dependency).
- **Two death-certificate pages exist** — `deathCertificate` and `deathCertificateRequired` — one or the other shows based on `relationshipToVeteran` (funeral director vs others). Easy to break by changing one and forgetting the other.

# VAF21P-0969 — Income and Asset Statement

- **Primary directory:** `src/applications/income-and-asset-statement/` ([original README](income-and-assets/original-readmes/0969.md)). Entry: `app-entry.jsx`, config: `config/form.js`, eleven chapter folders under `config/chapters/`. No legacy or duplicate copy on `main`.

## Issue index

| Issue | Title | Status |
|---|---|---|
| [#121601](https://github.com/department-of-veterans-affairs/va.gov-team/issues/121601) | PBB \| 0969 integration to BPDS | **In-progress (57% — 4/7). Milestone due 2026-03-06. No assignee on the epic.** |
| — | *— closed below —* | |
| [#121584](https://github.com/department-of-veterans-affairs/va.gov-team/issues/121584) | 0969 Post-MVP improvements based on usability testing | Closed (Mar 2026) |

## Diagrams

- **Main flow:** [income-and-assets/diagrams/0969-main-flow.md](income-and-assets/diagrams/0969-main-flow.md)
- **Downtime / off-ramp:** [income-and-assets/diagrams/0969-downtime.md](income-and-assets/diagrams/0969-downtime.md)

## Feature toggles in scope

| Toggle | Registry presence | Where it's read |
|---|---|---|
| `incomeAndAssetsFormEnabled` | `featureFlagNames.json:170` | `containers/App.jsx:29-30` |
| `incomeAndAssetsContentUpdates` | `featureFlagNames.json:171` | (registered; no read sites found in app — likely deprecated/cleanup candidate per README) |
| `incomeAndAssetsBrowserMonitoringEnabled` | `featureFlagNames.json:172` | `containers/App.jsx:49` |
| 0969 email notifications (per README) | **None found in registry** | (TBD: confirm with team — README mentions four email-notification toggles) |

## In-flight work

### [#121601](https://github.com/department-of-veterans-affairs/va.gov-team/issues/121601) — 0969 integration to BPDS
Same pattern as 530 BPDS integration — bypass PDF/CentralMail/OCR by sending JSON directly to BPDS. Milestone due 2026-03-06. **No assignee on the epic.** Mostly vets-api.

No frontend artifacts found yet on `main`. **(TBD: ask team about FE changes during BPDS cutover — confirmation page, error handling.)**

### [#121584](https://github.com/department-of-veterans-affairs/va.gov-team/issues/121584) — Post-MVP improvements (Closed Mar 2026)
Plain-language content edits, formatting standardization, removed standalone supporting-documents section in favor of inline list-loop, prefill support for all claimant types, design QA.

## Gotchas, factors, and watch-outs

- **Schema is commented out** at the top of `config/form.js:1`: `// import fullSchema from 'vets-json-schema/dist/21P-0969-schema.json';`. The form runs without a JSON schema-level validation source — schemas are defined per-chapter inside this app. Confirm with the team whether the central schema was deliberately abandoned or just not yet integrated.
- **`saveInProgress.messages` is fully commented out** (`config/form.js:44-50`). Veterans get default platform copy on save/expire/notFound. Likely a TODO or a stylistic choice; worth asking.
- **Required supplemental for 527EZ and 534EZ** (`README.md:5`). Content/copy changes here have ripple effects in the 527EZ flow.
- **Custom `PreSubmitInfo` component** (`containers/PreSubmitInfo.jsx`) — overrides the platform default. If you change preSubmit copy, edit there, not in `config/form.js`.
- **Heavy array-builder usage** (10 of 11 chapters). Same idiosyncrasies as 527EZ — summary page + per-item pages + ID stability on save-in-progress.
- **No assignee on #121601.** Same situation as #121603 (530 BPDS).
- **Toggles to know:** `incomeAndAssetsFormEnabled`, `incomeAndAssetsContentUpdates` (deprecated per README), `incomeAndAssetsBrowserMonitoringEnabled`, plus four email-notification toggles (per README).

# VAF21-0538 — Status of Dependents (Dependents Verification)

- **Primary directory:** `src/applications/dependents/dependents-verification/` ([original README](dependents/original-readmes/dependents-verification.md)). The legacy embedded 0538 inside `src/applications/dependents/view-dependents/manage-dependents/` is dormant — see [view-dependents README](dependents/original-readmes/view-dependents.md). Two places to know about; only `dependents-verification` ships.

## Issue index

| Issue | Title | Status |
|---|---|---|
| [#108392](https://github.com/department-of-veterans-affairs/va.gov-team/issues/108392) | Remove dependents during DV process (Phase 2) | **Open (no registered work; cross-listed with 686c)** |

(All other DV-related work is reflected in 686c issues — the 0538 itself has no headline epic. No closed-but-relevant issues.)

## Diagrams

- **Main flow:** [dependents/diagrams/0538-main-flow.md](dependents/diagrams/0538-main-flow.md)
- **Downtime / off-ramp:** [dependents/diagrams/0538-downtime.md](dependents/diagrams/0538-downtime.md)

## Feature toggles in scope

| Toggle | Registry presence | Where it's read |
|---|---|---|
| `vaDependentsVerification` (`va_dependents_verification`) | `featureFlagNames.json:347` | `containers/App.jsx:32` |
| `manageDependents` (`dependents_management`) | `featureFlagNames.json:177` | `tests/e2e/manage-dependents.cypress.spec.js:21` (legacy embedded 0538) |
| `dependentsModuleEnabled` (`dependents_module_enabled`) | `featureFlagNames.json:83` | switches between `/v0/dependents_applications/show` and `/dependents_benefits/v0/claims/show` |

## In-flight work

### [#108392](https://github.com/department-of-veterans-affairs/va.gov-team/issues/108392) — Remove dependents during DV process (Phase 2)
Phase 2 of the DV Tool — enable removing dependents. Originally planned via digital 0538; team pivoted to picklist in 686c first ([#120229](https://github.com/department-of-veterans-affairs/va.gov-team/issues/120229)). Whether 0538-based removal still happens is unclear.

| Surface | File |
|---|---|
| Candidate location (0538) | `components/DependentsInformation.jsx` |
| Alternative (686c picklist) | `config/chapters/formConfigRemovePicklist.js` + `components/picklist/*` (already shipped via #120229) |

**(TBD: ask team about long-term home for dependent removal.)**

## Gotchas, factors, and watch-outs

- **`downtime.dependencies` is fully commented out** (`config/form.js:55-64`) — so the prefill-required form has no declared external dependencies for the platform's downtime detection. That is suspicious: 0538 still depends on BGS for prefill (via the 686 prefill transformer pattern) and a real BGS outage will produce confusing errors. Likely needs to be re-enabled.
- **No declared schema** — `defaultDefinitions: {}`. Schemas live per-chapter; same story as 0969.
- **README says the form was last updated 2026-03-27** with `va_dependents_verification` "not fully enabled" yet. Verify current production state.
- **Status-changed ExitForm path** is a redirect, not a submission. Veterans who say "yes, things changed" get pushed to the 686 — there's no audit trail of that intent in the 0538 backend.
- **The legacy embedded 0538 in `view-dependents/manage-dependents/`** uses progressive-disclosures and is dormant. Don't accidentally revive it.
- **Phase 2 ([#108392](https://github.com/department-of-veterans-affairs/va.gov-team/issues/108392)) has no registered sub-issue work** — it's the closest thing to a "next-up" item for the 0538 stack.

# Cross-Cutting Initiatives

## [#113842](https://github.com/department-of-veterans-affairs/va.gov-team/issues/113842) — Improve dependent management findability on VA.gov

### What this affects
Site-wide IA, navigation, MyVA dashboard, and search results. Touches `vets-website` content/IA layer plus content-build (Drupal) for nav/search. Doesn't change the forms themselves.

### Where the work lives
Search results / nav definitions live in content-build, not vets-website. The frontend touchpoints in vets-website are the dependents application READMEs and probably some dashboard cards under `src/applications/personalization/dashboard/` — verify scope with the team.

### Status
In-progress (83% — 5/6 sub-issues closed).

### Gotchas
- Cross-team work with the **Platform Content IA team** — coordination overhead. Not all PRs land in this repo.
- Affects discoverability metrics; if you change the dashboard cards or the manage-dependents URL, ask analytics what they're tracking before you ship.

### Questions
- What's the remaining 1/6 sub-issue? Is it blocked on Platform IA or on this team?
- Are dashboard changes happening in `src/applications/personalization/dashboard/`, or in content-build, or both?

---

## [#122987](https://github.com/department-of-veterans-affairs/va.gov-team/issues/122987) (DRAFT) — Integrate dependent management info into MyVA

### What this affects
The authenticated MyVA experience (separate from the dependents apps themselves). Goal: surface dependent info inside the MyVA profile rather than in a separate "Manage dependents" silo. Touches `vets-website` (MyVA dashboard, Authenticated Experience Team's code) and partially the dependents team's existing card components.

### Where the work lives
Likely under `src/applications/personalization/` — MyVA dashboard / profile lives there. The DMT's "dependent-card patterns" referenced in the issue are probably in `src/applications/dependents/view-dependents/components/` (TBD — the issue body doesn't pin the exact path; ask the old team).

### Status
**Open, DRAFT** prefix in title — still scoping. Milestone 1665 at 50%.

### Gotchas
- DRAFT means scope is still moving. Don't build against this until it leaves DRAFT.
- Cross-team — owned partly by **Authenticated Experience Team**, not just the dependents team.

### Questions
- Why is it still DRAFT? What's the gating decision — scope, ownership, or timeline?
- Who owns it on the Authenticated Experience side? Is there a counterpart PM/EM to introduce me to?
- Which DMT components are being shared into MyVA — exact paths?

---

## [#119026](https://github.com/department-of-veterans-affairs/va.gov-team/issues/119026) — Schema integration on vets-website (FDF)

### What this affects
Build pipeline for **all** Fully Digital Forms-related work. Today, schemas are pulled from a private NPM/git package; the proposal was to fetch them from a Forms API endpoint at build time. Affects `vets-website` build (Webpack), CI/CD, and local dev.

### Where the work lives
`package.json:340`:
```json
"vets-json-schema": "https://va.ghe.com/software/vets-json-schema.git#ea08303608c47dc35ada003546bb43d7b8ba794b",
```
Pinned to a commit on internal GitHub Enterprise. No build-time GET implementation in the repo — the issue closed `not_planned` Sept 2025, so the work was abandoned in this form.

### Status
**Closed — `state_reason: not_planned`.** A descope.

### Gotchas
- The descope is unexplained beyond the closing comment. The architectural choice (schema-from-endpoint vs. pinned package) is real and not obviously settled.
- All FDF-related schema bumps still happen by updating the commit hash in package.json and re-running `yarn install-safe`. CI pinning means schema lag.

### Questions
- **Why was this closed `not_planned`?** What's the current FDF schema-distribution plan, and is the pinned-package approach the long-term answer or a stopgap?
- Are there constraints (CI security, build determinism, dev-loop friction) that killed the original plan? Worth knowing before someone re-proposes it.

---

## [#114340](https://github.com/department-of-veterans-affairs/va.gov-team/issues/114340) — 21-686c and 21-674 Separation, Submission Orchestration, Modularization (Engines)

### What this affects
Architectural backbone for clean release of 686 vs 674. Touches:
- **vets-api** — Rails Engines pattern, separating 686/674 backend code into independent modules, plus orchestration for multi-system submissions (BGS, VBMS eFolder, Lighthouse, Forms API).
- **vets-website** — chapter-level modularity in `src/applications/dependents/686c-674/config/chapters/` (the 674 sub-folder, the per-flow `formConfig*.js` files). No app-level split yet.

### Where the work lives
- vets-website: `src/applications/dependents/686c-674/config/chapters/674/` (674 subform), `config/chapters/formConfigAdd.js` (add-spouse), `config/chapters/formConfigRemoveV2.js` (legacy v2 remove), `config/chapters/formConfigRemovePicklist.js` (v3 picklist remove). All of these compose into `config/form.js`.
- vets-api: TBD — repo not present locally.

### Status
In-progress (89% — 66/74 sub-issues).

### Gotchas
- **The "Engines" naming is vets-api Rails terminology**, not webpack/JS. Don't go looking for vets-website packages.
- **"Submission orchestration" is the linchpin** — it's what makes failure recovery across BGS/VBMS/Lighthouse/Forms API survivable. Frontend confirmation/error UX hangs on it.
- **The remaining 8 sub-issues are likely the gnarliest** — that's where the failure-recovery and edge-case handling lives. Ask explicitly.
- **The 686c/674 backend documentation** at `documentation/686c.md` and `documentation/674.md` describes the **current** ordered job pattern (686 succeeds → 674 queues). Engines work will change this.

### Questions
- What are the 8 remaining sub-issues, and which of them block frontend release coupling?
- On the frontend, is the long-term plan to split into two `applications/` directories (`686c/` + `674/`)? If yes, on what timeline and who's leading?
- After engines lands, will `BGS::SubmitForm674Job` still queue from `SubmitForm686cJob` success, or will they be independent?

---

## Other forms not covered above (mentioned by issues)

- [#119718](https://github.com/department-of-veterans-affairs/va.gov-team/issues/119718) (Forms API / FDF Pilot) is listed under [VAF21-686c](#vaf21-686c--application-request-to-add-andor-remove-dependents), not here, because the pilot use-case is the 686c.
- [#121603](https://github.com/department-of-veterans-affairs/va.gov-team/issues/121603) / [#121601](https://github.com/department-of-veterans-affairs/va.gov-team/issues/121601) are listed under their respective forms (530, 0969) since each has a single-form deliverable.

---

# Glossary

Definitions extracted from the source doc, the codebase READMEs, and platform code. Items marked `(implied)` were inferred from context. `TBD` means I could not find or confidently infer a definition — ask the old team.

| Term | Meaning |
|---|---|
| **RBPS** | Rules-Based Processing System — automated VBA system that processes claims (esp. dependency-related) when input is clean. "Off-ramps to manual processing" are when RBPS rejects a case and a human adjudicator must take it. (implied — used heavily in #113855, #124480, #113850; not defined in repo.) |
| **BGS** | Benefits Gateway Service — VA's master service for benefits records. The 686 will not submit without a valid VA file number sourced from BGS. See `externalServices.bgs` in `src/platform/monitoring/DowntimeNotification/config/externalServices.js`. |
| **BPDS** | Benefits Processing Data Store — destination JSON store for forms. Today many forms reach BPDS only after PDF→CentralMail→OCR; the BPDS integration epics (#121603, #121601) wire JSON directly. (implied from issue bodies.) |
| **VBMS** | Veterans Benefits Management System — the case-management system of record. Mentioned in 686 downtime dependencies and in eFolder uploads. |
| **eFolder** | The veteran case file inside VBMS. Evidence (uploads, PDFs) lands here via the Claim Evidence API. (implied.) |
| **Lighthouse** | VA's external developer API gateway. Forms API (FDF) is reached through Lighthouse-style integrations. (implied — no in-repo definition.) |
| **OCTO** | Office of the Chief Technology Officer (VA OCTO). VA's central tech leadership; sponsors of Platform. (implied general VA usage.) |
| **OBA** | Office of Benefits Administration — VBA umbrella in the digital-experience-products repo path. (implied.) |
| **OMB** | Office of Management and Budget. Owns approval of every federal form revision; new PDF versions of 527EZ/530EZ/674 are "OMB versions." |
| **CentralMail** | The legacy intake system for paper/digital forms — JSON gets converted to PDF and sent to CentralMail, then OCRed back into BPDS. The BPDS integration epics bypass it. (implied.) |
| **Claim Evidence API** | API used by vets-api to upload supporting evidence (uploads from the form) into VBMS eFolder. Mentioned in #119718. (implied; no in-repo definition.) |
| **Forms API** | The new API the FDF effort introduces, accepting structured JSON form submissions. Pilot is 686c via #119718. (implied.) |
| **FDF** | Fully Digital Forms — the program that submits structured JSON instead of PDF, end-to-end. 686c is the pilot. Toggle: `dependents_enable_form_viewer_mfe`. |
| **MFE** | Micro-Frontend — small, embeddable UI hosted independently. The FDF "form viewer" MFE renders form data without a PDF, used by both VBMS and VA.gov. Toggle: `dependentsEnableFormViewerMFE`. |
| **MVP** | Minimum Viable Product — first release scope. Used in titles like "0969 Post-MVP Improvements" and "686c Removal Picklist MVP." |
| **EP** | End Product — VBA term for a tracked claim record / case type. (implied general VBA usage; not in repo.) |
| **MyVA** | The authenticated VA.gov dashboard / profile experience for signed-in veterans. Lives under `src/applications/personalization/`. |
| **DMT** | Dependents Management Team — the squad that has owned this portfolio. (implied — appears only in #122987 referencing "the DMT's dependent-card patterns.") |
| **DMC** | Dependents Management Cluster — VA program area that includes the 686/674/0538/view-dependents work. (implied from source-doc framing of "DMC roadmap.") |
| **PBB** | Pension, Burial & Benefits — the program area that owns 527EZ/530EZ/0969 work. (implied from issue-title prefixes like "PBB \| 530 integration with BPDS.") |
| **LSS** | Lifecycle Sustainment — the third roadmap track named in the source doc framing. (implied; not defined further in repo.) |
| **ADC** | TBD — not found in source doc or codebase. Ask the old team. |
| **RAD** | Release from Active Duty — as in "RAD-date conflicts" (#124480). Used to indicate timing of military separation. (implied.) |
| **IPT** | Integrated Project Team — cross-functional team; #120876 mentions "an Integrated Project Team led by Kayce White was kicked off 9/30/25." |
| **OCR** | Optical Character Recognition — the step that converts scanned/printed PDFs back into structured data. The PDF→CentralMail→OCR→BPDS pipeline is what BPDS-direct integration replaces. |
| **POI** | Pension Optimization Initiative — the umbrella effort to automate pension-claim processing. BPDS integrations feed it. (implied from #121603.) |
| **VBA** | Veterans Benefits Administration — the VA org that owns claim adjudication and the paper forms. |
| **VAF** | VA Form — the prefix used for federal-form numbering (e.g., VAF21-686c). (implied general usage.) |
| **SIP** | Save-In-Progress — the platform pattern that lets Veterans pause and resume a form. State persists per-form on vets-api. |

