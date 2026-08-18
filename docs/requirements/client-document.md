# Requirement: Client Document Management

## Business Need

The business needs to keep basic information about documents related to a client
(e.g. ID card, passport, contract) directly on the Account in Salesforce.

## Scope

### Data Model

A new custom object, **Client Document**, storing one record per document.

| Field (label)          | Type          | Purpose                                   | Notes                                   |
|-------------------------|---------------|--------------------------------------------|------------------------------------------|
| Name                    | Auto Number   | Internal record identifier                  | Format `CD-{00000}`                      |
| Document Number         | Text          | Identifier printed on the document          | Required, no uniqueness constraint       |
| Document Type           | Picklist (local) | ID Card, Passport, Driving License, Contract, Other | Required |
| Issue Date              | Date          | Date the document was issued                | Required                                 |
| Expiration Date         | Date          | Date the document expires                   | Optional (not all documents expire)      |
| Account                 | Master-Detail | The Account the document belongs to         | Required; sharing controlled by parent   |
| Document Status         | Picklist (local) | Valid, Expired, Revoked                  | Required, manual only (no automation)    |

### User Interface

* A Custom Tab for Client Document.
* A Lightning Record Page for the Client Document object, set as the org default:
  highlights panel (Document Type, Document Status, Expiration Date), two-column
  details section, custom icon.
* A "Client Documents" related list added to the default **Account-Account Layout**
  (Account has no custom Lightning Record Page in this org).

### Security

* A dedicated Permission Set, `Client_Document_Access`, granting:
  * Object permissions (Read/Create/Edit/Delete) on Client Document
  * Field-level security on all fields above
  * Tab visibility for the Client Document tab
* No changes to existing profiles.

### Localization

* Czech (`cs`) and Slovak (`sk`) translations of: object label + plural label,
  all field labels, all picklist values (Document Type, Document Status), and
  the Custom Tab label.
* Help text and validation messages are out of scope (none exist).

## Out of Scope

* Automation (flows, triggers, validation rules) — not requested for this iteration.
* Reporting/dashboards.
* Integration with external document storage.

## Acceptance Criteria

* [ ] Custom object and fields deployed and visible on a FlexiPage.
* [ ] Permission set deployable and grants full CRUD + FLS on the object.
* [ ] Czech and Slovak translations deployed for object, fields, and picklist values.
* [ ] Metadata successfully retrieved back from the org after deployment (round-trip check).
