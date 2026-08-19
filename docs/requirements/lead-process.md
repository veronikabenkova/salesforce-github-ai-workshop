# Requirement: Simple Lead Process

## Business Need

Help users register new Leads, track their current status, and close Leads with a
clear closing reason.

## Scope

### Data Model

Uses the standard **Lead** object — no new custom object. Leads are created manually
in Salesforce only (no web-to-lead, no API/integration inflow).

| Field (label)    | Type              | Notes                                                        |
|-------------------|-------------------|---------------------------------------------------------------|
| First Name        | Standard          | Existing standard field                                       |
| Last Name         | Standard          | Existing standard field, required                              |
| Phone             | Standard          | Used as the primary phone                                      |
| Secondary Phone   | Phone (new)       | `Secondary_Phone__c`, optional                                  |
| Email             | Standard          | Existing standard field                                        |
| Address           | Standard compound | Street / City / State / Postal Code / Country                  |
| Lead Status       | Picklist (standard, values replaced) | New, In Progress, Closed Won, Closed Lost   |
| Closing Reason    | Picklist (new)    | `Closing_Reason__c`; Converted, No Budget, No Response, Duplicate, Not Interested, Other |

### Business Rules

* A new Lead defaults to Status = **New**.
* **Closing Reason is required** once Status is **Closed Won** or **Closed Lost**
  (enforced by a validation rule).

### Automation

* When a user creates an Activity (Task — e.g. Log a Call, email) against a Lead that
  is still **New**, the Lead's Status automatically changes to **In Progress**.
* Implemented as a single simple record-triggered Flow on Task creation. Deliberately
  does not touch Leads that are already In Progress or Closed, so it never reopens or
  overwrites a closed Lead.

## Out of Scope

* Web-to-Lead, Lead assignment rules, lead scoring/qualification automation.
* Integrations or API-based Lead creation.
* Changes to the standard Lead Conversion process (Account/Contact/Opportunity).
* Calendar Events triggering the status change (Task/Activity only, for this iteration).
* Localization (cs/sk translations) — not requested for this feature.

## Acceptance Criteria

* [ ] `Secondary_Phone__c` and `Closing_Reason__c` fields deployed on Lead.
* [ ] Lead Status picklist reduced to New / In Progress / Closed Won / Closed Lost.
* [ ] Validation rule blocks saving a Closed Won/Lost Lead without a Closing Reason.
* [ ] Creating a Task against a New Lead flips its Status to In Progress; creating a
      Task against an In Progress or Closed Lead leaves its Status unchanged.
* [ ] Metadata successfully retrieved back from the org after deployment (round-trip check).
