---
{"dg-publish":true,"permalink":"/screens/07a-unit-browse-screen/","dgShowToc":true}
---

# UnitBrowseScreen

**Purpose:** Display all housing units with occupancy status and vacancy filtering.
**Status:** Complete
**Data Sources:** Units, Unit Types

## Overview

UnitBrowseScreen provides a comprehensive view of all housing units in the Upper Post Clearview portfolio. The screen displays units in a three-column grid layout with color coding based on unit type, making it easy to visually distinguish between different funding sources (Section 8 PBV, LTH Housing Support, CoC Rental Assistance, and HUD-VASH PBV).

The screen includes a "Show Vacancies" toggle filter that allows users to quickly focus on available units. This is particularly useful during referral placement when staff need to identify which units can receive new referrals. Unit cards display key information including unit number, type, and current occupancy status.

When a user selects a unit, the app sets the `varActiveUnit` global variable and navigates to UnitInfoScreen where detailed unit information and workflow actions (like "Receive Referral") are available.

## Key Components

### Main Controls
- **UnitBrowseGallery** - Primary gallery control displaying unit cards in 3-column grid layout
- **Toggle1** - "Show Vacancies" checkbox filter
- **Header Rectangle** - Blue header bar with "Units" title
- **Back Navigation** - Icon for returning to previous screen

### Data Flow
- **OnVisible:** Loads `colStatusPalette` collection with color coding for unit types
- **Gallery Items:** Filters and displays units based on toggle state
- **Gallery OnSelect:** Sets `varActiveUnit` variable and navigates to UnitInfoScreen
- **Color Coding:** Gallery template fill uses LookUp against colStatusPalette to color-code by unit type

## Features

### Unit Gallery Display
**Status:** Complete
**Description:** Displays all units with essential information including unit number, type, and occupancy status. Uses responsive 3-column grid layout.

### Vacancy Filtering
**Status:** Complete
**Description:** Toggle filter that shows only vacant units (where `Is occupied` = false OR `Has given notice` = true)

**Formula:**
```powerapps
Items: =If(
    Toggle1.Value,  // Show vacancies only
    Filter(
        Units,
        Or(
            'Is occupied' = false,
            'Has given notice' = true
        )
    ),
    Units  // Show all
)
```

### Color-Coded Unit Types
**Status:** Complete
**Description:** Units are color-coded by funding type for quick visual identification:
- Section 8 PBV: Light blue (#DDEBF7)
- LTH Housing Support: Light green (#C6EFCE)
- CoC Rental Assistance: Light yellow (#FFF2CC)
- HUD-VASH PBV: Light purple (#E5DFEC)

**Formula:**
```powerapps
TemplateFill: =LookUp(colStatusPalette, key = Lower(ThisItem.'Unit Type'.Value), color)
```

### Unit Selection Navigation
**Status:** Complete
**Description:** Clicking any unit card sets the selected unit as context and navigates to the detailed unit information screen.

**Formula:**
```powerapps
OnSelect: |-
  =Set(varActiveUnit, ThisItem);
  Navigate(UnitInfoScreen, ScreenTransition.None)
```

## Known Issues

None currently. Screen is fully functional and ready for production use.

## Testing Notes

**Test Cases:**
- Verify all units display in gallery when toggle is off
- Verify only vacant units display when "Show Vacancies" is toggled on
- Verify units with "Has given notice" = true appear in vacancy filter
- Verify color coding matches unit types correctly
- Verify clicking unit navigates to UnitInfoScreen
- Verify varActiveUnit is set correctly on selection

**Performance Considerations:**
- Gallery uses inline LookUp for unit type colors (acceptable for datasets under 100 units)
- For larger datasets, consider pre-loading unit type color mappings in a collection

## Related Documentation

- [UnitInfoScreen](07d-UnitInfoScreen.md) - Detail view for individual units
- [Data Model - Units List](../03-Data-Model.md#units-list) - Schema for Units list
- [Receive Referral Workflow](../workflows/receive-referral.md) - Primary workflow initiated from unit view
