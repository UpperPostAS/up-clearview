---
{"dg-publish":true,"permalink":"/screens/07b-rna-browse-screen/","dgShowToc":true}
---

# RNABrowseScreen

**Purpose:** Browse all active referrals and applicants (excludes "former" records).
**Status:** Complete
**Data Sources:** Referrals & Applicants, Units

## Overview

RNABrowseScreen displays all active referrals and applicants in a three-column grid gallery. The screen automatically filters out any records with a "Former" status (Former Referral or Former Applicant), ensuring staff only see active cases requiring attention.

Records are sorted by unit number in ascending order, making it easy to see which units have pending referrals or applications. The gallery uses the same color-coding pattern as UnitBrowseScreen, allowing users to quickly identify funding types at a glance.

Each card displays the head of household name, unit number, household status, and other key information. Selecting a card navigates to RNAInfoScreen where staff can view complete referral details and access workflow actions like Phone Screening, Intake, Create Resident, and Close Referral.

## Key Components

### Main Controls
- **RNAGallery** - Primary gallery displaying referral/applicant cards in 3-column grid
- **Header Rectangle** - Blue header bar with screen title
- **Back Navigation** - Icon for returning to previous screen

### Data Flow
- **OnVisible:** Loads `colStatusPalette` collection for unit type color coding
- **Gallery Items:** Filters out "former" records and sorts by unit number
- **Gallery OnSelect:** Sets `varSelectedReferral` and `varReferralUnit` variables, then navigates to RNAInfoScreen
- **Color Coding:** Template fill uses LookUp against unit type for visual identification

## Features

### Active Referrals Gallery
**Status:** Complete
**Description:** Displays all referrals and applicants that are not marked as "Former", providing a focused view of active cases.

**Formula:**
```powerapps
Items: |-
  =Sort(
      Filter(
          'Referrals & Applicants',
          !StartsWith(Lower(Coalesce('Household Status'.Value, "")), "former")
      ),
      'Unit Number'.Value,
      SortOrder.Ascending
  )
```

**Design Notes:**
- Uses `!StartsWith()` for efficient filtering (delegable to SharePoint)
- `Coalesce()` handles null household status values gracefully
- Sorts by unit number value for logical grouping

### Referral Selection Navigation
**Status:** Complete
**Description:** Selecting a referral card sets context variables and navigates to the detailed information screen.

**Formula:**
```powerapps
OnSelect: |-
  =Set(varSelectedReferral, ThisItem);
  Set(varReferralUnit, LookUp(Units, ID = varSelectedReferral.'Unit Number'.Id));
  Set(varFormMode, FormMode.View);
  Navigate(RNAInfoScreen, ScreenTransition.None)
```

**Design Notes:**
- Sets `varSelectedReferral` to the entire record for downstream use
- Pre-loads `varReferralUnit` to avoid repeated lookups on detail screen
- Sets form mode to view (non-edit) by default
- Uses ScreenTransition.None for instant navigation

### Color-Coded Unit Types
**Status:** Complete
**Description:** Cards are color-coded by the unit type associated with the referral, matching the color scheme used throughout the application.

## Known Issues

None currently. Screen is fully functional and ready for production use.

## Testing Notes

**Test Cases:**
- Verify active referrals display in gallery
- Verify "Former Referral" records do not appear
- Verify "Former Applicant" records do not appear
- Verify sorting by unit number works correctly
- Verify records without unit number appear (at beginning or end)
- Verify clicking referral navigates to RNAInfoScreen
- Verify varSelectedReferral is set correctly
- Verify varReferralUnit lookup succeeds for referrals with unit
- Verify varReferralUnit handles referrals without unit gracefully
- Verify color coding matches unit types

**Performance Considerations:**
- Filter with `!StartsWith()` is delegable to SharePoint (efficient for large datasets)
- Sort by lookup field `.Value` is delegable
- Consider data limits if referral list exceeds 2000 records

## Related Documentation

- [RNAInfoScreen](07e-RNAInfoScreen.md) - Detail view for individual referrals/applicants
- [RNACloseScreen](07i-RNACloseScreen.md) - Close referral workflow
- [scrConvert](07g-scrConvert.md) - Convert referral to resident workflow
- [Data Model - Referrals & Applicants List](../03-Data-Model.md#referrals--applicants-list) - Schema for list
