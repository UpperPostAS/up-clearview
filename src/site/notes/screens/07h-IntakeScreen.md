---
{"dg-publish":true,"permalink":"/screens/07h-intake-screen/","dgShowToc":true}
---

# IntakeScreen

**Purpose:** Guide staff through intake documentation requirements based on unit type.
**Status:** Incomplete (30% complete)
**Data Sources:** Referrals & Applicants, Units, Unit Types

## Overview

IntakeScreen is designed to guide staff through the intake documentation verification process for referrals. The screen currently displays eligibility requirements specific to the unit type (showing what documents and verifications are needed), but the form for recording intake completion is not yet built.

The left panel shows comprehensive eligibility requirements including household composition rules, income restrictions, homelessness definitions, and disability documentation requirements. This information is dynamically pulled from the Unit Types list based on the referral's assigned unit, ensuring staff verify the correct criteria.

The right panel contains a placeholder form (Form2) that needs to be developed with intake checklist fields, completion dates, and staff notes.

## Key Components

### Main Controls
- **Header Section** - Displays unit and referral info
- **Container6** - Left panel displaying eligibility requirements
  - Household Composition (min/max occupancy)
  - Income Restriction (guidelines and limits)
  - Other Eligibility (homelessness, disability docs)
- **Form2** - Empty placeholder form (NOT CONFIGURED)

### Data Flow
- **OnVisible:** Sets three variables for context:
  - `varIntakeReferral` = varSelectedReferral
  - `varIntakeUnit` = LookUp unit by referral's Unit Number
  - `varUnitType` = LookUp unit type by unit's Unit Type value

```powerapps
OnVisible: |-
  Set(varIntakeReferral, varSelectedReferral);
  Set(varIntakeUnit, LookUp(Units, ID = varIntakeReferral.'Unit Number'.Id));
  Set(varUnitType, LookUp('Unit Types', 'Unit Type' = varIntakeUnit.'Unit Type'.Value));
```

## Features

### Eligibility Requirements Display
**Status:** Complete
**Description:** Dynamically displays unit type specific eligibility requirements to guide intake verification.

**Requirements Shown:**
- **Household Composition:**
  - Minimum Occupancy (from Unit Types)
  - Maximum Occupancy (from Unit Types)
  - Guidance on household size requirements
- **Income Restriction:**
  - Income Guideline (% of AMI)
  - Income Limit (dollar amount)
  - Description of income verification needed
- **Homelessness Requirement:**
  - Definition and verification requirements (if applicable)
- **Disability Documentation:**
  - Required documentation types (if applicable)

**Design Pattern:**
Uses conditional visibility based on unit type to show only relevant requirements. For example, homelessness requirements only appear for CoC units.

### Intake Documentation Form
**Status:** Not Built
**Description:** Form for recording intake completion and documentation verification.

**What Needs to Be Built:**

**Required Form Fields:**
1. **Intake Completion Date** (Date, required)
2. **Intake Completed By** (Person or Text, required)
3. **Documentation Checklist** (Individual Yes/No fields or multi-choice):
   - State ID or Driver's License
   - Social Security Card
   - Birth Certificate (all household members)
   - Income Verification (paystubs, award letters, etc.)
   - Disability Documentation (if required by unit type)
   - Homelessness Verification (if required by unit type)
   - Professional Statement of Need (if required)
   - Landlord References
   - Other unit-type-specific documents
4. **Verification Notes** (Multi-line text, optional)
5. **Eligibility Confirmed** (Yes/No, required)
6. **Income Verified Amount** (Number, optional)
7. **Issues or Concerns** (Multi-line text, optional)

**Conditional Field Logic:**
- Disability Documentation checkbox should only appear if unit type requires it
- Homelessness Verification should only appear if unit type requires it
- Professional Statement should only appear if unit type requires it

**Form Configuration:**
```powerapps
Form2.DataSource: ='Referrals & Applicants'
Form2.Item: =varIntakeReferral
Form2.DefaultMode: =FormMode.Edit
```

