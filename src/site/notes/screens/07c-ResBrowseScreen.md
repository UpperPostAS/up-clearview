---
{"dg-publish":true,"permalink":"/screens/07c-res-browse-screen/","dgShowToc":true}
---

# ResBrowseScreen

**Purpose:** Browse all current residents (excludes former residents).
**Status:** Complete
**Data Sources:** Current Residents, Units

## Overview

ResBrowseScreen displays all current residents in a gallery view, automatically filtering out former residents (exited occupants). This screen provides staff with a comprehensive view of all active residential households across the Upper Post Clearview portfolio.

The gallery uses color coding based on unit type, consistent with the visual design pattern established in UnitBrowseScreen and RNABrowseScreen. Records are sorted by unit number for easy navigation and reference.

Each card displays the resident's name, unit number, household status, and other key information. Selecting a card navigates to ResInfoScreen where staff can view complete resident details, edit information, manage associations with service providers and accounts, and initiate workflows like Exit Resident.

## Key Components

### Main Controls
- **ResidentGallery** - Primary gallery displaying resident cards in grid layout
- **Header Rectangle** - Blue header bar with screen title
- **Back Navigation** - Icon for returning to previous screen

### Data Flow
- **OnVisible:** Loads `colStatusPalette` collection for unit type color coding
- **Gallery Items:** Filters out former residents and sorts by unit number
- **Gallery OnSelect:** Sets `varSelectedResident` and `varResidentUnit` variables, then navigates to ResInfoScreen
- **Color Coding:** Template fill uses LookUp against unit type

## Features

### Current Residents Gallery
**Status:** Complete
**Description:** Displays all residents with active household status, filtering out former occupants.

**Formula:**
```powerapps
Items: |-
  =SortByColumns(
      Filter(
          'Current Residents',
          !StartsWith(Lower(Coalesce('Household Status'.Value, "")), "former")
      ),
      "UnitNumber", Ascending
  )
```

**Design Notes:**
- Uses `SortByColumns()` instead of `Sort()` for better SharePoint delegation
- Sorts by internal field name "UnitNumber" (delegation-friendly)
- `!StartsWith()` filter is delegable to SharePoint
- `Coalesce()` handles null household status gracefully

### Resident Selection Navigation
**Status:** Complete
**Description:** Selecting a resident card sets context variables and navigates to the detailed information screen.

**Formula:**
```powerapps
OnSelect: |-
  =Set(varSelectedResident, ThisItem);
  Set(varResidentUnit, LookUp(Units, ID = varSelectedResident.'Unit Number'.Id));
  Navigate(ResInfoScreen, ScreenTransition.None)
```

**Design Notes:**
- Sets `varSelectedResident` to the entire record
- Pre-loads `varResidentUnit` to avoid repeated lookups on detail screen
- Uses ScreenTransition.None for instant navigation

### Color-Coded Unit Types
**Status:** Complete
**Description:** Cards are color-coded by the unit type where the resident lives, providing visual consistency across all browse screens.

## Known Issues

None currently. Screen is fully functional and ready for production use.

## Testing Notes

**Test Cases:**
- Verify current residents display in gallery
- Verify "Former Occupant" records do not appear
- Verify sorting by unit number works correctly
- Verify residents without unit number appear (at beginning or end)
- Verify clicking resident navigates to ResInfoScreen
- Verify varSelectedResident is set correctly
- Verify varResidentUnit lookup succeeds for residents with unit
- Verify varResidentUnit handles residents without unit gracefully
- Verify color coding matches unit types

**Performance Considerations:**
- `SortByColumns()` is more delegation-friendly than `Sort()`
- Using internal field name "UnitNumber" improves SharePoint delegation
- Filter with `!StartsWith()` is delegable to SharePoint
- Should perform well even with hundreds of residents

## Related Documentation

- [ResInfoScreen](07f-ResInfoScreen.md) - Detail view for individual residents
- [scrResidentAssociations](07j-scrResidentAssociations.md) - Manage service providers, accounts, and household members
- [scrResidentAccounts](07k-scrResidentAccounts.md) - Manage bill pay accounts
- [Data Model - Current Residents List](../03-Data-Model.md#current-residents-list) - Schema for list
