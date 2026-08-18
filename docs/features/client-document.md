# Client Document

Stores basic information about documents belonging to a client (ID card, passport,
driving license, contract, etc.), one record per document, linked to the client's Account.

See [requirements](../requirements/client-document.md) for the original business
requirement and acceptance criteria.

## What's included

* **Custom object** `Client_Document__c` — Master-Detail to Account (sharing controlled
  by parent). Name field is an Auto Number, `CD-{00000}`.
* **Fields**: Document Number (text), Document Type (picklist), Issue Date, Expiration
  Date, Document Status (picklist), Account (Master-Detail).
* **Custom Tab** `Client_Document__c` for direct access via the App Launcher.
* **Lightning Record Page** `Client_Document_Record_Page`, set as the org default for
  the object — highlights panel (Type/Status/Expiration) plus a two-column detail layout.
* **Permission Set** `Client_Document_Access` — full CRUD + field-level security on
  Client Document, Read on Account (required by the Master-Detail relationship), and
  tab visibility. Assign it to any user who should work with Client Documents.
* **Translations** — Czech (`cs`) and Slovak (`sk`) for the object/plural label, all
  field labels, all picklist values, and the tab label.

## Known gap: Account related list

A "Client Documents" related list was added to `Account-Account Layout` in metadata
(`force-app/main/default/layouts/Account-Account Layout.layout-meta.xml`), but it
**cannot be deployed via the Metadata API alone**: Salesforce requires a brand-new
custom object's related list to be added to an existing standard object's layout once
through the Setup UI before automated deploys can reference it (deploy fails with
`Cannot find related list:Client_Documents__r`, confirmed as a server-side, not
naming, issue).

**To finish this piece in any org:**
1. Setup → Object Manager → Account → Page Layouts → Account Layout.
2. Drag the "Client Documents" related list onto the layout, save.
3. Retrieve the layout (`sf project retrieve start --metadata "Layout:Account-Account Layout"`)
   to capture the working identifier, and commit the update if it differs from what's
   in this branch.

Everything else (object, fields, tab, FlexiPage, permission set, translations) deploys
and round-trips cleanly with no manual steps.
