## QualiWare Audit Clone Automation

Custom QualiWare web-front-end script that adds a **Clone** button to `AuditProgram` object pages and automates the creation of a new audit cycle.

## Description

Clone QualiWare AuditProgram objects with linked QualityAudits, target-year date updates, field resets, and Risk/AcknowledgeList relationship preservation.

## What this script does

This script is designed for a QualiWare HTML Feature / browser JavaScript context. When a user opens an `AuditProgram` object, the script injects a **Clone** button into the page header. When selected, it prompts for a new AuditProgram name ending in a year, such as:

```text
Internal Audits - 2047
```

The script then:

1. Clones the current `AuditProgram`.
2. Extracts the target year from the new AuditProgram name.
3. Updates selected date fields to the target year while preserving month/day where possible.
4. Clones each linked `QualityAudit` from the source AuditProgram's `HasAudit` attribute.
5. Rebuilds the cloned AuditProgram's `HasAudit` links to point to the newly cloned QualityAudits.
6. Clears historical, status, revision, result, classification, and operational fields that should not carry into the new cycle.
7. Preserves selected reverse-link relationships:
   - `Risk` relationships that point to the source AP/QA.
   - `AcknowledgeList.ToBeAcknowledged` relationships that point to the source AP/QA.
8. Displays a simple completion message: **Clone is Complete**.

## Intended environment

This script is intended to run inside the QualiWare web front end where the following client-side objects and methods are available:

- `window.$$`
- `window.$$.Data`
- `Clone(false)`
- `SaveObject()`
- `GetObjectSetByIds(...)`
- `UsingObjects()`
- `RelatedObjects()` / `GetRelatedObjects()` where available
- `QisCommon.Oid`
- `QisCommon.ObjectInstantiationLevel.FullData`
- `RepositoryObject`

It is not intended to run as a standalone Node.js script.

## Main templates and attributes

### Source and cloned objects

- `AuditProgram`
- `QualityAudit`

### AuditProgram attributes used

- `HasAudit`
- `StartDate`
- `ApprovalDate`
- `RiskConsiderationDate`

### QualityAudit attributes used

- `TargetRegulation`
- `TargetProcess`
- `HasAuditee`
- `RiskConsiderationDate`
- `VerificationDate`
- `AuditDate`

### Reverse-link templates and attributes

- `Risk`
  - The script detects the MultiLink attribute that already contains the source AP/QA and adds the cloned AP/QA to that same attribute.
- `AcknowledgeList`
  - `ToBeAcknowledged`
  - If this attribute contains the source AP/QA, the cloned AP/QA is added to the same list.

## Naming rules

The cloned AuditProgram name must end with a valid four-digit year beginning with `20`.

Accepted examples:

```text
Internal Audits - 2047
External Audits - 2047
```

QualityAudit clone names are generated from the source QualityAudit name by removing the old trailing year and appending the new year.

If the new AuditProgram name starts with `External`, and the source QualityAudit name does not already start with `External`, the cloned QualityAudit name is prefixed with `External`.

Example:

```text
Source QA: Process Audit - 2043
Target AP: External Audits - 2047
Cloned QA: External Process Audit - 2047
```

## Date handling

The script updates configured date fields to the target year and attempts to preserve the existing date format. It supports:

- JavaScript `Date` objects
- QualiWare wrapped date attributes such as `{ Format: "DateTime", Value: ... }`
- Microsoft date strings such as `/Date(1781053260000)/`
- `YYYY-MM-DD`, `YYYY.MM.DD`, or similar year-first strings
- `MM/DD/YYYY`, `MM.DD.YYYY`, or similar month-first strings
- Other parseable ISO-like date strings

If a date such as February 29 is moved to a non-leap year, the day is capped to the last valid day of that month.

## Field reset behavior

The script clears fields that should not carry forward into a new audit cycle, including but not limited to:

- Revision fields
- Classification fields
- Result fields
- Completion/progress fields
- Cost/time tracking fields
- Circulation fields
- Nonconformance and ChangeRequest links on QualityAudits
- HasAuditee on cloned QualityAudits

Review the `clearAuditProgramFields()` and `clearQualityAuditFields()` functions before deploying, because these lists are business-rule specific.

## Reverse-link preservation

After each cloned object is saved, the script searches for objects that use the source object.

For each source `QualityAudit`, it:

1. Clones and saves the new QualityAudit.
2. Finds reverse-linked objects using `UsingObjects()` and related fallback calls.
3. If a reverse-linked object is a `Risk`, it adds the cloned QualityAudit to the same MultiLink attribute that contains the source QualityAudit.
4. If a reverse-linked object is an `AcknowledgeList`, it checks `ToBeAcknowledged`; if the source QualityAudit is present, it adds the cloned QualityAudit.

For the source `AuditProgram`, it performs the same relationship-preservation step after the cloned AuditProgram is saved.

## Installation

1. Open the relevant QualiWare configuration area where HTML Feature JavaScript can be added.
2. Create or edit the HTML Feature that should run on `AuditProgram` object pages.
3. Paste the full contents of `audit-program-clone-automation.commented.js`.
4. Save/publish the QualiWare configuration as required in your environment.
5. Refresh an `AuditProgram` object page.
6. Confirm the **Clone** button appears in the page header.

## Usage

1. Open the source `AuditProgram` object in QualiWare.
2. Click **Clone**.
3. Enter the new AuditProgram name. The name must end with a year.
4. Wait for the process to finish.
5. Confirm the completion message appears.
6. Review the cloned AuditProgram and linked QualityAudits.
7. Verify that expected `Risk` and `AcknowledgeList.ToBeAcknowledged` relationships were updated.

## Safety controls

The script includes a browser-session running flag to prevent duplicate clone operations from accidental double-clicks or duplicate button events.

The clone process runs sequentially to reduce the risk of simultaneous saves against the same related object.

## Known limitations

- The script depends on QualiWare client-side APIs and must run inside a QualiWare object page.
- Reverse-link preservation depends on `UsingObjects()` or fallback relationship methods returning the relevant objects.
- If an expected Risk or AcknowledgeList relationship is not returned by QualiWare's relationship APIs, the script will not update it.
- The field-clearing lists are hard-coded business rules and should be reviewed before use in another repository or client environment.
- The script does not include server-side transaction rollback. If part of the process fails after some objects are created, manual cleanup may be required.


## Testing checklist

Before using this in production, test with a non-production QualiWare repository/configuration.

Minimum checks:

- Clone button appears only on `AuditProgram` pages.
- Clone name validation works.
- New AuditProgram is created with the correct name.
- Linked QualityAudits are created with the correct names.
- `HasAudit` on the cloned AuditProgram points to the cloned QualityAudits.
- Target date fields use the new year.
- Cleared fields are blank or empty as expected.
- `Risk` relationships are preserved where expected.
- `AcknowledgeList.ToBeAcknowledged` is updated where expected.
- Repeated clicks do not create duplicate clones.