**OnSuccess Logic Needed:**
```powerapps
Form2.OnSuccess: |-
  =Patch(
      'Referrals & Applicants',
      varIntakeReferral,
      {
          'Intake Completed': Form2.LastSubmit.'Intake Completion Date',
          'Household Status': { Value: "Applicant, Pending Approval" }
      }
  );
  Notify("Intake completed successfully. Status updated to Applicant, Pending Approval.", NotificationType.Success);
  Refresh('Referrals & Applicants');
  Navigate(RNAInfoScreen, ScreenTransition.None)
```

**Design Notes:**
- After successful intake, referral should advance from "Referral" status to "Applicant" status
- Form should validate that required documentation is checked before allowing save
- Consider adding a "Save Draft" option for partial completion
- Intake date should be saved to referral record for tracking

## Known Issues

### Form Not Configured
**Severity:** High (Blocking)
**Impact:** Core intake workflow unavailable - staff cannot record intake completion
**Symptoms:** Screen shows requirements but has no way to record verification
**Cause:** Form2 is placeholder, not connected to data source or configured with fields
**Workaround:** Manual intake tracking outside the app (spreadsheet or paper)
**Fix Required:** Build complete intake form with fields and workflow
**Estimated Effort:** 6-8 hours
  - 2 hours: Design form fields and layout
  - 2 hours: Build form and connect to data source
  - 1 hour: Add conditional logic for unit-type-specific fields
  - 1 hour: Implement OnSuccess workflow
  - 1-2 hours: Testing and refinement

## Testing Notes

**Current State Testing:**
- [x] Screen loads without errors
- [x] Eligibility requirements display correctly
- [x] OnVisible sets variables correctly
- [x] Unit type lookup works
- [x] Requirements are unit-type specific

**Future Testing (After Form Built):**
- [ ] Form displays all required fields
- [ ] Required fields enforce validation
- [ ] Documentation checklist items save correctly
- [ ] Conditional fields show/hide based on unit type
- [ ] Homelessness verification only appears for CoC units
- [ ] Disability documentation only appears for units requiring it
- [ ] Save updates referral record
- [ ] Save sets Intake Completed date
- [ ] Save advances Household Status to "Applicant, Pending Approval"
- [ ] Success notification appears
- [ ] Navigation returns to RNAInfoScreen
- [ ] Intake button disabled/hidden after completion
- [ ] Data refreshes on RNAInfoScreen after return

**Edge Cases to Test:**
- [ ] Intake for referral without unit assigned (should show error or default requirements)
- [ ] Intake with partial documentation (save draft functionality)
- [ ] Multiple staff members viewing same intake simultaneously
- [ ] Editing completed intake record

## Development Roadmap

**Week 1: Form Design (2 hours)**
- [ ] Define complete field list based on unit type requirements
- [ ] Design form layout (single column, two column, or tabs)
- [ ] Determine which fields are required vs optional
- [ ] Map conditional visibility rules

**Week 2: Form Implementation (4 hours)**
- [ ] Configure Form2 data source and item
- [ ] Add all data cards for intake fields
- [ ] Implement documentation checklist
- [ ] Add conditional visibility to unit-type-specific fields
- [ ] Configure default values (intake date = today, completed by = current user)
- [ ] Add OnSuccess handler with status update and navigation

**Week 3: Testing & Refinement (2 hours)**
- [ ] Test with all unit types
- [ ] Verify conditional fields work correctly
- [ ] Test validation and required field enforcement
- [ ] Test workflow end-to-end
- [ ] Refine UI based on user feedback

**Priority:** HIGH - This is a critical workflow blocker for deployment.

## Related Documentation

- [RNAInfoScreen](07e-RNAInfoScreen.md) - Source screen for intake workflow
- [Unit Types Configuration](../03-Data-Model.md#unit-types-list) - Eligibility requirements by funding type
- [Referrals & Applicants Schema](../03-Data-Model.md#referrals--applicants-list) - Fields available for intake data
- [Development Roadmap](../10-Roadmap.md) - Priority and timeline for completion
