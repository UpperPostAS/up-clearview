---
{"dg-publish":true,"permalink":"/up-clearview-data-sources/"}
---

**SharePoint lists powering the application**

## About This Reference

All operational data for Upper Post Clearview lives in SharePoint at `site-510-upv-upper-post-veterans-homes`. The CSV exports in `data/` reflect the current list schemas. Field types below come from the embedded SharePoint `ListSchema`; any cells that look like JSON arrays (for multi-select choices) or expanded lookup columns reflect how SharePoint formats exports. Pair this reference with the [[UP Clearview – Technical Documentation\|technical documentation]] when wiring formulas or workflows, and check the [[UP Clearview - Development Roadmap\|roadmap]] for pending schema updates.

## Account Information
Tracks service accounts tied to a resident so staff can pay bills and reference credentials.
- `Account Number` (computed link to item) – the structured account identifier shown in the app.
- `Participant` (text, renamed Title) – head of household connected to the account; should align with `Current Residents`.
- `Provider Name` (lookup to `Account Providers`) – normalizes billing details and portal links.
- `Billing Frequency`, `Account Password`, `Username`, `Unlinked Name` (text) – cadence, optional credentials, and alternate labels.
- `Is Payable Account`, `Is Active` (Yes/No) – drive UI for payable accounts and filter inactive records.
- `Next Due Date` (date), `Recurring Cost` (number/currency) – schedule and budgeting data.
- `Account Username` (Person/Group multi-select) – tracks staff with saved credentials.

## Account Providers
Reference table for organizations that issue participant accounts.
- `Account Provider` (computed Title) – provider display name.
- `Account Type`, `Online Portal URL` (text) – categorization and login address.
- `Provider Details` (multi-line text) – troubleshooting notes, escalation steps.

## Agencies
Directory of partner organizations that employ service providers.
- `Agency Name` (computed Title) plus `Phone`, `Website`, `Services Provided` (text/URL).
- `Service Providers` (calculated view column) lists linked contacts from the Service Providers list.

## Archived Records
Not included in this export. Based on the app design, expect it to store finalized referral/resident records with `Record Type`, `Linked Referral/Resident` lookups, `Unit`, `Archive Date/Reason`, and narrative outcome notes. Verify once the CSV is available.

## Current Residents
Active households living on site, with demographic and tenancy data.
- `Head of Household` (computed Title) – primary resident name.
- `Unit Number` (lookup to `Units`) plus `Unit Subsidy` (expanded lookup value) – keeps the resident tied to the physical unit and its subsidy program.
- `Preferred Name`, `Preferred Language`, `Phone`, `Email`, case numbers, and benefit flags (text/multi-choice).
- `Disability`, `Benefits` (multi-select choices exported as JSON arrays).
- `Household Status` (choice), `Household Size` (number), `Client Obligation` (currency).
- `Date Referred`, `Intake Completed`, `Lease Signing Date`, `Move-Out Date` (dates) – workflow milestones.
- `Associated Providers` (multi-lookup to `Service Providers`) – maintains connected partners.

## Referrals & Applicants
Pipeline for pre-housing services, mirroring most resident data while households move toward lease-up.
- `Head of Household` (computed Title) plus the same demographic and benefits fields as `Current Residents`.
- `Unit Number` (lookup to `Units`) – drives eligibility checks shown in the app.
- `Household Status` (choice) – workflow stage (Referral → Applicant → Occupant/Former).
- `Homelessness Eligibility Confirmation`, `Close-out Reason` (choices with color-formatting in SharePoint).
- `Close-out Date/Notes`, `CoC Self Cert Months` (dates/number) – reporting metrics.
- `State issued ID or Driver's License` (Yes/No) – documents on hand.
- `Associated Providers` (multi-lookup to `Service Providers`) – carried forward if the referral becomes a resident.

## Service Providers
Contact list for individuals working with households.
- `Name` (computed Title) plus `Phone`, `Email`.
- `Organization` (currently stored as text; values mirror `Agencies.Agency Name`).
- `Program`, `Title/Role` (text) – clarifies scope of support.
- `Is active` (Yes/No) – filters obsolete contacts.

## Unit Types
Program definitions applied to multiple physical units.
- `Unit Type` (Title) – label surfaced in color-coding.
- `Subsidy`, `Referral Source`, `Subsidy Administrator` (text) – high-level program metadata.
- `HMIS Program ID`, `HS Vendor ID` (text/number) – reporting identifiers.
- `Bedrooms`, `Minimum Occupancy`, `Maximum Occupancy`, `Additional Occupancy Requirements` (text/number).
- Eligibility & documentation fields: `Maximum Income`, `Income Restriction`, `Homelessness Eligibility Requirement`, `Homelessness Verification Requirements`, `Disability Documentation Options`, `Veteran Requirement`.
- Checkbox columns (numerous Yes/No fields) track required documents: IDs, SS cards, ROI forms, HUD verification, chronic homelessness, LTH eligibility, Professional Statement of Need, etc.
- `Additional Criteria`, `Exclusion Criteria`, `Notes` (long text) – narrative guidance for staff.

## Units
Inventory of physical units and their real-time status.
- `Unit` (computed Title) – unit identifier.
- `Unit Type` (lookup to `Unit Types`), with expanded columns such as `Unit Type: Subsidy` and `Unit Type: Maximum Income` to surface program metadata inline.
- Occupancy fields: `Is occupied`, `Has given notice`, `Is offline`, `Has referral`, `ADA` (Yes/No).
- Workflow dates: `Date Available`, `Referral Requested`.
- `Occupancy Status` (choice) – vacancy categories used in the Units view.
- `Occupant Household` (lookup to `Current Residents`) and `Referral Household` (lookup to `Referrals & Applicants`) keep the pipeline aligned to each unit.
- `Notes` (multi-line text) – maintenance, access, or exception details.

## Lookup Relationships
- `Units.Unit Type` → `Unit Types` supplies program requirements for vacancy screens.
- `Current Residents.Unit Number` and `Referrals & Applicants.Unit Number` → `Units`, aligning people with units and enabling vacancy toggles.
- `Units.Occupant Household` → `Current Residents`, and `Units.Referral Household` → `Referrals & Applicants`, keeping the pipeline in sync.
- `Account Information.Provider Name` → `Account Providers`, reusing portal instructions; `Account Information.Participant` mirrors resident records to keep billing aligned.
- `Current Residents.Associated Providers` and `Referrals & Applicants.Associated Providers` → `Service Providers`, preserving partnership continuity when referrals convert to residents.
- `Service Providers.Organization` values pair with `Agencies.Agency Name`; agencies display their linked providers through the `Service Providers` view column.

Confirm each lookup in SharePoint list settings before editing formulas or automations, especially where the export currently shows plain text (e.g., `Participant`, `Organization`). Those fields are expected to be configured as lookups even if the CSV renders display values only.
