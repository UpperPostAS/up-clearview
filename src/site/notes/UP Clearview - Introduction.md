---
{"dg-publish":true,"permalink":"/up-clearview-introduction/","dgShowToc":true}
---

**Supporting participants throughout their entire journey**

---

## About This Guide

This document is the day-in-the-life walkthrough for service staff. It follows a participant from the moment a unit opens through move-out, highlighting which UP Clearview screens and workflows support each step. For overall orientation, start at the [[UP Clearview - Welcome & Index\|documentation hub]]. If you need implementation details, hop to the [[UP Clearview – Technical Documentation\|technical manual]] or the [[UP Clearview – Data Sources\|data source reference]].

UP Clearview sits alongside existing CommonBond processes; this guide shows how the app keeps information in one place so staff always know the next action.

---

## Following a Participant's Journey

### 1. A Unit Becomes Available

**The Units view** shows all units at a glance, color-coded by type—LTH Housing Support is green, Section 8 is blue, CoC in yellow, HUD-VASH is purple.

Toggle "Show Vacancies" to see only current and upcoming vacancies. Click on any unit to see:

**On the left side (static information):**
- Bedrooms, ADA accessibility, unit type
- Subsidy administrator and referral source
- Income limits, homelessness requirements, disability documentation requirements
- Everything needed to make an accurate referral request

**On the right side (current status):**
- Current occupant (if occupied)
- Date referral was requested
- Referral household information (if referral received)
- Expected availability date (if offline)

When requesting a referral, update "Date Referral Requested." When the referral arrives, click "Receive Referral."

### 2. Receiving a Referral

The **"Receive Referral" workflow** opens a form to record the household head's name. The app automatically:
- Links the referral to the specific unit
- Creates a household record for the referral
- Records today's date as "Date Referred"
- Marks the unit as having an active referral

After saving, you're taken directly to the referral's information page.

### 3. Referrals & Applicants View and Information Pages

The **Referrals & Applicants view** shows all participants who are receiving pre-housing services from CommonBond aimed at securing housing in a vacant unit at Upper Post. Click on any referral to access their information page. At the top are workflow buttons:

### 4. Referral & Applicant Workflows

**Phone Screening** (available now): Opens a form with all standard phone screening questions. Complete it during the call, save, and the information becomes part of the permanent record.

**Intake** (in development): Will guide staff through collecting and verifying required documentation based on the specific unit type's requirements.

**Convert to Resident** (available now): Converts an approved applicant to a resident upon move-in. Staff select the lease signing date, assign a coordinator, and choose service providers. The workflow automatically creates the resident record, updates the unit status, removes the referral, and opens the new resident's information page for any final edits.

**Close Referral** (available now): Records the date, reason, and notes when a referral declines or needs to be closed out. Automatically updates status to "Former Referral" and clears the unit for the next referral.

### 5. Associated Service Providers

From the moment we receive a referral, we maintain and build awareness of external partners working with—or potentially working with—this participant. Service provider associations are tracked throughout the participant's time in the program.

Associated service providers are visible both on the information page and on a separate page called Resident Associations.

On the associations page, clicking on any service provider will bring up their contact information, program, and organizational affiliation. Navigate to the Organizations view to see agency contact details and the providers affiliated with each organization.

### 6. They Become a Resident

Once someone moves in, they appear in the **Residents view**, showing all occupied units with resident names and unit types.

Click on any resident to see their information:
- Demographics and contact information
- Income, benefits, veteran status
- Lease dates, case numbers, household size
- Any notes or relevant information

Information can be edited as needed. New fields should be added as providers voice their needs. Whatever information staff may find useful at a glance could be integrated through the introduction of new optional fields.

### 7. Housing Support Bill Pay (Resident Accounts)

For Housing Support participants, the **Resident Accounts** section tracks information about accounts with various providers (utilities, internet, etc.). This includes usernames, account numbers, billing frequency, and recurring costs.

