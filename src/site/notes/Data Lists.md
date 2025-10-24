---
{"dg-publish":true,"permalink":"/data-lists/","dgShowToc":true}
---

# Data Lists

**SharePoint Lists - Schema & Relationships**
**Last Updated:** October 24, 2025

All UP Clearview data is stored in SharePoint Online lists. This document describes each list's purpose, key fields, and relationships.

---

## Core Data Lists

### Units
**Purpose:** Housing unit inventory with eligibility requirements and current status

**Key Fields:**
- **Title** (Text) - Unit number/identifier
- **Unit Type** (Lookup to Unit Types) - Links to eligibility template
- **Is occupied** (Yes/No) - Current occupancy status
- **Has given notice** (Yes/No) - Resident planning to move out
- **Has referral** (Yes/No) - Referral assigned to this unit
- **Has move-in scheduled** (Yes/No) - Move-in date set
- **Is offline** (Yes/No) - Unit temporarily unavailable
- **Occupant Household** (Lookup to Current Residents) - Current resident
- **Referral Household** (Lookup to Referrals & Applicants) - Assigned referral
- **Referral Requested** (Date) - When referral was requested
- **Date Available** (Date) - When unit became/will become available
- **HMIS Program ID** (Text) - External system identifier
- **Referral Source** (Text) - Where referrals come from for this unit
- **Administrator** (Text) - Responsible party

**Relationships:**
- → Current Residents (one-to-one via Occupant Household)
- → Referrals & Applicants (one-to-one via Referral Household)
- → Unit Types (many-to-one)

**Used By:** UnitBrowseScreen, UnitInfoScreen, all workflows

---

### Current Residents
**Purpose:** Active resident records with demographics and service connections

**Key Fields:**
- **Title** (Text) - Resident name (Last, First or First Last)
- **Household Status** (Choice) - Current status (e.g., "Occupant, Current", "Former Occupant")
- **Unit Number** (Lookup to Units) - Assigned unit
- **Associated Providers** (Lookup to Service Providers, multi-select) - Connected service providers
- **Lease Signing Date** (Date) - Move-in date
- **Expected Move Out** (Date) - Planned move-out date
- **Preferred Name** (Text)
- **Preferred Language** (Text)
- **Date of Birth** (Date)
- **Phone** (Text)
- **Email** (Text)
- **Veteran Status** (Choice) - e.g., "Veteran", "Not a Veteran"
- **Disability** (Choice) - Yes/No or type
- **Benefits** (Text) - SSI, SSDI, etc.
- **Other Source of Income** (Text)
- **Monthly Income** (Number/Currency)
- **HMIS Record Number** (Text)
- **County Case Number** (Text)
- **Household Size** (Number)

**Relationships:**
- → Units (many-to-one via Unit Number)
- ← Units (one-to-one reverse via Occupant Household)
- → Service Providers (many-to-many via Associated Providers)
- → Account Information (one-to-many)
- → Household Members (one-to-many) - *Planned*

**Used By:** ResBrowseScreen, ResInfoScreen, scrResidentAssociations, exit workflow

---

### Referrals & Applicants
**Purpose:** Referral and applicant pipeline tracking

**Key Fields:**
- **Title** (Text) - Applicant/referral name
- **Household Status** (Choice) - Current stage (e.g., "Referral, Received", "Applicant, Approved", "Former Referral")
- **Unit Number** (Lookup to Units) - Assigned unit
- **Date Referred** (Date) - When referral received
- **Intake Completed** (Date) - When intake was completed
- **Phone** (Text)
- **Email** (Text)
- **Preferred Name** (Text)
- **Preferred Language** (Text)
- **Date of Birth** (Date)
- **Veteran Status** (Choice)
- **Disability** (Choice)
- **Benefits** (Text)
- **Other Source of Income** (Text)
- **Monthly Income** (Number/Currency)
- **HMIS Record Number** (Text)
- **County Case Number** (Text)
- **Household Size** (Number)

**Relationships:**
- → Units (many-to-one via Unit Number)
- ← Units (one-to-one reverse via Referral Household)

**Used By:** RNABrowseScreen, RNAInfoScreen, IntakeScreen, scrConvert, RNACloseScreen

**Workflow Transitions:**
- Receive Referral → Creates record
- Phone Screening → Updates record
- Intake → Updates record, advances status
- Convert to Resident → Transfers to Current Residents, deletes referral
- Close-Out → Archives to Exits, deletes referral

---

### Unit Types
**Purpose:** Eligibility requirement templates for different unit types

**Key Fields:**
- **Title** (Text) - Unit type name (e.g., "LTH Housing Support", "Ramsey CoC")
- **Income Restriction** (Choice or Text) - e.g., "30% AMI", "50% AMI"
- **Maximum Income Limit** (Number/Currency) - Dollar amount
- **Occupancy Requirement** (Text) - Who can occupy
- **Homelessness Requirement** (Yes/No or Choice) - Must be experiencing homelessness?
- **Disability Requirement** (Choice) - e.g., "None", "Physical", "Mental Health", "Either"
- **Disability Documentation** (Text) - What documentation required
- **Specific Eligibility Requirements** (Multi-line Text) - Additional details

**Relationships:**
- ← Units (one-to-many)

**Used By:** IntakeScreen (displays requirements), UnitInfoScreen (eligibility section)

---

### Exits
**Purpose:** Exit and close-out logging for historical tracking and reporting

**Key Fields:**
- **Title** (Text) - Person's name
- **Stage** (Choice) - "Resident", "Applicant", or "Referral"
- **Exit / Close-Out Date** (Date) - When they exited/closed out
- **Close-Out Reason** (Choice) - Reason for exit (optional, populated for referrals)
- **Notes** (Multi-line Text) - Additional details (optional)

