# Lead Process

Lets users register Leads manually, track them through a simple status lifecycle, and
close them with a required reason.

See [requirements](../requirements/lead-process.md) for the original business
requirement and acceptance criteria.

## What's included

* **Fields** on the standard `Lead` object:
  * `Secondary_Phone__c` (Phone) — an alternate phone number alongside the standard `Phone` field.
  * `Closing_Reason__c` (Picklist) — Converted, No Budget, No Response, Duplicate, Not Interested, Other.
* **Lead Status** picklist values replaced with: `New` (default), `In Progress`, `Closed Won`, `Closed Lost`.
  Implemented via the `LeadStatus` **StandardValueSet** (`standardValueSets/LeadStatus.standardValueSet-meta.xml`),
  not a `CustomField` override on `Lead.Status` — Lead Status is one of the handful of
  standard picklists Salesforce manages as a StandardValueSet rather than per-field
  metadata; a `CustomField`-level `valueSet` on `Status` deploys "successfully" but is
  silently ignored.
  **This replaces the entire org-wide picklist.** It is only safe to deploy into an org
  with no (or negligible) existing Lead data: existing Leads sitting on the old default
  values (`Open - Not Contacted`, `Working - Contacted`, `Closed - Converted`,
  `Closed - Not Converted`) will have those values become inactive on their records —
  they won't error, but they can no longer be re-selected and won't match any
  `ISPICKVAL` logic. Do not deploy this into an org with meaningful existing Lead data
  without a remediation/migration plan for those records first.
* **Validation Rule** `Require_Closing_Reason` — blocks saving a Lead as Closed Won or
  Closed Lost unless `Closing_Reason__c` is set. Exempts the standard Lead Convert
  action (`NOT(ISCHANGED(IsConverted))`) — see "Converted flag" below for why.
* **Validation Rule** `Closing_Reason_Must_Match_Status` — blocks contradictory
  combinations: `Closing_Reason__c = Converted` is only valid when `Status = Closed Won`;
  it is rejected on `Closed Lost`, and any other reason is rejected on `Closed Won`.
  Unlike `Require_Closing_Reason`, this rule is **not** exempted for Lead Convert — a
  stale/incorrect non-blank reason still blocks conversion.
* **Flow** `Lead_Status_To_In_Progress_On_Activity` — record-triggered, after a Task is
  created against a Lead. If that Lead's Status is still `New`, it's moved to
  `In Progress`. Leads that are already In Progress or Closed are left untouched.
  Runs one SOQL query + one DML per triggering Task; a very large bulk Task insert
  across many distinct Leads in a single transaction (150+) could theoretically approach
  the synchronous SOQL limit. Not a concern for manual Task logging, the flow's intended
  use case, so this was accepted as-is rather than optimized.
* **Permission Set** `Lead_Process_Access` — grants Read/Edit field-level security on
  `Secondary_Phone__c` and `Closing_Reason__c`. Assign it to any user who needs to see
  or edit these fields; it does not grant object-level Lead permissions.
* **Page Layout** `Lead-Lead Layout` — retrieved from the `training` org and updated to
  include `Secondary_Phone__c` (next to `Phone`) and `Closing_Reason__c` (next to
  `Status`). Other Lead layouts (Sales/Support/Marketing) were not touched.

## Known gaps / manual steps

1. **Lead Status "Converted" flag.** Marking a Lead Status value as the one used by
   the standard Convert action (Setup → Object Manager → Lead → Fields & Relationships
   → Lead Status → edit a value → "Converted" checkbox) is **not exposed via the
   Metadata API** and cannot be deployed. After deploying this change to a new org,
   an admin must manually mark the appropriate status (typically `Closed Won`) as
   Converted, or the standard Lead Convert action may not behave as expected.
   The two validation rules assume `Closed Won` will be the Converted status; if a
   different status is chosen, revisit `Require_Closing_Reason`'s
   `NOT(ISCHANGED(IsConverted))` exemption and `Closing_Reason_Must_Match_Status`'s
   formula.
2. **Permission set assignment.** `Lead_Process_Access` grants field-level security on
   `Secondary_Phone__c` and `Closing_Reason__c` but is not auto-assigned to anyone —
   assign it (via Setup or a Permission Set Group) to the users who need it.

## Verification notes

* Deployed and confirmed in the `training` org via `sf project retrieve start` and
  direct Tooling API calls: both new fields and the `LeadStatus` StandardValueSet
  exist server-side with the expected definitions.
* The `Lead_Status_To_In_Progress_On_Activity` Flow was verified end-to-end: creating
  a Task against a `New` Lead flipped its Status to `In Progress`.
* In this specific org, `Secondary_Phone__c` and `Closing_Reason__c` took an unusually
  long time (6+ minutes, still ongoing as of this writing) to become queryable via
  SOQL/REST/Apex describe after deploying, even though the Metadata/Tooling API
  confirmed they were created correctly. This looked like a slow schema-cache refresh
  specific to this org/pod rather than a problem with the metadata itself. If you hit
  "No such column" errors on these fields right after a fresh deploy elsewhere, it's
  worth waiting a few minutes (or checking Setup → Object Manager directly, which may
  reflect the change sooner) before assuming something is wrong. The validation rule
  itself could not be end-to-end tested via API in this session for that reason —
  worth a quick manual check in Setup once the field is visible.

## Open items for the client

* Should Calendar Events (not just Tasks) also trigger the New → In Progress transition?
* Exact wording of the Closing Reason values above is a starter set — confirm or adjust.