Having this information centralized means:
- Bill pay can happen at the same time each month (or as needed, depending on the case)
- Any authorized staff member can access account information when the primary person is unavailable
- No need to search through multiple documents to find account numbers

Toggle "Is Payable Account" to show billing fields only for accounts CommonBond pays. This allows staff to use the resident accounts feature for other types of accounts as well, such as with Hennepin County InfoSeek. 

**Note:** Password security should be addressed before full deployment. Interim solution is using browser-based password managers.

### 8. Associated Family Members

For family units, the **Household Members section** (beginning development on Friday) will track everyone living in the unit—names, ages, relationships. Currently, this information lives in Yardi or on other intake paperwork and doesn't provide the level of detail most helpful for service coordination. UP Clearview will allow staff to add or remove household members as composition changes and track information relevant to service delivery.

### 9. Move-Out

When a participant exits, the **"Move Out" function** on their info page archives their record for future reference and removes them from the active residents list. The unit becomes available for the next referral.

---

## What Still Needs Work

**Intake Workflow** needs the data entry form built to record intake completion and documentation verification.

**Household Members section** interface is designed and will be connected to data with add/edit functionality this week.

**Move-Out process** needs to complete unit status updates (vacancy marking, availability dates).

**Service Provider creation** requires permission troubleshooting to enable adding new providers through the app interface.

**Password security** for account tracking needs resolution before full deployment of the bill pay features.

**Testing suite** needs comprehensive test scripts developed for all workflows.

**Empty field hiding** in view mode will reduce visual clutter by showing only populated fields.

**User notifications** will provide confirmation messages when actions are saved successfully.

For prioritization details and owners, refer to the [[UP Clearview - Development Roadmap\|Development Roadmap]].

---

## Future Capabilities

As UP Clearview develops, additional functionality will enhance workflow efficiency:

**Automated Reminders**: Email or Teams notifications when professional statements of need require renewal or other time-sensitive tasks are approaching.

**Guided Workflows**: Links to training materials and instructions embedded within workflows, providing just-in-time guidance without leaving the app.

**In-House Responsiveness**: Because this tool is built in-house, it can be adapted quickly to meet the actual needs of staff doing the work, rather than waiting for external vendors.

**Power Platform Integration**: Built on SharePoint with Power Apps and Power Automate, the tool can integrate with other systems and automate additional processes, even with basic licensing.

**Dashboard Views**: At-a-glance information about upcoming renewal dates, service plan follow-ups, and other time-sensitive items.

**ETO Integration Potential**: Depending on integration feasibility, new participant creation in ETO could be automated through the referral workflow, reducing duplicate data entry.

**Streamlined HMIS Updates**: Having centralized, up-to-date participant information will simplify the process of updating HMIS records.

**Workflow Automation**: Continued development of automated workflows will handle more of the tedious administrative tasks, allowing staff to spend time on direct service provision rather than data management.

To help test new functionality as it ships, keep the [[UP Clearview - Testing Guide\|testing guide]] handy.

---

## The Bottom Line

UP Clearview brings participant information together in one place and provides workflows for common tasks. It's designed to support the work staff are already doing, making it easier to find what you need and reducing time spent on administrative tasks.

---

## Data Sources & Schemas

UP Clearview stores its operational data in SharePoint lists within the `site-510-upv-upper-post-veterans-homes` site. Each list backs one or more screens in the app and is designed to track a specific part of the participant journey. Field names reflect what is surfaced in Power Apps today and should be validated against the live list configuration before making schema changes.

For the full column-by-column breakdown, jump to the [[UP Clearview – Data Sources\|data source reference]].

### Cross-Cutting Patterns
- Lookup columns tie lists together (e.g., units point to unit types, referrals point to units, residents point to both units and service providers).
- Choice columns capture statuses that drive workflow buttons (referral state, unit availability, archive reason).
- Yes/No flags (such as `Is Payable Account` or `Show Vacancy`) power conditional UI elements and filters.
- Date columns (move-in, referral requested, last paid) are used to trigger automations and reminders.