**Relationships:** None (archive only)

**Created By:**
- Exit Resident workflow (ResInfoScreen)
- Close-Out Referral workflow (RNACloseScreen)

**Used For:**
- Historical reporting
- Turnover analysis
- Future ETO integration
- Audit trails

**Status:** ✅ Fully implemented (October 24, 2025)

---

## Service Provider & Organization Lists

### Service Providers
**Purpose:** Case manager and service provider directory

**Key Fields:**
- **Title** (Text) - Provider name
- **Role/Title** (Text) - Job title or role
- **Organization** (Lookup to Agencies) - Affiliated organization
- **Program** (Text) - Program or department
- **Phone** (Text)
- **Email** (Text)
- **Is Active** (Yes/No) - Currently active provider?

**Relationships:**
- → Agencies (many-to-one via Organization)
- ← Current Residents (many-to-many reverse via Associated Providers)

**Used By:** scrServiceProviders, scrResidentAssociations

---

### Agencies
**Purpose:** Organizations providing services

**Key Fields:**
- **Title** (Text) - Organization name
- **Contact Person** (Text)
- **Phone** (Text)
- **Email** (Text)
- **Services Provided** (Multi-line Text) - Description of services
- **Service Providers** (Lookup to Service Providers, multi-select) - Staff from this organization

**Relationships:**
- ← Service Providers (one-to-many)

**Used By:** OrgScreen, scrServiceProviders

---

### Account Information
**Purpose:** Bill pay tracking for Housing Support program

**Key Fields:**
- **Title** (Text) - Account name/description
- **Associated Resident** (Lookup to Current Residents) - Who this account is for
- **Account Provider** (Choice) - e.g., "Excel Energy", "CenterPoint", "Xcel Energy"
- **Username** (Text) - Login username
- **Account Number** (Text) - Provider account number
- **Is Payable Account** (Yes/No) - Is this a bill-pay account?
- **Billing Frequency** (Choice) - e.g., "Monthly", "Quarterly"
- **Recurring Cost** (Number/Currency) - Expected monthly/regular charge
- **Is Active** (Yes/No) - Currently active account?

**Relationships:**
- → Current Residents (many-to-one via Associated Resident)

**Used By:** scrResidentAccounts, scrResidentAssociations

**Security Note:** Password field was removed for security reasons. Users should use browser-built password managers.

---

### Household Members
**Purpose:** Track household composition (family members)

**Status:** ⚠️ Planned, not yet implemented

**Proposed Fields:**
- **Title** (Text) - Member name
- **Associated Resident** (Lookup to Current Residents) - Head of household
- **Relationship** (Choice) - e.g., "Spouse", "Child", "Parent", "Other"
- **Date of Birth** (Date)
- **Notes** (Multi-line Text) - Additional information

**Relationships:**
- → Current Residents (many-to-one via Associated Resident)

**Will Be Used By:** scrResidentAssociations (household members section)

---

## Lookup Relationships Summary

### Bidirectional Lookups (Require Manual Maintenance)

**Units ↔ Current Residents:**
- Units.'Occupant Household' → Current Residents
- Current Residents.'Unit Number' → Units
- **Note:** Power App workflows maintain both sides of relationship

**Units ↔ Referrals & Applicants:**
- Units.'Referral Household' → Referrals & Applicants
- Referrals & Applicants.'Unit Number' → Units
- **Note:** Power App workflows maintain both sides of relationship

### One-Way Lookups

**Multiple lists → Unit Types:**
- Units.'Unit Type' → Unit Types
- Simple lookup, no reverse relationship

**Current Residents → Service Providers (Multi-Select):**
- Current Residents.'Associated Providers' → Service Providers (multiple)
- No reverse lookup

**Service Providers → Agencies:**
- Service Providers.'Organization' → Agencies
- Agencies.'Service Providers' is reverse multi-select (manually maintained)

**Account Information → Current Residents:**
- Account Information.'Associated Resident' → Current Residents
- No reverse lookup (can filter accounts by resident)

---

## Field Type Notes

### Choice Fields
Choice fields in SharePoint require special handling in Power Apps:
```powerapps
// Setting a choice field
Patch(List, Item, {
    ChoiceField: {
        '@odata.type': "#Microsoft.Azure.Connectors.SharePoint.SPListExpandedReference",
        Value: "Choice Option Text"
    }
})

// Reading a choice field
Item.ChoiceField.Value  // Returns the text
```

### Lookup Fields
Lookup fields store objects with Id and Value:
```powerapps
// Setting a lookup field
Patch(List, Item, {
    LookupField: LookupItem  // Pass the entire record
})

// Or with explicit object
Patch(List, Item, {
    LookupField: {
        '@odata.type': "#Microsoft.Azure.Connectors.SharePoint.SPListExpandedReference",
        Id: LookupItem.ID,
        Value: LookupItem.Title
    }
})

// Reading a lookup field
Item.LookupField.Value  // Returns display value (usually Title)
Item.LookupField.Id     // Returns ID for further lookups
```

### Multi-Select Lookups
Multi-select lookup fields store arrays:
```powerapps
// Setting multi-select (e.g., Associated Providers)
Patch('Current Residents', Resident, {
    'Associated Providers': ComboBox1.SelectedItems  // Array of records
})

// Reading multi-select
ForAll(Resident.'Associated Providers', Value)  // Returns collection of names
```

---

**Related Documentation:**
- [[Screen Status\|Screen Status]] - Which screens use which lists
- [[Workflow Status\|Workflow Status]] - How workflows interact with data
- [[UP Clearview – Technical Documentation\|UP Clearview – Technical Documentation]] - Code patterns and examples
- [[UP Clearview - Implementation Guide\|UP Clearview - Implementation Guide]] - Setting up lists for new sites
