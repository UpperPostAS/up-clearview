---
{"dg-publish":true,"permalink":"/workflows/08f-close-out-referral/","dgShowToc":true}
---

# Close-Out Referral

**Purpose:** Document the closure of a referral that will not result in housing (declined, withdrew, ineligible, etc.)
**Status:** ✅ Complete and tested
**Trigger:** User clicks "Close Referral" button on RNAInfoScreen
**Data Sources:** Referrals & Applicants (updates), Units (updates)

## Overview

The Close-Out Referral workflow handles situations where a referral does not progress to housing placement. This occurs when a referral declines the housing opportunity, withdraws from the process, is determined to be ineligible, accepts housing elsewhere, or becomes unreachable. Properly closing out these referrals is essential for maintaining accurate vacancy data and preventing referrals from remaining indefinitely in "active" status.

The workflow captures the reason for closure and relevant notes, then systematically updates both the referral record (adding "Former" prefix to status and clearing unit assignment) and the associated unit record (clearing the referral household and making the unit available for a new referral). This ensures data integrity and allows the unit to immediately receive a new referral if needed.

The close-out process uses intelligent status logic to determine the appropriate final status ("Former Referral" vs "Former Applicant") based on how far the individual progressed through the intake process before closure.

## User Journey

1. **Access Screen:** User views a referral or applicant on RNAInfoScreen
2. **Initiate Close-Out:** User clicks "Close Referral" workflow button
3. **Navigate to Screen:** System navigates to RNACloseScreen
4. **View Form:** Form displays with three data entry fields
5. **Enter Close-Out Date:** User selects date (defaults to today)
6. **Select Close-Out Reason:** User chooses from dropdown:
   - Declined offer
   - Withdrew application
   - Determined ineligible
   - Accepted housing elsewhere
   - Unreachable/No contact
   - Other
7. **Enter Notes:** User enters detailed explanation in notes field
8. **Submit:** User clicks Save button
9. **Process:** System executes multi-step operation:
   - Captures form submission
   - Calculates new status ("Former Referral" or "Former Applicant")
   - Updates referral status and clears unit assignment
   - Updates unit record (clears referral household, sets Has referral to false)
   - Refreshes data
10. **Navigate:** System returns to RNABrowseScreen
11. **Confirm:** Closed referral no longer appears in browse view (filtered out by "former" status)

## Technical Implementation

### Screen(s) Involved
- **RNAInfoScreen** - Starting point with "Close Referral" button (Button4)
- **RNACloseScreen** - Close-out documentation form
- **RNABrowseScreen** - Destination after close-out completion

### Data Operations

**Creates:**
- None directly (future enhancement may create close-out log records)

**Updates:**
- Referrals & Applicants record with:
  - Close-out Date (user-selected date)
  - Close-out Reason (from dropdown)
  - Close-out Notes (user-entered text)
  - Household Status (updated to "Former Referral" or "Former Applicant")
  - Unit Number (cleared/set to Blank)

- Units record with:
  - Referral Household (cleared/set to Blank)
  - Has referral (set to false)

**Deletes:**
- None (referral record preserved with "Former" status for reporting)

### Key Formulas

**Navigation from RNAInfoScreen:**
```powerapps
OnSelect: |-
  =Set(varCloseRec, varSelectedReferral);
  Navigate(RNACloseScreen, ScreenTransition.None)
```

**Complete OnSuccess Handler:**
```powerapps
OnSuccess: |-
  =Set(varClosedReferral, EditForm1.LastSubmit);

  // Calculate new status with "Former" prefix
  Set(
      varClosedStatus,
      With(
          { currentStatus: Lower(Coalesce(varClosedReferral.'Household Status'.Value, "")) },
          If(
              StartsWith(currentStatus, "referral"),
              "Former Referral",
              If(
                  StartsWith(currentStatus, "applicant"),
                  "Former Applicant",
                  varClosedReferral.'Household Status'.Value
              )
          )
      )
  );

  // Update referral status and clear unit assignment
  Patch(
      'Referrals & Applicants',
      varClosedReferral,
      {
          'Household Status': { Value: varClosedStatus },
          'Unit Number': Blank()
      }
  );

  // Update unit record (clear referral household)
  If(
      !IsBlank(varClosedReferral.'Unit Number'),
      Patch(
          Units,
          LookUp(Units, ID = varClosedReferral.'Unit Number'.Id),
          {
              'Referral Household': Blank(),
              'Has referral': false
          }
      )
  );

  // Refresh and navigate
  Refresh('Referrals & Applicants');
  Navigate(RNABrowseScreen, ScreenTransition.None)
```

**Status Calculation Logic:**
- If current status starts with "referral" → set to "Former Referral"
- If current status starts with "applicant" → set to "Former Applicant"
- Otherwise → keep current status (edge case handling)

**Null Safety Pattern:**
The formula uses `If(!IsBlank(varClosedReferral.'Unit Number'), ...)` to handle referrals that were never assigned to a unit, preventing errors when trying to update non-existent unit relationships.

## Testing Procedure