### Account Information
**Purpose:** Tracks every account a resident or household uses for recurring services so staff can manage bill pay and documentation.
- `Account` (Title): Friendly name that matches what staff recognize (e.g., "Xcel Energy – Apt 102").
- `Resident` (Lookup → Current Residents): Connects the account to the active tenant record.
- `Account Provider` (Lookup → Account Providers): Standardizes provider metadata and contact information.
- `Account Type` (Choice): Categorizes accounts (utilities, communications, benefits portals) for filtering.
- `Username / Account Number` (Single line): Stores credentials identifiers; password storage is intentionally excluded pending security decisions.
- `Billing Frequency` (Choice): Monthly, bi-monthly, on-demand—drives payment calendar views.
- `Average Cost` and `Last Billed Amount` (Currency): Support budgeting and forecasting.
- `Is Payable Account` (Yes/No): When true, exposes payment workflow controls in the app.
- `Notes` (Multiple lines): Captures one-off details like billing quirks or escalation paths.

### Account Providers
**Purpose:** Master list of organizations that issue accounts so billing info stays consistent across residents.
- `Provider Name` (Title): Display name surfaced in lookups.
- `Category` (Choice): Heating, electricity, telecom, county portal, etc., to group providers.
- `Customer Service Phone / Email` (Single line): Contact details for quick follow-up.
- `Billing Portal URL` (Hyperlink): Direct link for staff completing payments online.
- `Default Payable` (Yes/No): Indicates whether accounts from this provider are typically paid by CommonBond.
- `Notes` (Multiple lines): Used for troubleshooting steps, after-hours contacts, or documentation requirements.

### Agencies
**Purpose:** Represents external organizations that partner with CommonBond (VA, county offices, nonprofits).
- `Agency` (Title): Formal organization name.
- `Primary Contact` (Lookup → Service Providers or Person field): Who staff should reach first.
- `Phone`, `Email`, `Address` (Single line): Standard directory details.
- `Programs Offered` (Multiple lines or Choice): Summarizes programs the agency operates (HUD-VASH, SSVF, mental health services).
- `Service Area` (Choice): Helps filter by geography or specialization.
- `Notes` (Multiple lines): Captures relationship history, MOUs, or referral criteria.

### Archived Records
**Purpose:** Retains historical snapshots for referrals and residents who have exited, keeping analytics and audit history intact.
- `Record Name` (Title): Usually combines participant name and archive date.
- `Record Type` (Choice): Referral, Applicant, Resident—drives downstream automations.
- `Linked Referral` / `Linked Resident` (Lookup): Points back to the original list item for context.
- `Unit` (Lookup → Units): Preserves which unit was associated at the time of archive.
- `Archive Date` (Date/Time): Set during the Move-Out or Close Referral workflow.
- `Archive Reason` (Choice): Declined, Housed, No Contact, Safety Exit, etc.
- `Outcome Notes` (Multiple lines): Free-form narrative on why the record was archived.

### Current Residents
**Purpose:** Active tenancy roster with personal, lease, and benefits information.
- `Resident` (Title): Household head name used across the app.
- `Unit` (Lookup → Units): Keeps resident and housing inventory in sync.
- `Move In Date` / `Expected Move Out` (Date/Time): Drives vacancy planning.
- `Household Size` (Number) and `Household Members` (Lookup → future list): Tracks composition for service planning.
- `Income Source`, `Benefits`, `Veteran Status` (Choice or Lookup): Key eligibility and reporting data.
- `Case Manager` (Person): Primary staff contact.
- `Status` (Choice): Active, Pending Move-Out, Transferred—surfaces in dashboards.
- `Notes` (Multiple lines): Additional context or case considerations.

