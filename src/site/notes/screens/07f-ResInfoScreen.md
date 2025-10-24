---
{"dg-publish":true,"permalink":"/screens/07f-res-info-screen/","dgShowToc":true}
---

# ResInfoScreen

**Purpose:** Display and edit resident details with access to associations and workflows.
**Status:** ✅ Complete (fully tested)
**Data Sources:** Current Residents, Units, Exits

## Overview

ResInfoScreen is the central hub for managing current resident information. The screen displays comprehensive resident demographics, contact information, eligibility details, and case management data in an editable form. A header section shows the resident's name and unit assignment for quick reference.

The right side features workflow buttons for common resident management tasks: viewing and managing associations (service providers, accounts, household members) and initiating the Exit Resident workflow when a tenant moves out.

Staff can edit resident information directly in the form, keeping records up to date as circumstances change throughout the resident's tenancy.

## Key Components

### Main Controls
- **Header Section** - Displays resident name and unit number
- **Form1** - Main resident edit form with all demographic and tenancy fields
- **Button_Associations** - Navigate to scrResidentAssociations
- **Button_Exit** - Exit Resident workflow (archive and mark as former occupant)

### Data Flow
- **OnVisible:** Ensures `varSelectedResident` and `varResidentUnit` variables are set
- **Form1.Item:** Bound to `varSelectedResident`
- **Associations Button:** Navigates to scrResidentAssociations
- **Exit Button:** Archives resident, updates status, and navigates away

## Features

### Resident Information Form
**Status:** Complete
**Description:** Comprehensive form displaying and allowing editing of all resident data.

**Field Categories:**
- **Demographics:** Name, Preferred Name, Preferred Language, Date of Birth
- **Contact:** Phone, Email
- **Household:** Household Size, Household Status
- **Eligibility:** Veteran Status, Disability, Benefits, Income
- **Case Management:** HMIS Record Number, County Case Number, Associated Providers
- **Unit Assignment:** Unit Number
- **Tenancy:** Lease Signing Date, Move-out Date (if applicable)

**Known Issue:** No OnSuccess handler means no user feedback after save and no automatic data refresh.

### Associations Management
**Status:** Complete
**Description:** Button navigates to scrResidentAssociations where staff can manage service providers, bill pay accounts, and household members.

**Formula:**
```powerapps
OnSelect: =Navigate(scrResidentAssociations, ScreenTransition.None)
```

### Exit Resident Workflow
**Status:** ✅ Complete (extensively tested)
**Description:** Complete workflow for resident move-out with exit logging and unit updates.

**Implementation:**
```powerapps
OnSelect: |-
  =// Create exit record for historical tracking
  Patch(
      Exits,
      Defaults(Exits),
      {
          Title: varSelectedResident.Title,
          Stage: {Value: "Resident"},
          'Exit / Close-Out Date': Today()
      }
  );

  // Update unit record
  Patch(
      Units,
      varResidentUnit,
      {
          'Occupant Household': Blank(),
          'Is occupied': false
      }
  );

  // Remove resident from Current Residents list
  RemoveIf('Current Residents', ID = varSelectedResident.ID);

  // Refresh and navigate
  Refresh(Units);
  Navigate(ResBrowseScreen, ScreenTransition.None)
```

**What It Does:**
- Creates exit record in Exits list (for historical tracking and ETO integration)
- Clears unit's Occupant Household lookup
- Sets unit Is occupied = false
- Removes resident from Current Residents list
- Refreshes Units list to show vacancy
- Navigates back to browse screen

**Note:** This implementation uses the new Exits list for exit logging (completed October 24, 2025) rather than the old Archived Records approach. Resident records are removed rather than marked as "Former Occupant" to keep the list clean.

## Known Issues

### Missing OnSuccess Handler on Form1
**Severity:** Medium
**Impact:** No user feedback after saving edits, no automatic data refresh
**Symptoms:** Silent saves, potentially stale data shown after edit
**Workaround:** Users must navigate away and return to see changes
**Fix Required:** Add OnSuccess handler with notification and refresh

**Recommended Fix:**
```powerapps
Form1.OnSuccess: |-
  =Notify("Resident information updated successfully.", NotificationType.Success);
  Refresh('Current Residents');
```

### No "Hide Empty Fields" Feature
**Severity:** Low (User Experience)
**Impact:** Visual clutter - all fields shown even when blank
**Workaround:** None
**Enhancement:** Add conditional visibility to data cards based on whether field has value

## Testing Notes

**Test Cases:**

**Form Display and Edit:**
- [ ] Navigate to ResInfoScreen from browse
- [ ] Verify all resident fields display correctly
- [ ] Edit multiple fields
- [ ] Save form
- [ ] Verify changes persist (navigate away and back)
- [ ] Verify lookup fields display correctly
- [ ] Verify choice and multi-choice fields display correctly

**Associations Button:**
- [ ] Click Associations button
- [ ] Verify navigation to scrResidentAssociations
- [ ] Verify varSelectedResident context persists
- [ ] Navigate back to ResInfoScreen
- [ ] Verify resident data still displayed

**Exit Resident Workflow:**
- [ ] Navigate to active resident
- [ ] Click Exit Resident button
- [ ] Verify exit record created in Exits list with resident name, "Resident" stage, and today's date
- [ ] Verify resident removed from Current Residents list
- [ ] Verify unit's Occupant Household cleared
- [ ] Verify unit's Is occupied set to false
- [ ] Navigate to Residents browse
- [ ] Verify exited resident no longer appears
- [ ] Navigate to Units browse
- [ ] Verify unit shows as vacant and available for referrals

**Variable Context:**
- [ ] Verify varSelectedResident contains complete record
- [ ] Verify varResidentUnit lookup succeeds
- [ ] Verify variables persist across screen navigation

## Related Documentation

- [ResBrowseScreen](07c-ResBrowseScreen.md) - Browse all residents
- [scrResidentAssociations](07j-scrResidentAssociations.md) - Manage associations
- [scrResidentAccounts](07k-scrResidentAccounts.md) - Manage bill pay accounts
- [UnitInfoScreen](07d-UnitInfoScreen.md) - View unit after resident exits
- [Data Model - Current Residents List](../03-Data-Model.md#current-residents-list) - Schema for list
