---
{"dg-publish":true,"permalink":"/workflows/08e-exit-resident/","dgShowToc":true}
---

# Exit Resident

**Purpose:** Document a resident move-out and archive their record when they exit housing
**Status:** ✅ Complete and tested
**Trigger:** User clicks "Exit Resident" button on ResInfoScreen
**Data Sources:** Current Residents (updates), Units (updates), Archived Records (creates)

## Overview

The Exit Resident workflow documents the conclusion of a resident's tenancy at Upper Post Clearview. When a resident moves out (whether due to successful housing stability, program completion, eviction, or voluntary departure), staff use this workflow to properly archive the resident's information and update the unit status.

This workflow ensures that the resident's complete record is preserved in the Archived Records list for historical tracking and reporting purposes while updating their status to "Former Occupant" in the Current Residents list. The workflow also updates the associated unit record to reflect the vacancy, setting the unit as unoccupied and marking it as available with the current date.

The exit process maintains data integrity by updating all three related entities (resident status, unit occupancy, and archive record) in a coordinated manner, preventing orphaned records or incorrect vacancy status.

## User Journey

1. **Access Screen:** User views a current resident on ResInfoScreen
2. **Initiate Exit:** User clicks "Exit Resident" workflow button
3. **Create Archive:** System creates archive record with:
   - Link to resident record
   - Archive date (today)
   - Archive reason ("Moved Out")
4. **Update Resident:** System updates resident status to "Former Occupant"
5. **Update Unit:** System updates unit record:
   - Clears Occupant Household
   - Sets Is occupied to false
   - Sets Date Available to today
6. **Refresh Data:** System refreshes Units list to reflect changes
7. **Navigate:** System navigates to ResBrowseScreen
8. **Confirm:** Former resident no longer appears in browse view (filtered out by "former" status)

## Technical Implementation

### Screen(s) Involved
- **ResInfoScreen** - Primary screen with "Exit Resident" button (Button_Exit)
- **ResBrowseScreen** - Destination after exit completion

### Data Operations

**Creates:**
- New record in Archived Records list with:
  - Linked Resident (lookup to resident being archived)
  - Archive Date (today's date)
  - Archive Reason ("Moved Out")
  - Exit logging documentation

**Updates:**
- Current Residents record with:
  - Household Status changed to "Former Occupant"

- Units record with:
  - Occupant Household cleared (set to Blank)
  - Is occupied set to false
  - Date Available set to today

**Deletes:**
- None (resident record preserved with "Former Occupant" status)

### Key Formulas

**Complete Exit Resident Button OnSelect:**
```powerapps
OnSelect: |-
  =// Archive resident
  Patch(
      'Archived Records',
      Defaults('Archived Records'),
      {
          'Linked Resident': varSelectedResident,
          'Archive Date': Today(),
          'Archive Reason': "Moved Out"
      }
  );

  // Update resident status
  Patch(
      'Current Residents',
      varSelectedResident,
      { 'Household Status': { Value: "Former Occupant" } }
  );

  // Update unit record
  Patch(
      Units,
      varResidentUnit,
      {
          'Occupant Household': Blank(),
          'Is occupied': false,
          'Date Available': Today()
      }
  );

  // Refresh and navigate
  Refresh(Units);
  Navigate(ResBrowseScreen, ScreenTransition.None)
```

**Exit Logging Details:**
The exit workflow creates a comprehensive archive record that includes:
- Complete resident demographic information (via lookup)
- Exit date for reporting
- Exit reason for tracking outcomes
- Preserved relationship to original resident record for historical queries

## Testing Procedure

### Prerequisites:
- Current resident record with status "Occupant, Current"
- Resident assigned to a valid unit
- Unit marked as occupied with resident
- User has permissions to update all three lists

### Basic Testing:

1. **Setup:**
   - Create test resident or use existing resident
   - Verify resident status is "Occupant, Current"
   - Verify resident assigned to test unit
   - Note the unit number for later verification
   - Navigate to ResInfoScreen for test resident

2. **Execute:**
   - Verify "Exit Resident" button is visible
   - Click "Exit Resident" button
   - System processes exit (no modal or confirmation currently)

3. **Verify Exit Processing:**
   - Verify navigation to ResBrowseScreen
   - Verify former resident does NOT appear in browse view
   - Navigate to Units browse view
   - Find the unit the resident was in
   - Verify unit shows as vacant (not occupied)

### Detailed Record Verification:

4. **Verify Resident Record:**
   - Access Current Residents SharePoint list directly
   - Find the test resident record (search by name)
   - Verify record still exists (not deleted)
   - Verify Household Status = "Former Occupant"
   - Verify Unit Number still shows (preserved for history)
   - Verify all other demographic data intact

5. **Verify Unit Record:**
   - Navigate to UnitInfoScreen for the unit
   - Verify "Occupant Household" field is blank/empty
   - Verify "Is occupied" shows No/False
   - Verify "Date Available" shows today's date
   - Verify "Receive Referral" button now visible (unit available)

6. **Verify Archive Record:**
   - Access Archived Records SharePoint list directly
   - Find newest archive record
   - Verify "Linked Resident" points to test resident
   - Verify "Archive Date" = today
   - Verify "Archive Reason" = "Moved Out"
   - Verify can access resident details through lookup

### Data Integrity Testing:

7. **Verify Browse View Filters:**
   - Navigate to ResBrowseScreen
   - Confirm test resident does NOT appear
   - Verify filter excludes "Former Occupant" status
   - Check count of residents decreased by 1

8. **Verify Unit Availability:**
   - Navigate to UnitBrowseScreen
   - Toggle "Show Vacancies" ON
   - Verify unit now appears in vacancy list
   - Click unit to view details
   - Verify all occupancy fields cleared

9. **Verify Data Relationships:**
   - From unit, verify no link to former resident as occupant
   - From archive record, verify can access full resident record
   - From resident record, verify still linked to unit (historical)

### Edge Case Testing:

10. **Test Resident with Accounts:**
    - Exit resident who has bill pay accounts
    - Verify accounts still accessible (not deleted)
    - Verify accounts remain associated with former resident

11. **Test Resident with Service Providers:**
    - Exit resident with multiple associated providers
    - Verify provider associations preserved
    - Verify can still view relationship through resident record

12. **Test Multiple Exits:**
    - Exit multiple residents in succession
    - Verify each exit processed correctly
    - Verify no cross-contamination of data

## Known Issues

None currently - workflow tested and fully functional

**Recent Fix:** This workflow was recently updated to include unit record updates. Previous version only updated resident and archive records, leaving units incorrectly marked as occupied.

## Related Documentation

- [ResInfoScreen Documentation](../screens/ResInfoScreen.md)
- [ResBrowseScreen Documentation](../screens/ResBrowseScreen.md)
- [UnitInfoScreen Documentation](../screens/UnitInfoScreen.md)
- [Convert to Resident Workflow](./08d-Convert-to-Resident.md) - Beginning of residency
- [Data Model - Current Residents](../03-Data-Model.md)
- [Data Model - Units](../03-Data-Model.md)
- [Data Model - Archived Records](../03-Data-Model.md)

**Future Enhancements:**
- Add confirmation dialog before exit ("Are you sure?")
- Add exit reason dropdown (voluntary, eviction, death, incarceration, etc.)
- Add exit notes field for detailed documentation
- Add exit date field (separate from archive date for backdating)
- Add notification to resident's service providers
- Add ability to undo exit (within certain timeframe)
- Generate exit documentation/letter automatically
