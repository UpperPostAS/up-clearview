---
{"dg-publish":true,"permalink":"/screens/07i-rna-close-screen/","dgShowToc":true}
---

# RNACloseScreen

**Purpose:** Close out a referral that declined, withdrew, or was otherwise not housed.
**Status:** Complete
**Data Sources:** Referrals & Applicants, Units

## Overview

RNACloseScreen handles the workflow for closing out referrals and applicants who will not be housed. This may occur because the applicant declined the unit, withdrew from the process, was determined ineligible, or for other reasons. The screen presents a simple form with three fields for documenting the close-out.

When the form is submitted, the system performs several coordinated updates: it changes the household status to "Former Referral" or "Former Applicant" (preserving the original type), clears the unit assignment from the referral record, and updates the associated unit to clear the referral household and make it available for a new referral.

This workflow ensures data integrity by maintaining bidirectional relationships between referrals and units, preventing orphaned references when referrals are closed.

## Key Components

### Main Controls
- **EditForm1** - Form bound to Referrals & Applicants list
- **DataCard (Close-out Date)** - DatePicker for close-out date
- **DataCard (Close-out Reason)** - ComboBox for reason (dropdown)
- **DataCard (Close-out Notes)** - TextInput for detailed notes

### Data Flow
- **Form.Item:** Bound to `varCloseRec` (set from RNAInfoScreen)
- **Form.OnSuccess:** Multi-step workflow:
  1. Capture submitted record in varClosedReferral
  2. Calculate new status (add "Former" prefix)
  3. Update referral (set new status, clear unit number)
  4. Update unit (clear referral household, set has referral = false)
  5. Refresh data
  6. Navigate back to RNABrowseScreen

## Features

### Close-out Documentation
**Status:** Complete
**Description:** Form captures three essential pieces of information about the close-out.

**Fields:**
1. **Close-out Date** (DatePicker, defaults to today)
2. **Close-out Reason** (ComboBox with predefined reasons)
3. **Close-out Notes** (Multi-line text for additional context)

### Automatic Status Calculation
**Status:** Complete
**Description:** Intelligently determines the new "Former" status based on the referral's current household status.

**Logic:**
```powerapps
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
```

**Design Notes:**
- Uses `With()` for cleaner intermediate calculation
- Preserves distinction between referrals and applicants
- Handles null/blank status gracefully with Coalesce()
- Falls back to current status if neither referral nor applicant

### Referral Record Update
**Status:** Complete
**Description:** Updates the referral record with new status and clears unit assignment.

**Formula:**
```powerapps
Patch(
    'Referrals & Applicants',
    varClosedReferral,
    {
        'Household Status': { Value: varClosedStatus },
        'Unit Number': Blank()
    }
);
```

**Design Notes:**
- Close-out fields (date, reason, notes) saved by form submission
- Status and unit clearing done in OnSuccess via Patch
- Clearing unit number breaks the referral → unit relationship

### Unit Record Update
**Status:** Complete
**Description:** Updates the associated unit to clear the referral household and make it available for a new referral.

**Formula:**
```powerapps
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
```

**Design Notes:**
- Uses conditional `If()` to only update unit if one was assigned
- Handles referrals without unit assignment gracefully
- Clears referral household (breaks unit → referral relationship)
- Sets has referral flag to false (makes unit available for new referral)
- Uses LookUp to find unit record by ID from the referral's unit number field

### Complete OnSuccess Workflow
**Status:** Complete
**Description:** Comprehensive multi-step workflow ensuring data consistency.

**Complete Formula:**
```powerapps
OnSuccess: |-
  =Set(varClosedReferral, EditForm1.LastSubmit);

  // Calculate new status
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

  // Update referral status and clear unit
  Patch(
      'Referrals & Applicants',
      varClosedReferral,
      {
          'Household Status': { Value: varClosedStatus },
          'Unit Number': Blank()
      }
  );

  // Update unit (clear referral household)
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

  Refresh('Referrals & Applicants');
  Navigate(RNABrowseScreen, ScreenTransition.None)
```

## Known Issues

None currently. Workflow is fully functional and maintains data integrity.

## Testing Notes

**Test Cases - Complete Workflow:**
- [ ] Navigate to RNAInfoScreen for active referral with unit assigned
- [ ] Click "Close Referral" button
- [ ] Verify navigation to RNACloseScreen
- [ ] Verify form displays with referral context
- [ ] Verify Close-out Date defaults to today
- [ ] Select close-out reason from dropdown
- [ ] Enter close-out notes
- [ ] Click Save/Submit
- [ ] Verify form submits without errors
- [ ] Verify navigation back to RNABrowseScreen
- [ ] Verify closed referral no longer appears in browse (filtered out)
- [ ] Navigate to Units browse
- [ ] Find the unit that had the referral
- [ ] Verify unit no longer shows referral household
- [ ] Verify unit's "Has referral" flag is false
- [ ] Verify unit can now receive a new referral (button appears on UnitInfoScreen)

**Test Cases - Status Transitions:**
- [ ] Close "Referral, New" → Should become "Former Referral"
- [ ] Close "Referral, Contacted" → Should become "Former Referral"
- [ ] Close "Applicant, Pending Approval" → Should become "Former Applicant"
- [ ] Close "Applicant, Approved" → Should become "Former Applicant"

**Test Cases - Edge Cases:**
- [ ] Close referral with no unit assigned (should work, skip unit update)
- [ ] Close referral with minimal data (name only)
- [ ] Close referral with all fields populated
- [ ] Close referral twice (should not be possible - filtered from browse)
- [ ] Leave close-out notes blank (should work if not required)
- [ ] Very long close-out notes (test character limits)

**Test Cases - Data Integrity:**
- [ ] After close, verify referral record has close-out date
- [ ] After close, verify referral record has close-out reason
- [ ] After close, verify referral record has close-out notes
- [ ] After close, verify referral record has "Former" status
- [ ] After close, verify referral record has blank unit number
- [ ] After close, verify unit record has blank referral household
- [ ] After close, verify unit record has has referral = false
- [ ] Navigate away and back - verify all changes persist

**Test Cases - Concurrent Users:**
- [ ] Two users viewing same referral
- [ ] Both attempt to close simultaneously
- [ ] Verify one succeeds and other gets appropriate error/feedback

## Related Documentation

- [RNAInfoScreen](07e-RNAInfoScreen.md) - Source screen for close referral workflow
- [RNABrowseScreen](07b-RNABrowseScreen.md) - Destination after close (former referrals filtered out)
- [UnitInfoScreen](07d-UnitInfoScreen.md) - Unit becomes available for new referral after close
- [Data Model - Referrals & Applicants List](../03-Data-Model.md#referrals--applicants-list) - Schema including close-out fields