### Prerequisites:
- Active referral or applicant record
- Referral assigned to a valid unit (for full testing)
- User has permissions to update both lists

### Basic Testing (Referral with Unit):

1. **Setup:**
   - Create test referral with status "Referral, New"
   - Assign to test unit
   - Verify unit shows "Has referral" = true
   - Navigate to RNAInfoScreen for test referral

2. **Execute:**
   - Verify "Close Referral" button is visible
   - Click "Close Referral" button
   - Verify navigation to RNACloseScreen
   - Verify form loads with referral data
   - Verify Close-out Date defaults to today (or select different date)
   - Select Close-out Reason from dropdown (e.g., "Declined offer")
   - Enter notes in Close-out Notes field (e.g., "Individual accepted housing at different location")
   - Click Save button

3. **Verify Processing:**
   - Verify navigation to RNABrowseScreen
   - Verify closed referral does NOT appear in browse view
   - Navigate to Units browse view
   - Find the unit the referral was assigned to
   - Verify unit "Referral Household" is now blank/empty
   - Verify unit "Has referral" shows No/False

### Detailed Record Verification:

4. **Verify Referral Record:**
   - Access Referrals & Applicants SharePoint list directly
   - Find the test referral (search by name)
   - Verify record still exists (not deleted)
   - Verify Household Status = "Former Referral"
   - Verify Close-out Date saved correctly
   - Verify Close-out Reason saved correctly
   - Verify Close-out Notes saved correctly
   - Verify Unit Number field is blank/empty

5. **Verify Unit Record:**
   - Navigate to UnitInfoScreen for the unit
   - Verify "Referral Household" field is blank
   - Verify "Has referral" shows No/False
   - Verify "Receive Referral" button now visible (unit available)

### Status Logic Testing:

6. **Test "Referral" Status:**
   - Create referral with status "Referral, New"
   - Execute close-out
   - Verify final status = "Former Referral"

7. **Test "Applicant" Status:**
   - Create referral and progress to "Applicant, Pending Approval"
   - Execute close-out
   - Verify final status = "Former Applicant"

8. **Test Different Referral Statuses:**
   - Test with "Referral, Screened" → should become "Former Referral"
   - Test with "Applicant, Approved" → should become "Former Applicant"

### Edge Case Testing:

9. **Test Referral Without Unit:**
   - Create referral without unit assignment (Unit Number blank)
   - Execute close-out
   - Verify successful processing (no error)
   - Verify status updated to "Former Referral"
   - Verify no attempt to update unit (no unit to update)

10. **Test All Close-Out Reasons:**
    - Test each dropdown option:
      - Declined offer
      - Withdrew application
      - Determined ineligible
      - Accepted housing elsewhere
      - Unreachable/No contact
      - Other
    - Verify each saves correctly

11. **Test Date Selection:**
    - Test with today's date (default)
    - Test with past date (backdating close-out)
    - Verify selected date saves correctly

12. **Test Notes Field:**
    - Test with minimal notes (few words)
    - Test with extensive notes (multiple paragraphs)
    - Verify all text saves correctly without truncation

### Data Integrity Testing:

13. **Verify Browse View Filters:**
    - Navigate to RNABrowseScreen
    - Confirm closed referral does NOT appear
    - Verify filter excludes records starting with "former"
    - Check count of active referrals decreased by 1

14. **Verify Unit Availability:**
    - Unit should now be available for new referral
    - Navigate to unit details
    - Click "Receive Referral" button
    - Create new referral
    - Verify successful (proves unit properly cleared)

## Known Issues

None currently - workflow tested and fully functional

**Design Pattern Excellence:**
This workflow demonstrates excellent multi-step coordination:
- Uses `With()` for cleaner intermediate calculations
- Handles nullable references gracefully with `If(!IsBlank(...))`
- Updates both referral and unit records for data consistency
- Preserves referral data by updating status rather than deleting
- Uses intelligent status logic based on progression through process

## Related Documentation

- [RNACloseScreen Documentation](../screens/RNACloseScreen.md)
- [RNAInfoScreen Documentation](../screens/RNAInfoScreen.md)
- [RNABrowseScreen Documentation](../screens/RNABrowseScreen.md)
- [UnitInfoScreen Documentation](../screens/UnitInfoScreen.md)
- [Receive Referral Workflow](./08a-Receive-Referral.md) - Beginning of referral lifecycle
- [Phone Screening Workflow](./08b-Phone-Screening.md) - May precede close-out
- [Intake Workflow](./08c-Intake.md) - May precede close-out if applicant
- [Data Model - Referrals & Applicants](../03-Data-Model.md)
- [Data Model - Units](../03-Data-Model.md)

**Future Enhancements:**
- Add confirmation dialog before close-out ("Are you sure?")
- Add ability to reopen closed referral (undo close-out)
- Create separate Referral Close-Out log list for audit trail
- Add notification to unit coordinator when referral closed
- Generate close-out letter/documentation automatically
- Add required fields validation (ensure reason and notes provided)
- Track close-out statistics for reporting (closure reasons analysis)
