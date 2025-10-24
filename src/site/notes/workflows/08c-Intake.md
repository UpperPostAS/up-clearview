---
{"dg-publish":true,"permalink":"/workflows/08c-intake/","dgShowToc":true}
---

# Intake

**Purpose:** Guide staff through comprehensive intake documentation and eligibility verification based on unit type requirements
**Status:** ❌ Incomplete - Form not built
**Trigger:** User clicks "Intake" button on RNAInfoScreen
**Data Sources:** Referrals & Applicants (updates)

## Overview

The Intake workflow is a critical step in the housing placement process where staff conduct comprehensive eligibility verification and collect required documentation from the applicant. This process ensures that all program requirements are met before an applicant is approved for housing.

The intake requirements vary significantly based on the unit type (Section 8 PBV, HUD-VASH PBV, LTH Housing Support, or CoC Rental Assistance), and the IntakeScreen is designed to display the specific requirements for the applicant's assigned unit type. Staff use this screen to systematically verify and document compliance with household composition requirements, income restrictions, homelessness verification criteria, disability documentation, and other program-specific eligibility factors.

Currently, the screen displays the eligibility requirements but does not yet have a functional form for documenting completion. This is a high-priority development task blocking the workflow from being fully operational.

## User Journey

1. **Access Screen:** User views an applicant on RNAInfoScreen (typically after phone screening)
2. **Initiate Intake:** User clicks "Intake" workflow button
3. **Navigate to Screen:** System navigates to IntakeScreen
4. **Review Requirements:** User reviews unit-specific eligibility requirements displayed on left panel:
   - Household Composition (min/max occupancy)
   - Income Restriction (guidelines and limits)
   - Other Eligibility (homelessness verification, disability documentation, etc.)
5. **Collect Documentation:** User works with applicant to collect required documents
6. **Document Checklist:** *(FORM NOT YET BUILT)* User checks off each document as received and verified
7. **Enter Notes:** *(FORM NOT YET BUILT)* User enters verification notes and any special circumstances
8. **Confirm Eligibility:** *(FORM NOT YET BUILT)* User confirms all eligibility criteria met
9. **Submit:** *(FORM NOT YET BUILT)* User saves intake documentation
10. **Update Status:** System updates referral status to "Applicant, Pending Approval"
11. **Return:** System navigates back to RNAInfoScreen

## Technical Implementation

### Screen(s) Involved
- **RNAInfoScreen** - Starting point with "Intake" button (Button2)
- **IntakeScreen** - Intake documentation screen (partially complete)
- **Container6** - Container displaying eligibility requirements
- **Form2** - Placeholder form (not yet configured)

### Data Operations

**Creates:**
- None currently (may create intake documentation records in future)

**Updates:**
- Referrals & Applicants record with:
  - Intake Completion Date
  - Intake Completed By (staff member name)
  - Documentation checklist status
  - Verification notes
  - Eligibility confirmed flag
  - Household Status (updated to "Applicant, Pending Approval")

**Deletes:**
- None

### Key Formulas

**Navigation from RNAInfoScreen:**
```powerapps
OnSelect: =Navigate(IntakeScreen, ScreenTransition.None)
```

**OnVisible - Load Unit Type Requirements:**
```powerapps
OnVisible: |-
  Set(varIntakeReferral, varSelectedReferral);
  Set(varIntakeUnit, LookUp(Units, ID = varIntakeReferral.'Unit Number'.Id));
  Set(varUnitType, LookUp('Unit Types', 'Unit Type' = varIntakeUnit.'Unit Type'.Value));
```

**OnSuccess Handler (TO BE IMPLEMENTED):**
```powerapps
OnSuccess: |-
  =Patch(
      'Referrals & Applicants',
      varIntakeReferral,
      {
          'Intake Completed': IntakeForm.LastSubmit.'Intake Completion Date',
          'Household Status': { Value: "Applicant, Pending Approval" }
      }
  );
  Notify("Intake completed successfully.", NotificationType.Success);
  Refresh('Referrals & Applicants');
  Navigate(RNAInfoScreen, ScreenTransition.None)
```

## Testing Procedure

**Note:** Full testing cannot be completed until form is built.

### Current Testing (Requirements Display Only):

1. **Setup:**
   - Create a test referral with status "Referral, Screened"
   - Assign referral to a unit with known type
   - Navigate to RNAInfoScreen for the test referral

2. **Execute:**
   - Verify "Intake" button is visible and enabled
   - Click "Intake" button
   - Verify navigation to IntakeScreen successful

3. **Verify:**
   - Screen loads without errors
   - Left panel displays eligibility requirements
   - Requirements match the unit type (verify against Unit Types list)
   - Unit information displays correctly
   - Referral information displays correctly

### Future Testing (After Form Completion):

1. **Setup:** Same as above

2. **Execute:**
   - Navigate to IntakeScreen
   - Review eligibility requirements
   - Complete intake form with all required documentation checked
   - Enter test verification notes
   - Mark eligibility as confirmed
   - Click Save

3. **Verify:**
   - Success notification appears
   - Form saves without errors
   - Referral status updated to "Applicant, Pending Approval"
   - Intake completion date set to today
   - Documentation checklist saved
   - Verification notes saved
   - System navigates back to RNAInfoScreen
   - Updated status visible on referral record
   - Intake button disabled after completion (optional enhancement)

## Known Issues

**Critical Blocker - Form Not Built:**
- **Severity:** HIGH
- **Impact:** Core workflow unavailable
- **Symptoms:** Screen displays requirements but has no data entry form
- **Cause:** Form2 placeholder not configured with fields or logic
- **Workaround:** Manual intake documentation outside app (Word doc, paper form)
- **Fix Required:** Build intake form with documentation checklist (estimated 6-8 hours)

**Missing Form Fields:**
- Intake Completion Date (Date picker)
- Intake Completed By (Text or Person field)
- Documentation Checklist (Multi-choice or individual Yes/No fields for each document type)
- Verification Notes (Multi-line text)
- Eligibility Confirmed (Yes/No checkbox)

**Missing Business Logic:**
- No validation to ensure required documents checked before save
- No conditional logic to show/hide document requirements based on unit type
- No OnSuccess handler to update referral status
- No navigation back to RNAInfoScreen after save

## Related Documentation

- [IntakeScreen Documentation](../screens/IntakeScreen.md)
- [RNAInfoScreen Documentation](../screens/RNAInfoScreen.md)
- [Phone Screening Workflow](./08b-Phone-Screening.md) - Previous step
- [Convert to Resident Workflow](./08d-Convert-to-Resident.md) - Next step after approval
- [Data Model - Unit Types](../03-Data-Model.md) - Eligibility requirements by unit type
- [Data Model - Referrals & Applicants](../03-Data-Model.md)
- [Development Roadmap](../10-Roadmap.md) - Intake completion priority
