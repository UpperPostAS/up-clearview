---
{"dg-publish":true,"permalink":"/workflows/08b-phone-screening/","dgShowToc":true}
---

# Phone Screening

**Purpose:** Conduct initial phone screening with referral to verify basic eligibility and interest
**Status:** ✅ Complete
**Trigger:** User clicks "Phone Screening" button on RNAInfoScreen
**Data Sources:** Referrals & Applicants (updates)

## Overview

The Phone Screening workflow guides staff through an initial eligibility conversation with a new referral. This is typically the first substantive contact between Upper Post Clearview staff and the referred individual, occurring shortly after the referral is received from a community partner.

During the phone screening, staff verify basic information, confirm the individual's interest in the housing opportunity, conduct preliminary eligibility screening based on unit type requirements, and schedule an intake appointment if appropriate. This workflow helps filter referrals early in the process and ensures that staff time is focused on viable housing placements.

The screen provides quick access to unit type eligibility requirements so staff can reference them during the conversation. Upon completion, the referral status is typically updated to reflect the screening outcome (e.g., "Referral, Screened" or "Former Referral" if not eligible/interested).

## User Journey

1. **Access Screen:** User views a referral on RNAInfoScreen
2. **Initiate Screening:** User clicks "Phone Screening" workflow button
3. **Navigate to Screen:** System navigates to PhoneScreeningScreen
4. **Review Info:** User reviews referral details and unit eligibility requirements displayed on screen
5. **Conduct Call:** User conducts phone screening conversation with referral
6. **Document Results:** User enters screening notes and outcomes
7. **Update Status:** User updates household status based on screening outcome
8. **Schedule Next Step:** If eligible and interested, user schedules intake appointment
9. **Submit:** User saves screening documentation
10. **Return:** System navigates back to RNAInfoScreen with updated status

## Technical Implementation

### Screen(s) Involved
- **RNAInfoScreen** - Starting point with "Phone Screening" button (Button1)
- **PhoneScreeningScreen** - Screen for conducting and documenting phone screening (not included in current code export)

### Data Operations

**Creates:**
- None directly (may create associated notes/activities in future)

**Updates:**
- Referrals & Applicants record with:
  - Phone screening notes/documentation
  - Household Status (based on screening outcome)
  - Scheduled intake date (if applicable)

**Deletes:**
- None

### Key Formulas

**Navigation from RNAInfoScreen:**
```powerapps
OnSelect: =Navigate(PhoneScreeningScreen, ScreenTransition.None)
```

**Note:** The PhoneScreeningScreen implementation details are not available in the current code export, indicating this screen may be in a different version or still under development.

## Testing Procedure

1. **Setup:**
   - Create a test referral with status "Referral, New"
   - Ensure referral has basic contact information (phone number)
   - Navigate to RNAInfoScreen for the test referral

2. **Execute:**
   - Verify "Phone Screening" button is visible
   - Click "Phone Screening" button
   - Verify navigation to PhoneScreeningScreen
   - Review displayed referral information
   - Review unit eligibility requirements
   - Enter test screening notes
   - Update household status (e.g., to "Referral, Screened")
   - Click Save

3. **Verify:**
   - Successful save notification appears
   - System navigates back to RNAInfoScreen
   - Referral status updated correctly
   - Screening notes saved
   - If scheduled, intake date appears on referral record
   - Updated referral appears correctly in RNABrowseScreen

## Known Issues

**Screen Not in Export:** The PhoneScreeningScreen is not included in the current code export. This may indicate:
- Screen exists in a different app version
- Screen is under active development
- Navigation reference exists but screen not yet built

**Status:** Marked as Complete based on technical documentation, but implementation details need verification.

## Related Documentation

- [RNAInfoScreen Documentation](../screens/RNAInfoScreen.md)
- [Receive Referral Workflow](./08a-Receive-Referral.md) - Previous step
- [Intake Workflow](./08c-Intake.md) - Next step after successful screening
- [Close-Out Referral Workflow](./08f-Close-Out-Referral.md) - Used if referral declines/ineligible
- [Data Model - Referrals & Applicants](../03-Data-Model.md)
