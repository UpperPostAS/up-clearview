---
{"dg-publish":true,"permalink":"/workflows/08a-receive-referral/","dgShowToc":true}
---

# Receive Referral

**Purpose:** Create a new referral record when a unit receives a housing referral from a community partner
**Status:** ✅ Complete
**Trigger:** User clicks "Receive Referral" button on UnitInfoScreen for a vacant unit
**Data Sources:** Referrals & Applicants (creates), Units (updates)

## Overview

The Receive Referral workflow enables staff to quickly document new housing referrals when they arrive. When a unit becomes available and a referral is received from a community partner (such as coordinated entry, a county case manager, or a service provider), staff can initiate this workflow to create a referral record in the system.

The workflow is context-aware and only appears when appropriate - specifically, when viewing a vacant unit that does not already have a referral assigned. This prevents duplicate referral assignments and ensures data integrity.

The system automatically pre-populates key information (unit number, unit type, current date) to streamline data entry. After submission, both the referral record and the associated unit record are updated to reflect the new referral assignment.

## User Journey

1. **Navigate to Unit:** User views a vacant unit on UnitInfoScreen
2. **Check Eligibility:** System displays "Receive Referral" button only if unit is vacant and has no existing referral
3. **Open Form:** User clicks "Receive Referral" button, modal form appears
4. **Enter Basic Info:** User enters referral name (head of household)
5. **Review Pre-filled Data:** System displays auto-populated fields:
   - Unit Number (from current unit)
   - Unit Type (from current unit)
   - Date Referred (today's date)
   - Household Status (default: "Referral, New")
6. **Submit:** User clicks Save button
7. **Process:** System creates referral record
8. **Update Unit:** System updates unit to show referral household and sets "Has referral" flag
9. **Close Modal:** Form closes and data refreshes
10. **Navigate:** User can immediately navigate to new referral record for additional details

## Technical Implementation

### Screen(s) Involved
- **UnitInfoScreen** - Primary screen where workflow is initiated
- **Container3** - Modal container holding the referral form
- **ReferralForm** - Form control for data entry

### Data Operations

**Creates:**
- New record in Referrals & Applicants list with:
  - Title (head of household name)
  - Unit Number (lookup to Units)
  - Unit Type (lookup to Unit Types)
  - Date Referred (today's date)
  - Household Status (default: "Referral, New")

**Updates:**
- Units record with:
  - Referral Household (lookup to newly created referral)
  - Has referral (boolean set to true)

**Deletes:**
- None

### Key Formulas

**Button Visibility (Only show when applicable):**
```powerapps
Visible: =And(
    varActiveUnit.'Is occupied' = false,
    IsBlank(varActiveUnit.'Referral Household')
)
```

**Form Field Defaults (Pre-populate fields):**
```powerapps
Default: =If(
    DataCard.DisplayName = "Unit Number",
    varActiveUnit,
    If(
        DataCard.DisplayName = "Date Referred",
        Today(),
        // ... other defaults
    )
)
```

**OnSuccess Handler (After form saves):**
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

## Testing Procedure

1. **Setup:**
   - Ensure at least one vacant unit exists (Is occupied = false)
   - Ensure unit has no existing referral (Referral Household is blank)
   - Navigate to UnitInfoScreen for the vacant unit

2. **Execute:**
   - Verify "Receive Referral" button is visible
   - Click "Receive Referral" button
   - Verify modal form opens
   - Enter name for head of household (e.g., "Test Resident")
   - Verify Unit Number field shows correct unit
   - Verify Date Referred shows today's date
   - Click Save

3. **Verify:**
   - Success: Modal closes without errors
   - Unit record updated: Refresh UnitInfoScreen and verify "Referral Household" field shows new referral name
   - Unit flag updated: Verify "Has referral" shows as Yes/True
   - Referral created: Navigate to RNABrowseScreen and verify new referral appears in list
   - Referral data: Click on new referral and verify all fields populated correctly
   - Button hidden: Return to UnitInfoScreen and verify "Receive Referral" button no longer shows (since unit now has referral)

## Known Issues

None currently

## Related Documentation

- [UnitInfoScreen Documentation](../screens/UnitInfoScreen.md)
- [RNABrowseScreen Documentation](../screens/RNABrowseScreen.md)
- [Data Model - Referrals & Applicants](../03-Data-Model.md)
- [Data Model - Units](../03-Data-Model.md)
- [Phone Screening Workflow](./08b-Phone-Screening.md) - Next step after receiving referral
- [Intake Workflow](./08c-Intake.md) - Subsequent step in referral process
