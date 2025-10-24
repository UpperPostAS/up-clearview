---
{"dg-publish":true,"permalink":"/screens/07d-unit-info-screen/","dgShowToc":true}
---

# UnitInfoScreen

**Purpose:** Display detailed unit information and handle "Receive Referral" workflow.
**Status:** Complete
**Data Sources:** Units, Unit Types, Referrals & Applicants

## Overview

UnitInfoScreen provides a comprehensive detail view for individual housing units, split into two main panels. The left panel displays static unit information including physical details, eligibility requirements, and subsidy information. The right panel shows the current unit status and allows staff to edit certain fields like referral status.

A key feature is the conditional "Receive Referral" button, which appears only when a unit is vacant and doesn't already have a pending referral. This button opens a modal form allowing staff to create a new referral record directly associated with the unit.

The screen uses a FormViewer control for the static information (non-editable) and a standard Form control for editable fields. This design pattern keeps unit configuration stable while allowing operational fields to be updated as needed.

## Key Components

### Main Controls
- **FormViewer1** - Displays static unit details (non-editable)
  - Unit number, bedrooms, bathrooms
  - Unit type, HMIS ID
  - Subsidy administrator
  - Eligibility requirements (income, homelessness, disability)
- **Form1** - Editable unit status form
  - Current occupant or referral information
  - Occupancy flags
  - Referral dates
- **Container3** - Modal container for "Receive Referral" form
- **Button2_2** - "Receive Referral" button (conditional visibility)
- **ReferralForm** - Form inside modal for creating new referrals

### Data Flow
- **OnVisible:** Sets `varUnitType` by looking up the unit's type in Unit Types list (for displaying eligibility requirements)
- **Receive Referral Button OnSelect:** Sets `varShowReceiveForm = true` to display modal
- **ReferralForm OnSuccess:** Creates referral, updates unit's Referral Household, sets Has referral = true, refreshes data, closes modal

## Features

### Static Unit Information Display
**Status:** Complete
**Description:** FormViewer shows all unit configuration details in a read-only format, preventing accidental changes to unit setup.

**Fields Displayed:**
- Unit (number)
- Bedrooms
- Bathrooms
- Unit Type
- HMIS ID
- Subsidy Administrator
- Minimum Occupancy
- Maximum Occupancy
- Income Restriction (percentage)
- Income Guideline
- Homelessness Requirement
- Disability Documentation Required

### Editable Unit Status
**Status:** Complete
**Description:** Form1 allows editing of operational fields related to current occupancy and referral status.

**Known Issue:** No OnSuccess handler, so no user feedback after save and no automatic data refresh.

**Recommended Enhancement:**
```powerapps
Form1.OnSuccess: |-
  =Notify("Unit information updated successfully.", NotificationType.Success);
  Refresh(Units);
```

### Receive Referral Workflow
**Status:** Complete
**Description:** Modal form workflow for creating a new referral record associated with the unit.

**Button Visibility Logic:**
```powerapps
Visible: =And(
    varActiveUnit.'Is occupied' = false,
    IsBlank(varActiveUnit.'Referral Household')
)
```
Shows button only when unit is vacant and has no existing referral.

**OnSuccess Handler:**
```powerapps
OnSuccess: |-
  =Patch(
      Units,
      varActiveUnit,
      {
          'Referral Household': {
              '@odata.type': "#Microsoft.Azure.Connectors.SharePoint.SPListExpandedReference",
              Id: ReferralForm.LastSubmit.ID,
              Value: ReferralForm.LastSubmit.Title
          },
          'Has referral': true
      }
  );
  Set(varShowReceiveForm, false);
  Refresh(Units);
  Refresh('Referrals & Applicants');
```

**Design Notes:**
- Uses `ReferralForm.LastSubmit` to get the newly created referral's ID and Title
- Updates unit record to link the referral (establishes bidirectional relationship)
- Refreshes both data sources to ensure UI reflects changes
- Closes modal by setting varShowReceiveForm to false

### Eligibility Requirements Display
**Status:** Complete
**Description:** Uses lookup to Unit Types list to display eligibility requirements specific to the unit's funding type.

**OnVisible Logic:**
```powerapps
OnVisible: |-
  =Set(varUnitType, LookUp('Unit Types', 'Unit Type' = varActiveUnit.'Unit Type'.Value));
```

## Known Issues

### Missing OnSuccess Handler on Form1
**Severity:** Medium
**Impact:** No user feedback after saving edits, no automatic data refresh
**Workaround:** Users must manually navigate away and return to see changes
**Fix Required:** Add OnSuccess handler with Notify() and Refresh()

## Testing Notes

**Test Cases:**

**Receive Referral Workflow:**
- [ ] Navigate to vacant unit without referral
- [ ] Verify "Receive Referral" button is visible
- [ ] Click button and verify modal opens
- [ ] Verify Unit Number pre-fills in referral form
- [ ] Verify Date Referred defaults to today
- [ ] Enter referral head of household name
- [ ] Click Save
- [ ] Verify success (referral created)
- [ ] Verify unit's Referral Household updated
- [ ] Verify unit's Has referral = true
- [ ] Verify modal closes
- [ ] Navigate to Referrals browse
- [ ] Verify new referral appears

**Button Visibility:**
- [ ] Occupied unit: Button should NOT appear
- [ ] Vacant unit with existing referral: Button should NOT appear
- [ ] Vacant unit with no referral: Button SHOULD appear

**Data Display:**
- [ ] Verify all static unit fields display correctly
- [ ] Verify eligibility requirements match unit type
- [ ] Verify current occupant displays if unit is occupied
- [ ] Verify referral household displays if unit has referral

## Related Documentation

- [UnitBrowseScreen](07a-UnitBrowseScreen.md) - Browse all units
- [RNAInfoScreen](07e-RNAInfoScreen.md) - View referral details after creation
- [Receive Referral Workflow](../workflows/receive-referral.md) - Complete workflow documentation
- [Data Model - Units List](../03-Data-Model.md#units-list) - Schema for Units list