### Referrals & Applicants
**Purpose:** Intake pipeline for households seeking placement, spanning phone screening through approval.
- `Referral` (Title): Household head or case identifier.
- `Unit Referred` (Lookup → Units): Determines eligibility requirements shown in workflows.
- `Referral Source` (Choice): HUD-VASH, Hennepin Coordinated Entry, internal transfer, etc.
- `Date Referred` / `Date Referral Requested` (Date/Time): Track throughput and SLA compliance.
- `Referral Status` (Choice): New, Phone Screening Complete, Intake Scheduled, Approved, Closed.
- `Phone Screening Completed`, `Intake Completed` (Yes/No): Trigger follow-up tasks or hide workflow buttons.
- `Documentation Checklist` (Multiple choice or attachments): Tracks progress toward eligibility verification.
- `Associated Providers` (Lookup → Service Providers, multi-select): Ensures partners remain visible as referral progresses.
- `Closure Reason` and `Closure Notes` (Multiple lines): Populate when workflow closes the referral.

### Service Providers
**Purpose:** Directory of individual contacts who work with residents or applicants.
- `Service Provider` (Title): Person’s name.
- `Agency` (Lookup → Agencies): Ties the individual to the organization list.
- `Program / Role` (Choice): Case manager, navigator, mental health clinician, etc.
- `Phone`, `Email`, `Office Location` (Single line): Communication basics.
- `Service Areas` (Choice, multi-select): Populations or services they support (employment, benefits, transportation).
- `Active` (Yes/No): Used to filter out outdated contacts without losing history.
- `Notes` (Multiple lines): Relationship details, preferred contact methods, or coverage days.

### Unit Types
**Purpose:** Defines program requirements that apply to units of a given type.
- `Unit Type` (Title): Friendly label shown in color coding (LTH Housing Support, Section 8, CoC).
- `Program` (Choice): Funding stream or regulatory program (HUD-VASH, LDHP, etc.).
- `Referral Source` (Choice): Which coordinated entry or agency provides referrals.
- `Income Limit` (Currency or Number) and `Income Basis` (Choice): Calculation rules for eligibility.
- `Homelessness Requirement` (Yes/No) and `Documentation Required` (Multiple lines): Summarize HUD or county criteria.
- `Disability Requirement` (Choice): Indicates if disability verification is mandatory.
- `Max Occupancy` (Number): Supports household size validation.
- `Notes` (Multiple lines): Guidance for staff about nuances or recent policy updates.

### Units
**Purpose:** Inventory of physical units, their attributes, and current availability.
- `Unit` (Title): Unit number or identifier.
- `Building` / `Floor` (Single line): Helps staff orient themselves on site.
- `Bedrooms`, `Bathrooms` (Number) and `ADA Accessible` (Yes/No): Core physical characteristics.
- `Unit Type` (Lookup → Unit Types): Pulls in program requirements and color coding.
- `Subsidy Administrator` (Choice or Lookup): Owner of subsidy contract.
- `Referral Source` (Choice): Confirms where to request new referrals.
- `Current Resident` (Lookup → Current Residents): Auto-populated when a resident moves in.
- `Vacancy Status` (Choice) and `Show Vacancy` (Yes/No): Drive the vacancy toggle in the Units view.
- `Date Referral Requested`, `Last Referral Outcome`, `Expected Availability` (Date/Time): Support pipeline planning.
- `Notes` (Multiple lines): Space for maintenance, accessibility accommodations, or exceptions.

### How These Lists Work Together
- Unit availability drives referrals: when `Units` indicate a vacancy, staff submit referral requests and create new `Referrals & Applicants` records tied back to the unit and its type.
- Approved referrals become residents: the planned `Create Resident` workflow will clone relevant referral data into `Current Residents`, archive the referral, and update `Units` so vacancy flags clear.
- Services follow the person: `Service Providers` associated with a referral automatically stay linked when the person becomes a resident, ensuring continuity of care documentation.
- Financial operations rely on shared metadata: `Account Information` inherits provider details from `Account Providers`, while unit eligibility rules pull directly from `Unit Types`.

Validate the above definitions against live SharePoint list settings before making structural changes. Use SharePoint list settings or schema exports to confirm internal column names—particularly if automations or Power Apps formulas depend on them.

---
