# Knowledge Transfer — Open Questions for the Old Team

Companion to [kt_form_briefing.md](kt_form_briefing.md). One section per form: items the in-app README already answers are removed or annotated `(README — answered: …)`.

## Table of Contents

1. [686c](#686c)
2. [674](#674)
3. [527EZ](#527ez)
4. [530EZ](#530ez)
5. [0969](#0969)
6. [0538](#0538)

---

## 686c

1. **#114340 Engines** — what's in the remaining 8/74 sub-issues, and which are blocking the frontend work? On the frontend, is the plan to split `686c-674/` into two `applications/` directories, or stay co-located and rely on chapter-level modularity?
2. **#119718 FDF pilot** — given the 4/27/2026 milestone has passed, what's the current go-live state? Is the JSON path live in production or still gated to staging? Who owns rollback?
3. **#119026 (closed `not_planned`)** — the package.json schema pin is still here at line 340. What replaced the build-time-GET approach for FDF schema distribution? Is the long-term plan still to leave it as a pinned NPM/git package, and if so, what was wrong with the original plan?
4. **#108392 (Phase 2)** — view-dependents README ([local copy](dependents/original-readmes/view-dependents.md)) confirms the embedded 0538 in `view-dependents/manage-dependents/` was never released because of accessibility issues with the progressive-disclosure pattern. *(README — partly answered: history and accessibility blocker.)* Open: which path forward — extend picklist (#120229 done), revive a digital 0538, or drop the embedded 0538 entirely?
5. **#124480 Other Causes off-ramps** — which of the five scenarios survived scope cuts? Are any deferred to RBPS/VBA permanently?
6. **`burialPdfFormAlignment` / `dependents_pension_check` registry gap** — both toggles are documented as active in the in-app READMEs ([burials](burials/original-readmes/530ez.md), [686c-674](dependents/original-readmes/686c-674.md)) but missing from `featureFlagNames.json`. Is the central registry meant to be exhaustive, or is there a deliberate reason to leave passthrough toggles off?
7. **`dependents_module_enabled` endpoint switch** — view-dependents README ([local copy](dependents/original-readmes/view-dependents.md)) confirms the toggle switches between `/v0/dependents_applications/show` and `/dependents_benefits/v0/claims/show`. *(README — partly answered: endpoint mapping.)* Open: which path is canonical, and on what timeline does the legacy endpoint go away?

---

## 674

1. **#120876** — what triggered the reopen? Has the OMB / Pension & Fiduciary Services PDF spec actually been finalized, or is the date still slipping toward Q4 2026? Need the source-of-truth SharePoint deck.
2. For #120876, will deprecated fields be removed from the JSON schema (breaking change) or just stop being rendered (additive)? That determines migration strategy.
3. The 674 array-builder page count is large (~17 pages × N students). Are there usability complaints we should address while regenerating the PDF, or is it strictly a like-for-like field swap?

---

## 527EZ

1. **#102564** — what's blocking the remaining 5/16 sub-issues? Have the OMB / VBA-side PDF revisions stabilized, or is the team still re-aligning fields on a moving target?
2. The disability-rating alert from #121731: README ([local copy](income-and-assets/original-readmes/527ez-pensions.md)) confirms purpose ("100% disability pays more than pension") and toggle name. *(README — partly answered: purpose.)* Open: is logging behind `pensionRatingAlertLoggingEnabled` (`featureFlagNames.json:252`) currently on in production? Are dismiss/ignore rates reviewed anywhere?
3. **Downtime list** — 527EZ declares only `externalServices.icmhs` (`config/form.js:36`). The README ([local copy](income-and-assets/original-readmes/527ez-pensions.md)) describes the submission path as `pension_claims_controller.rb` → Lighthouse via `Lighthouse::PensionBenefitIntakeJob` (no BGS or MVI mentioned), which is consistent with the slim downtime list. *(README — partly answered: pension does not appear to depend on BGS/MVI on the submit path.)* Open: confirm prefill path also avoids BGS, or document any other external dependency that should be in the downtime list.

---

## 530EZ

1. **#121603** — who's actually owning this? The epic has no assignee. Which sub-issues are blocking go-live, and is there frontend work on the confirmation page or status flow that this team owns?
2. The session-storage mirroring pattern in `BurialsApp.jsx:46-56` — was this intentional, or a workaround for sync `depends:` access to async Redux state? Is the intent to replace it with a Redux selector once the toggle stabilizes? *(README does not document this; see [local copy](burials/original-readmes/530ez.md).)*
3. `burialPdfFormAlignment` is missing from `featureFlagNames.json` — README ([local copy](burials/original-readmes/530ez.md)) confirms the toggle is active and "accompanies a v3→v4 in-progress form data migration." *(README — partly answered: confirms the toggle is real.)* Open: why is this off the registry, and is the registry meant to be exhaustive?
4. **#114166** is at 93% with a 2026-05-31 due date. What's blocking the remaining 3 sub-issues? Will it land before or after the BPDS integration in #121603, and which migration order is canonical?

---

## 0969

1. **#121601** — same as #121603: who owns it? Frontend changes during BPDS cutover (confirmation page, error handling)?
2. Why is the JSON schema import commented out at `config/form.js:1`? Is there a plan to re-add it, or is per-chapter schema definition the intentional pattern here? *(README does not document this; see [local copy](income-and-assets/original-readmes/0969.md).)*
3. Why are `saveInProgress.messages` commented out (`config/form.js:44-50`)? Pending copy review or deliberate?
4. The four email-notification toggles (`income_and_assets_error_email_notification`, `..._submitted_email_notification`, `..._email_notification`, `..._persistent_attachment_error_email_notification`) **are documented in the README** ([local copy](income-and-assets/original-readmes/0969.md)) with one-line descriptions, but **none of the four appear in `featureFlagNames.json`**. *(README — partly answered: purposes per toggle.)* Open: rollout plan, whether any are mid-experiment, and whether the central registry should be updated.

---

## 0538

1. **`downtime.dependencies`** is commented out (`config/form.js:55-64`) — was that deliberate (e.g. while a BGS instability is being investigated) or stale code? Should we re-enable BGS / MVI / VA Profile / VBMS? *(README does not document this; see [local copy](dependents/original-readmes/dependents-verification.md).)*
2. **#108392 Phase 2 — largely answered by the README.** The dependents-verification README ([local copy](dependents/original-readmes/dependents-verification.md)) states: *"The form itself does allow to remove spouse and child dependents, but our team opted to not duplicate the work that 686c-674 is doing. Additionally, Form 21-0538 doesn't allow removal of a dependent parent."* So removal lives in 686c (the picklist via #120229), and 0538 stays declarative on purpose. *(README — answered: the 0538 is not the long-term home for removal.)* The remaining question for the team is narrower: what scope, if any, of #108392 still applies to the 0538 codebase, vs. is fully owned by the 686 picklist?
3. **Legacy embedded 0538 in `view-dependents/manage-dependents/`** — view-dependents README ([local copy](dependents/original-readmes/view-dependents.md)) documents the never-released, accessibility-flawed embedded form. *(README — partly answered: history.)* Open: is there a plan to delete it, or does it stay for reference?
4. **localStorage diary date / no diary-date API** — view-dependents README ([local copy](dependents/original-readmes/view-dependents.md)) explains the localStorage dismiss-date approach was a stakeholder call ("Our stakeholder did not want us to store this date on the server, so we opted to use local storage after informing them of the limitations") and that *"there is no API or method available to retrieve the diary date (8 year timer that VA uses to send out a letter)."* *(README — answered: the why, and the missing-API state.)* Open question for the team is still forward-looking: is anyone owning a vets-api effort to expose that diary date so the alert can be event-driven rather than time-based?
