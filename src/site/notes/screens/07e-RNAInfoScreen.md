---
{"dg-publish":true,"permalink":"/screens/07e-rna-info-screen/","dgShowToc":true}
---

# RNAInfoScreen

**Purpose:** Display and edit detailed referral/applicant information with workflow buttons.
**Status:** Mostly Complete (95%)
**Data Sources:** Referrals & Applicants, Units

## Overview

RNAInfoScreen serves as the central hub for managing individual referrals and applicants. The screen is divided into three main areas: a header showing unit and referral summary information, a left sidebar with workflow action buttons, and the main content area displaying an editable form with all referral details.

The workflow buttons provide quick access to key processes in the referral-to-resident journey: Phone Screening (initial contact), Intake (documentation verification), Create Resident (move-in conversion), and Close Referral (when applicant declines or is not housed).

Staff can edit referral information directly in the form, updating demographics, contact information, eligibility details, and case management notes as the referral progresses through the housing process.

## Key Components

### Main Controls
- **Header Section** - Displays unit number and referral name
- **Form1** - Main referral edit form with all demographic and eligibility fields
- **Button1** - Phone Screening workflow
- **Button2** - Intake workflow
- **Button3** - Create Resident workflow
- **Button4** - Close Referral workflow

### Data Flow
- **OnVisible:** Ensures `varSelectedReferral` and `varReferralUnit` variables are set
- **Form1.Item:** Bound to `varSelectedReferral`
- **Workflow Buttons:** Navigate to respective workflow screens with context set

## Features

### Referral Information Form
**Status:** Complete
**Description:** Comprehensive form displaying and allowing editing of all referral/applicant data.

**Field Categories:**
- **Demographics:** Name, Preferred Name, Preferred Language, Date of Birth
- **Contact:** Phone, Email
- **Household:** Household Size, Household Status
- **Eligibility:** Veteran Status, Disability, Benefits, Income
- **Case Management:** HMIS Record Number, County Case Number, Associated Providers
- **Unit Assignment:** Unit Number
- **Dates:** Date Referred, Intake Completed, Application Submitted

**Known Issue:** No OnSuccess handler means no user feedback after save and no automatic data refresh.

### Phone Screening Workflow
**Status:** Complete (screen not in current export)
**Description:** Button navigates to phone screening workflow.

**Formula:**
```powerapps
OnSelect: =Navigate(PhoneScreeningScreen, ScreenTransition.None)
```

**Note:** PhoneScreeningScreen was not included in the current export and may be in a different version or not yet built.

### Intake Workflow
**Status:** In Progress (30% complete)
**Description:** Button navigates to IntakeScreen for documentation verification.

**Formula:**
```powerapps
OnSelect: =Navigate(IntakeScreen, ScreenTransition.None)
```

**Status:** IntakeScreen exists but is incomplete (see IntakeScreen documentation for details).

### Create Resident Workflow
**Status:** Complete
**Description:** Button navigates to scrConvert for converting referral/applicant to current resident upon move-in.

**Formula:**
```powerapps
OnSelect: =Navigate(scrConvert, ScreenTransition.None)
```

**Status:** This workflow was just completed and is fully functional (see scrConvert documentation).

### Close Referral Workflow
**Status:** Complete
**Description:** Button navigates to RNACloseScreen for closing out referrals that declined or were not housed.

**Formula:**
```powerapps
OnSelect: |-
  =Set(varCloseRec, varSelectedReferral);
  Navigate(RNACloseScreen, ScreenTransition.None)
```

**Design Notes:**
- Sets `varCloseRec` as the context for the close workflow
- Passes the entire referral record for processing

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
  =Notify("Referral information updated successfully.", NotificationType.Success);
  Refresh('Referrals & Applicants');
```

### Phone Screening Screen Missing
**Severity:** Low
**Impact:** Workflow button exists but target screen may not
**Status:** Unknown - screen may be in different export or not yet built
**Workaround:** None if screen doesn't exist
**Investigation Needed:** Confirm if PhoneScreeningScreen exists and is functional

## Testing Notes

**Test Cases:**

**Form Display and Edit:**
- [ ] Navigate to RNAInfoScreen from browse
- [ ] Verify all referral fields display correctly
- [ ] Verify form is in edit mode by default or can be switched to edit
- [ ] Edit multiple fields
- [ ] Save form
- [ ] Verify changes persist (navigate away and back)
- [ ] Verify lookup fields (Unit Number, Associated Providers) display correctly
- [ ] Verify choice fields (Household Status, Veteran Status) display correctly
- [ ] Verify multi-choice fields (Disability, Benefits) display correctly

**Workflow Buttons:**
- [ ] Verify all four workflow buttons are visible
- [ ] Click Phone Screening - verify navigation (or error if screen missing)
- [ ] Click Intake - verify navigation to IntakeScreen
- [ ] Click Create Resident - verify navigation to scrConvert
- [ ] Click Close Referral - verify navigation to RNACloseScreen
- [ ] Verify varCloseRec is set correctly for close workflow

**Variable Context:**
- [ ] Verify varSelectedReferral contains complete record
- [ ] Verify varReferralUnit lookup succeeds
- [ ] Verify variables persist when navigating away and back

**Edge Cases:**
- [ ] Referral with no unit assigned
- [ ] Referral with minimal fields populated
- [ ] Referral with all fields populated
- [ ] Multiple users editing same referral (concurrency)

## Related Documentation

- [RNABrowseScreen](07b-RNABrowseScreen.md) - Browse all referrals/applicants
- [IntakeScreen](07h-IntakeScreen.md) - Intake documentation workflow (incomplete)
- [scrConvert](07g-scrConvert.md) - Convert to resident workflow (complete)
- [RNACloseScreen](07i-RNACloseScreen.md) - Close referral workflow (complete)
- [Data Model - Referrals & Applicants List](../03-Data-Model.md#referrals--applicants-list) - Schema for list
