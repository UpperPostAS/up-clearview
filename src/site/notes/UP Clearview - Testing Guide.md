---
{"dg-publish":true,"permalink":"/up-clearview-testing-guide/","dgShowToc":true}
---

**Comprehensive Testing Documentation**

---

## About This Guide

Use this guide to plan, execute, and document testing for UP Clearview. It captures the progressive debugging philosophy we use in development, detailed checklists by screen, and templates for reporting issues. For open development tasks, cross-reference the [[UP Clearview - Development Roadmap\|development roadmap]]; for schema specifics, see the [[UP Clearview – Data Sources\|data source reference]].

---

## Testing Philosophy

### Core Principle: Start Small, Build Up

**When facing issues, always use the smallest piece of functionality you can, progressively adding more to discover exactly where the issue is.**

This approach:
- Isolates problems quickly
- Prevents wasted time debugging the wrong thing
- Builds confidence as each piece works
- Makes complex workflows manageable

### Testing Levels

1. **Component Testing** - Individual controls work correctly
2. **Screen Testing** - All controls on a screen work together
3. **Workflow Testing** - Multi-screen processes complete successfully
4. **Integration Testing** - Data stays consistent across related records
5. **System Testing** - App functions correctly as a whole

---

## Progressive Debugging Strategy

### When a Feature Doesn't Work

#### Step 1: Minimal Test
Start with the absolute minimum required fields/actions.

**Example - Testing a Patch:**
```powerapps
// Start with ONLY required field
Patch('Current Residents', Defaults('Current Residents'), {
    Title: "Test Name"
})
```

**Result:**
- ✅ Works? → Proceed to Step 2
- ❌ Fails? → Fix connection, permissions, or required field issue

#### Step 2: Add Simple Fields One at a Time
```powerapps
// Add one text field
Patch('Current Residents', Defaults('Current Residents'), {
    Title: "Test Name",
    Phone: "555-0100"
})
```

**Result:**
- ✅ Works? → Add next field
- ❌ Fails? → Phone field has an issue

#### Step 3: Add Complex Fields Individually
```powerapps
// Add lookup field
Patch('Current Residents', Defaults('Current Residents'), {
    Title: "Test Name",
    Phone: "555-0100",
    'Unit Number': varSelectedUnit
})
```

**Result:**
- ✅ Works? → Add next complex field
- ❌ Fails? → Unit Number lookup structure wrong

#### Step 4: Complete Formula
Once all individual fields work, combine them.

### Debugging Tools

**Display Variable Contents:**
```powerapps
// Add a Label with this formula to see variable structure
Label.Text: =JSON(varSelectedReferral, JSONFormat.IndentFour)
```

**Check Lookup Field Structure:**
```powerapps
// Verify lookup has Id and Value
Label.Text: ="ID: " & varSelectedReferral.'Unit Number'.Id &
            " | Value: " & varSelectedReferral.'Unit Number'.Value
```

**Check If Variable Is Blank:**
```powerapps
Label.Text: =If(IsBlank(varSelectedReferral), "No referral selected", "Referral loaded")
```

**Check Form Errors:**
```powerapps
// Shows last error from a form
Label.Text: =First(Errors('Current Residents')).Message
```

---

## Pre-Testing Setup

### Test Environment Requirements

**User Account:**
- [ ] Test user has appropriate SharePoint permissions
- [ ] Test user can create, read, update, delete in all lists
- [ ] Test user has Power Apps access

**Test Data:**
- [ ] At least 3 test units created
  - [ ] 1 vacant unit (no occupant, no referral)
  - [ ] 1 occupied unit (has current resident)
  - [ ] 1 unit with pending referral (has referral household)
- [ ] At least 2 test referrals
- [ ] At least 2 test residents
- [ ] At least 2 test service providers
- [ ] At least 1 test organization/agency

**Browser Setup:**
- [ ] Clear browser cache
- [ ] Disable browser extensions that might interfere
- [ ] Use supported browser (Edge, Chrome, Firefox)
- [ ] Enable JavaScript
- [ ] Allow pop-ups from Power Apps domain

**Documentation Ready:**
- [ ] Have SharePoint list URLs available
- [ ] Have bug reporting template ready
- [ ] Have screen recording tool available (for bug reports)

---

## Unit Testing by Screen

### Test Template

For each screen, verify:
1. Screen loads without errors
2. Variables are set correctly on load
3. Controls display data correctly
4. User interactions work as expected
5. Navigation functions properly

---

### UnitBrowseScreen

**Test ID:** UBS-001
**Prerequisites:** At least 3 units exist in Units list

#### Test Cases

**UBS-001.1: Screen Load**
- [ ] Navigate to UnitBrowseScreen
- [ ] Verify screen loads within 3 seconds
- [ ] Verify no error messages appear
- [ ] Verify colStatusPalette collection is populated

**Expected Result:** Screen displays without errors, color palette loaded

---

**UBS-001.2: Gallery Display - All Units**
- [ ] Ensure "Show Vacancies" toggle is OFF
- [ ] Verify all units appear in gallery
- [ ] Verify unit numbers display correctly
- [ ] Verify unit types display correctly
- [ ] Verify color coding matches unit type

**Expected Result:** All units visible with correct data and colors

**Color Verification:**
- LTH Housing Support: Light green (#C6EFCE)
- Section 8 PBV: Light blue (#DDEBF7)
- Ramsey CoC: Light yellow (#FFF2CC)
- HUD-VASH: Light purple (#E5DFEC)

---

**UBS-001.3: Gallery Display - Vacancies Only**
- [ ] Toggle "Show Vacancies" to ON
- [ ] Verify only vacant/notice units appear
- [ ] Verify occupied units are filtered out
- [ ] Verify referral-pending units appear if vacant

**Expected Result:** Gallery filters to show only available/upcoming vacancies

---

**UBS-001.4: Unit Selection and Navigation**
- [ ] Click on a unit card
- [ ] Verify unit card highlights (selection indicator)
- [ ] Verify navigation to UnitInfoScreen
- [ ] Verify varActiveUnit is set correctly

**Expected Result:** Selected unit data carries to UnitInfoScreen

**How to Verify:** Add temporary label on UnitInfoScreen:
```powerapps
Label.Text: ="Unit ID: " & varActiveUnit.ID & " | Unit: " & varActiveUnit.Unit
```

---

**UBS-001.5: Toggle Interaction**
- [ ] Toggle "Show Vacancies" ON
- [ ] Count units displayed
- [ ] Toggle OFF
- [ ] Verify count increases (all units shown)
- [ ] Toggle ON again
- [ ] Verify filter reapplies

**Expected Result:** Toggle controls gallery filter correctly

---

### UnitInfoScreen

**Test ID:** UIS-002
**Prerequisites:** At least 1 vacant unit exists

#### Test Cases

**UIS-002.1: Screen Load - Vacant Unit**
- [ ] Select a vacant unit from browse screen
- [ ] Navigate to UnitInfoScreen
- [ ] Verify left panel shows unit details
- [ ] Verify right panel shows editable status form
- [ ] Verify varUnitType is loaded

**Expected Result:** All data displays correctly

---

**UIS-002.2: Eligibility Display**
- [ ] Verify "Eligibility Information" section populates
- [ ] Check Income Limit displays (from Unit Type)
- [ ] Check Homelessness Requirements display
- [ ] Check Disability Documentation displays
- [ ] Verify all data comes from linked Unit Type

**Expected Result:** Eligibility requirements match unit's Unit Type configuration

---

**UIS-002.3: Receive Referral Button Visibility**
- [ ] Navigate to vacant unit with no referral
- [ ] Verify "Receive Referral" button is visible and enabled
- [ ] Navigate to occupied unit
- [ ] Verify button is hidden or disabled
- [ ] Navigate to vacant unit with existing referral
- [ ] Verify button is hidden or disabled

**Expected Result:** Button only appears when appropriate

**Visibility Logic:**
```powerapps
Visible: =And(
    varActiveUnit.'Is occupied' = false,
    IsBlank(varActiveUnit.'Referral Household')
)
```

---

**UIS-002.4: Receive Referral Workflow**
See [Workflow Testing - Receive Referral](#receive-referral-workflow) section

---

**UIS-002.5: Unit Status Edit**
- [ ] Click edit/pencil icon on right panel form
- [ ] Modify "Date Referral Requested"
- [ ] Click Save
- [ ] Verify save completes (check for error or success)
- [ ] Refresh SharePoint list
- [ ] Verify change persisted

**Expected Result:** Edit saves successfully

**Known Issue:** No OnSuccess handler - no user feedback

---

### RNABrowseScreen

**Test ID:** RBS-003
**Prerequisites:** At least 2 referrals exist, at least 1 "former" referral exists

#### Test Cases

**RBS-003.1: Screen Load**
- [ ] Navigate to RNABrowseScreen
- [ ] Verify screen loads within 3 seconds
- [ ] Verify colStatusPalette collection loads
- [ ] Verify gallery populates

**Expected Result:** Screen loads without errors

---

**RBS-003.2: Gallery Display - Active Referrals Only**
- [ ] Verify all active referrals display
- [ ] Verify "Former Referral" records do NOT appear
- [ ] Verify "Former Applicant" records do NOT appear
- [ ] Check each item shows unit number, name, status

**Expected Result:** Only active referrals/applicants shown

**Filter Logic to Verify:**
```powerapps
Items: =Sort(
    Filter(
        'Referrals & Applicants',
        !StartsWith(Lower(Coalesce('Household Status'.Value, "")), "former")
    ),
    'Unit Number'.Value,
    SortOrder.Ascending
)
```

---

**RBS-003.3: Sort Order**
- [ ] Verify referrals are sorted by Unit Number
- [ ] Check ascending order (101, 102, 103, etc.)
- [ ] Verify referrals without unit appear first or last (check behavior)

**Expected Result:** Consistent sort order by unit

---

**RBS-003.4: Color Coding**
- [ ] Verify each referral card has background color
- [ ] Check color matches unit's unit type
- [ ] Verify colors match UnitBrowseScreen palette

**Expected Result:** Consistent color coding across screens

---

**RBS-003.5: Referral Selection and Navigation**
- [ ] Click a referral card
- [ ] Verify card highlights
- [ ] Verify navigation to RNAInfoScreen
- [ ] Verify varSelectedReferral is set
- [ ] Verify varReferralUnit is set

**Expected Result:** Referral data and related unit data available on info screen

**Verification:** Add temporary label on RNAInfoScreen:
```powerapps
Label.Text: ="Referral: " & varSelectedReferral.Title &
            " | Unit: " & varReferralUnit.Unit
```

---

### RNAInfoScreen

**Test ID:** RIS-004
**Prerequisites:** At least 1 referral exists

#### Test Cases

**RIS-004.1: Screen Load**
- [ ] Select a referral from browse screen
- [ ] Navigate to RNAInfoScreen
- [ ] Verify referral details display
- [ ] Verify workflow buttons appear
- [ ] Verify form is in view mode by default

**Expected Result:** All referral data displays correctly

---

**RIS-004.2: Workflow Buttons Visibility**
- [ ] Verify "Phone Screening" button appears
- [ ] Verify "Intake" button appears
- [ ] Verify "Create Resident" button appears
- [ ] Verify "Close Referral" button appears

**Expected Result:** All four workflow buttons visible

---

**RIS-004.3: Edit Mode Toggle**
- [ ] Click edit/pencil icon
- [ ] Verify form switches to edit mode
- [ ] Modify a text field (e.g., Preferred Name)
- [ ] Click Save
- [ ] Verify save completes

**Expected Result:** Edits save successfully

**Known Issue:** No OnSuccess handler - no user feedback, no data refresh

---

**RIS-004.4: Phone Screening Navigation**
- [ ] Click "Phone Screening" button
- [ ] Verify navigation occurs (or modal opens)

**Expected Result:** Phone screening interface appears

**Note:** Phone screening screen not in current export - may not be testable

---

**RIS-004.5: Intake Navigation**
- [ ] Click "Intake" button
- [ ] Verify navigation to IntakeScreen
- [ ] Verify varIntakeReferral is set

**Expected Result:** IntakeScreen loads with referral context

---

**RIS-004.6: Create Resident Navigation**
- [ ] Click "Create Resident" button
- [ ] Verify navigation to scrConvert
- [ ] Verify varSelectedReferral is available

**Expected Result:** Convert screen loads with referral data

---

**RIS-004.7: Close Referral Navigation**
- [ ] Click "Close Referral" button
- [ ] Verify navigation to RNACloseScreen
- [ ] Verify varCloseRec is set

**Expected Result:** Close screen loads with referral context

---

### RNACloseScreen

**Test ID:** RCS-005
**Prerequisites:** At least 1 active referral exists

**Full workflow testing in [Workflow Testing](#workflow-testing) section**

#### Quick Unit Tests

**RCS-005.1: Screen Load**
- [ ] Navigate from RNAInfoScreen via "Close Referral" button
- [ ] Verify form loads
- [ ] Verify three fields present: Date, Reason, Notes
- [ ] Verify form is in edit mode

**Expected Result:** Form ready for data entry

---

**RCS-005.2: Form Fields**
- [ ] Verify Close-out Date field exists
- [ ] Verify date defaults to today or is blank
- [ ] Verify Close-out Reason is dropdown
- [ ] Verify dropdown has options
- [ ] Verify Close-out Notes is multi-line text

**Expected Result:** All fields functional

---

**RCS-005.3: Form Validation**
- [ ] Leave all fields blank
- [ ] Click Save
- [ ] Check if validation prevents save

**Expected Result:** Verify which fields are required

---

### scrConvert

**Test ID:** SCV-006
**Prerequisites:** At least 1 referral with complete data exists

**Full workflow testing in [Workflow Testing](#workflow-testing) section**

#### Quick Unit Tests

**SCV-006.1: Screen Load**
- [ ] Navigate from RNAInfoScreen via "Create Resident" button
- [ ] Verify screen loads
- [ ] Verify FormViewer shows referral data on left
- [ ] Verify input fields on right

**Expected Result:** Screen displays referral data for review

---

**SCV-006.2: Input Fields**
- [ ] Verify DatePicker1 (Lease Signing Date) present
- [ ] Verify TextInput1 (Coordinator) present
- [ ] Verify ComboBox1 (Service Providers) present
- [ ] Verify ComboBox allows multiple selections

**Expected Result:** All input controls functional

---

**SCV-006.3: Service Provider ComboBox**
- [ ] Click ComboBox1
- [ ] Verify dropdown shows service providers
- [ ] Select multiple providers
- [ ] Verify selections appear as tags/chips

**Expected Result:** Multi-select works correctly

---

**SCV-006.4: Convert Button Presence**
- [ ] Verify "Convert to Current Resident" button exists
- [ ] Verify button is enabled

**Expected Result:** Button ready for click

---

### IntakeScreen

**Test ID:** ITS-007
**Prerequisites:** At least 1 referral linked to a unit with unit type exists

#### Test Cases

**ITS-007.1: Screen Load**
- [ ] Navigate from RNAInfoScreen via "Intake" button
- [ ] Verify screen loads
- [ ] Verify varIntakeReferral is set
- [ ] Verify varIntakeUnit is set
- [ ] Verify varUnitType is set

**Expected Result:** All variables load correctly

---

**ITS-007.2: Eligibility Requirements Display**
- [ ] Verify left panel/container displays
- [ ] Check "Household Composition" section shows min/max occupancy
- [ ] Check "Income Restriction" section shows limits
- [ ] Check "Other Eligibility" section shows homelessness/disability requirements
- [ ] Verify data comes from varUnitType

**Expected Result:** Eligibility requirements from unit type display correctly

---

**ITS-007.3: Form Presence**
- [ ] Look for intake data entry form
- [ ] Check if Form2 is functional

**Expected Result (Current State):** ❌ Form is placeholder, not functional

**Status:** IN PROGRESS - Form needs to be built

---

### ResBrowseScreen

**Test ID:** RESB-008
**Prerequisites:** At least 2 current residents exist, at least 1 former resident exists

#### Test Cases

**RESB-008.1: Screen Load**
- [ ] Navigate to ResBrowseScreen
- [ ] Verify screen loads within 3 seconds
- [ ] Verify colStatusPalette loads
- [ ] Verify gallery populates

**Expected Result:** Screen loads without errors

---

**RESB-008.2: Gallery Display - Active Residents Only**
- [ ] Verify all current residents display
- [ ] Verify "Former Occupant" records do NOT appear
- [ ] Check each card shows unit number and resident name

**Expected Result:** Only active residents shown

**Filter Logic:**
```powerapps
Items: =SortByColumns(
    Filter(
        'Current Residents',
        !StartsWith(Lower(Coalesce('Household Status'.Value, "")), "former")
    ),
    "UnitNumber", Ascending
)
```

---

**RESB-008.3: Sort Order**
- [ ] Verify residents sorted by unit number
- [ ] Check ascending order

**Expected Result:** Consistent sort by unit

---

**RESB-008.4: Color Coding**
- [ ] Verify background colors match unit types
- [ ] Compare to UnitBrowseScreen colors

**Expected Result:** Consistent color palette

---

**RESB-008.5: Resident Selection and Navigation**
- [ ] Click a resident card
- [ ] Verify card highlights
- [ ] Verify navigation to ResInfoScreen
- [ ] Verify varSelectedResident is set
- [ ] Verify varResidentUnit is set

**Expected Result:** Resident and unit data available on info screen

---

### ResInfoScreen

**Test ID:** RESI-009
**Prerequisites:** At least 1 current resident exists

#### Test Cases

**RESI-009.1: Screen Load**
- [ ] Select a resident from browse screen
- [ ] Navigate to ResInfoScreen
- [ ] Verify resident details display in form
- [ ] Verify workflow buttons appear

**Expected Result:** All resident data displays correctly

---

**RESI-009.2: Edit Mode Toggle**
- [ ] Click edit icon
- [ ] Verify form switches to edit mode
- [ ] Modify a field
- [ ] Click Save
- [ ] Verify save completes

**Expected Result:** Edits save successfully

**Known Issue:** No OnSuccess handler - no user feedback

---

**RESI-009.3: Associations Button**
- [ ] Click "Associations" or similar button
- [ ] Verify navigation to scrResidentAssociations
- [ ] Verify varSelectedResident carries over

**Expected Result:** Navigation to associations screen

---

**RESI-009.4: Exit Resident Button**
- [ ] Verify "Exit Resident" or "Move Out" button exists
- [ ] Check button is enabled

**Expected Result:** Button present and clickable

**Note:** Full workflow test in [Workflow Testing](#workflow-testing)

---

### scrResidentAssociations

**Test ID:** SRA-010
**Prerequisites:** At least 1 resident with accounts and providers exists

#### Test Cases

**SRA-010.1: Screen Load**
- [ ] Navigate from ResInfoScreen
- [ ] Verify three-column layout appears
- [ ] Verify varSelectedResident is set

**Expected Result:** Screen loads with three sections

---

**SRA-010.2: Bill Pay Accounts Section**
- [ ] Verify left column shows "Accounts" or similar heading
- [ ] Verify gallery displays resident's accounts
- [ ] Check account names display correctly
- [ ] Verify click navigates to scrResidentAccounts

**Expected Result:** Accounts display and navigation works

---

**SRA-010.3: Service Providers Section**
- [ ] Verify center column shows "Service Providers" or similar
- [ ] Verify gallery displays associated providers
- [ ] Check provider names display
- [ ] Verify click opens provider details

**Expected Result:** Providers display correctly

---

**SRA-010.4: Household Members Section**
- [ ] Verify right column shows "Household Members" or similar
- [ ] Check if gallery displays members

**Expected Result (Current State):** ❌ Section is placeholder, not built

**Status:** NOT STARTED - Needs implementation

---

### scrResidentAccounts

**Test ID:** SRAC-011
**Prerequisites:** At least 1 resident exists

#### Test Cases

**SRAC-011.1: Screen Load**
- [ ] Navigate from scrResidentAssociations
- [ ] Verify form loads
- [ ] Verify varSelectedResident is pre-filled as default resident

**Expected Result:** Form ready for account creation/editing

---

**SRAC-011.2: Form Fields - Is Payable Account OFF**
- [ ] Ensure "Is Payable Account" toggle is OFF
- [ ] Verify billing-related fields are HIDDEN
- [ ] Check visible fields: Account Provider, Username, Account Number, Notes

**Expected Result:** Conditional visibility works - non-payable fields only

---

**SRAC-011.3: Form Fields - Is Payable Account ON**
- [ ] Toggle "Is Payable Account" to ON
- [ ] Verify billing fields appear:
  - [ ] Billing Frequency
  - [ ] Recurring Cost
  - [ ] Next Due Date
- [ ] Verify all fields are editable

**Expected Result:** Conditional visibility works - all fields visible

---

**SRAC-011.4: Create Account**
- [ ] Fill in required fields
- [ ] Select Account Provider
- [ ] Enter username/account number
- [ ] Click Save
- [ ] Verify account creates successfully
- [ ] Check SharePoint list for new record

**Expected Result:** Account saves successfully

---

**SRAC-011.5: Password Field**
- [ ] Check if password field exists
- [ ] If yes, verify warning message about security

**Expected Result:** Password field removed OR warning displayed

**Status:** PENDING security decision

---

### scrServiceProviders

**Test ID:** SSP-012
**Prerequisites:** At least 2 service providers exist

#### Test Cases

**SSP-012.1: Screen Load**
- [ ] Navigate to scrServiceProviders
- [ ] Verify gallery displays providers
- [ ] Verify form exists for viewing/editing

**Expected Result:** Screen loads with providers list

---

**SSP-012.2: Provider Selection**
- [ ] Click a provider in gallery
- [ ] Verify form populates with provider details
- [ ] Check all fields display: Name, Organization, Role, Contact info

**Expected Result:** Provider details display correctly

---

**SSP-012.3: Edit Provider**
- [ ] Select a provider
- [ ] Click Edit
- [ ] Modify a field
- [ ] Click Save
- [ ] Verify save succeeds

**Expected Result:** Edits save successfully

---

**SSP-012.4: Create Provider**
- [ ] Click "New Provider" or "+" button
- [ ] Fill in provider details
- [ ] Click Save
- [ ] Verify save succeeds

**Expected Result (Current State):** ❌ Create fails due to permissions

**Status:** KNOWN ISSUE - Permission problem needs resolution

---

### srcOrganizations

**Test ID:** SORG-013
**Prerequisites:** At least 2 organizations exist

#### Test Cases

**SORG-013.1: Screen Load**
- [ ] Navigate to srcOrganizations
- [ ] Verify gallery displays organizations
- [ ] Verify form exists

**Expected Result:** Screen loads with org list

---

**SORG-013.2: Organization Selection**
- [ ] Click an organization
- [ ] Verify form populates with org details
- [ ] Check all fields display

**Expected Result:** Organization details display

---

**SORG-013.3: Edit Organization**
- [ ] Select an organization
- [ ] Edit a field
- [ ] Save changes
- [ ] Verify success

**Expected Result:** Edits save successfully

---

**SORG-013.4: Create Organization**
- [ ] Click "New Organization" or "+" button
- [ ] Fill in org details
- [ ] Save
- [ ] Verify creation

**Expected Result (Current State):** ❌ Create fails due to permissions

**Status:** KNOWN ISSUE - Permission problem needs resolution

---

## Workflow Testing

### Workflow Test Template

For each workflow:
1. Define pre-conditions
2. List step-by-step actions
3. Verify expected results at each step
4. Check data consistency across related records
5. Verify final state

---

### Receive Referral Workflow

**Test ID:** WF-001
**Description:** Creating a new referral from a vacant unit
**Prerequisites:**
- [ ] At least one vacant unit exists (Unit 101)
- [ ] Unit 101 has no current referral
- [ ] Unit 101 is not occupied
- [ ] User has permission to create referrals

#### Test Steps

**Step 1: Navigate to Vacant Unit**
- [ ] Go to UnitBrowseScreen
- [ ] Toggle "Show Vacancies" to ON
- [ ] Verify Unit 101 appears
- [ ] Click Unit 101
- [ ] Verify navigation to UnitInfoScreen

**Expected:** UnitInfoScreen loads with Unit 101 data

---

**Step 2: Verify Receive Referral Button**
- [ ] Look for "Receive Referral" button on right panel
- [ ] Verify button is visible and enabled

**Expected:** Button appears and is clickable

---

**Step 3: Open Referral Form**
- [ ] Click "Receive Referral" button
- [ ] Verify modal form opens

**Expected:** Modal dialog appears with referral creation form

---

**Step 4: Check Form Pre-Population**
- [ ] Verify "Unit Number" field shows "101"
- [ ] Verify "Date Referred" field shows today's date
- [ ] Verify "Household Status" defaults (check default value)

**Expected:** Unit and date pre-filled correctly

---

**Step 5: Enter Referral Name**
- [ ] Enter "Test Referral John" in Title/Name field
- [ ] Verify text appears in field

**Expected:** Name field accepts input

---

**Step 6: Submit Form**
- [ ] Click Save/Submit button
- [ ] Watch for errors or success indicators
- [ ] Verify modal closes

**Expected:** Form saves without errors, modal closes

---

**Step 7: Verify Unit Update**
- [ ] Still on UnitInfoScreen, check right panel
- [ ] Verify "Referral Household" now shows "Test Referral John"
- [ ] Verify "Has referral" status indicates true

**Expected:** Unit now shows active referral

---

**Step 8: Verify Referral Created**
- [ ] Navigate to RNABrowseScreen
- [ ] Look for "Test Referral John" in gallery
- [ ] Verify referral appears with Unit 101

**Expected:** New referral visible in browse view

---

**Step 9: Verify SharePoint Data**
- [ ] Open SharePoint "Referrals & Applicants" list in browser
- [ ] Find "Test Referral John" record
- [ ] Check "Unit Number" field = Unit 101
- [ ] Check "Date Referred" = Today
- [ ] Open SharePoint "Units" list
- [ ] Find Unit 101 record
- [ ] Check "Referral Household" = link to John's record
- [ ] Check "Has referral" = Yes

**Expected:** SharePoint data consistent with app

---

**Step 10: Verify Button Now Hidden**
- [ ] Navigate back to Unit 101's UnitInfoScreen
- [ ] Verify "Receive Referral" button is now hidden or disabled

**Expected:** Button no longer appears (unit has referral)

---

#### Cleanup
- [ ] Delete "Test Referral John" from SharePoint
- [ ] Clear Unit 101's "Referral Household" field
- [ ] Set Unit 101's "Has referral" to No

---

### Close Referral Workflow

**Test ID:** WF-002
**Description:** Closing out a referral that declined or withdrew
**Prerequisites:**
- [ ] At least one active referral exists (Test Referral Jane, Unit 102)
- [ ] Referral has "Household Status" = "Referral, New"
- [ ] Unit 102 has "Referral Household" = Jane's record
- [ ] Unit 102 has "Has referral" = Yes

#### Test Steps

**Step 1: Navigate to Referral**
- [ ] Go to RNABrowseScreen
- [ ] Find "Test Referral Jane"
- [ ] Click on Jane's card
- [ ] Verify navigation to RNAInfoScreen

**Expected:** RNAInfoScreen displays Jane's details

---

**Step 2: Open Close Workflow**
- [ ] Click "Close Referral" workflow button
- [ ] Verify navigation to RNACloseScreen
- [ ] Verify form loads with three fields

**Expected:** Close form ready for input

---

**Step 3: Enter Close-Out Information**
- [ ] Verify "Close-out Date" = today (or change if needed)
- [ ] Select "Close-out Reason" from dropdown (e.g., "Declined")
- [ ] Enter "Close-out Notes": "Test - Declined due to location preference"

**Expected:** All fields accept input

---

**Step 4: Submit Close Form**
- [ ] Click Save/Submit
- [ ] Watch for success/error messages
- [ ] Verify navigation back to RNABrowseScreen

**Expected:** Form saves, returns to browse screen

---

**Step 5: Verify Referral Status Updated**
- [ ] Open SharePoint "Referrals & Applicants" list
- [ ] Find Jane's record
- [ ] Check "Household Status" = "Former Referral"
- [ ] Check "Close-out Date" = today
- [ ] Check "Close-out Reason" = "Declined"
- [ ] Check "Close-out Notes" = entered text
- [ ] Check "Unit Number" = blank/empty

**Expected:** Status changed to "Former", unit cleared, close data saved

---

**Step 6: Verify Referral Removed from Browse View**
- [ ] Go to RNABrowseScreen
- [ ] Verify "Test Referral Jane" does NOT appear

**Expected:** Former referrals filtered out

---

**Step 7: Verify Unit Updated**
- [ ] Open SharePoint "Units" list
- [ ] Find Unit 102
- [ ] Check "Referral Household" = blank/empty
- [ ] Check "Has referral" = No

**Expected:** Unit cleared, available for new referral

---

**Step 8: Verify Unit Browse View**
- [ ] Go to UnitBrowseScreen
- [ ] Toggle "Show Vacancies" ON
- [ ] Verify Unit 102 appears (now available)
- [ ] Click Unit 102
- [ ] Verify "Receive Referral" button appears

**Expected:** Unit 102 ready for new referral

---

#### Cleanup
- [ ] Change Jane's "Household Status" back to "Referral, New" (if retesting)
- [ ] Or delete Jane's record if no longer needed

---

### Convert to Resident Workflow

**Test ID:** WF-003
**Description:** Converting an approved applicant to current resident
**Prerequisites:**
- [ ] Referral "Test Applicant Mark" exists
- [ ] Mark is assigned to Unit 103
- [ ] Mark's record has complete data:
  - [ ] Preferred Name
  - [ ] Phone, Email
  - [ ] Date of Birth
  - [ ] Veteran Status, Disability, Benefits
  - [ ] Monthly Income, HMIS Record Number
- [ ] Unit 103 has "Referral Household" = Mark
- [ ] Unit 103 has "Has referral" = Yes
- [ ] Unit 103 is not occupied

#### Test Steps

**Step 1: Navigate to Referral**
- [ ] Go to RNABrowseScreen
- [ ] Find "Test Applicant Mark"
- [ ] Click on Mark's card
- [ ] Verify RNAInfoScreen displays Mark's details

**Expected:** Mark's referral info displays

---

**Step 2: Open Convert Workflow**
- [ ] Click "Create Resident" workflow button
- [ ] Verify navigation to scrConvert
- [ ] Verify screen loads

**Expected:** Convert screen appears

---

**Step 3: Review Referral Data**
- [ ] Verify left panel (FormViewer) shows Mark's data
- [ ] Check all fields display correctly:
  - [ ] Title, Preferred Name, Language
  - [ ] Phone, Email
  - [ ] Date of Birth
  - [ ] Veteran Status, Disability, Benefits
  - [ ] Income sources, HMIS number
  - [ ] Unit Number = 103

**Expected:** All referral data visible for review

---

**Step 4: Enter Move-In Details**
- [ ] Select "Lease Signing Date" from DatePicker1 (e.g., today)
- [ ] Enter "Assigned Coordinator" in TextInput1 (e.g., "Jane Smith")
- [ ] Click ComboBox1 (Associated Providers)
- [ ] Select 1-2 service providers
- [ ] Verify selections appear

**Expected:** All input fields accept data

---

**Step 5: Convert to Resident**
- [ ] Click "Convert to Current Resident" button
- [ ] Watch for success notification
- [ ] Verify navigation to ResInfoScreen
- [ ] Verify form is in edit mode

**Expected:** Success notification appears, navigation to new resident

---

**Step 6: Verify New Resident Created**
- [ ] On ResInfoScreen, verify all data copied correctly:
  - [ ] Title = "Test Applicant Mark"
  - [ ] Preferred Name = Mark's preferred name
  - [ ] Phone, Email match
  - [ ] Date of Birth matches
  - [ ] Veteran Status matches
  - [ ] Disability matches
  - [ ] Benefits match
  - [ ] Income data matches
  - [ ] Unit Number = 103
  - [ ] Lease Signing Date = date selected
  - [ ] Associated Providers = providers selected

**Expected:** All data transferred correctly

---

**Step 7: Verify Resident in Browse View**
- [ ] Navigate to ResBrowseScreen
- [ ] Find "Test Applicant Mark"
- [ ] Verify Mark appears with Unit 103
- [ ] Verify color coding matches Unit 103's type

**Expected:** New resident visible in browse

---

**Step 8: Verify SharePoint - Current Residents**
- [ ] Open SharePoint "Current Residents" list
- [ ] Find Mark's record
- [ ] Verify all fields populated correctly
- [ ] Check "Household Status" = "Occupant, Current"
- [ ] Check "Unit Number" = Unit 103 (lookup)
- [ ] Check "Associated Providers" = selected providers (multi-lookup)
- [ ] Check "Lease Signing Date" = selected date

**Expected:** Resident record complete in SharePoint

---

**Step 9: Verify Referral Removed**
- [ ] Go to RNABrowseScreen
- [ ] Verify "Test Applicant Mark" does NOT appear
- [ ] Open SharePoint "Referrals & Applicants" list
- [ ] Verify Mark's referral record is DELETED (not just status changed)

**Expected:** Referral completely removed from list

---

**Step 10: Verify Unit Updated**
- [ ] Open SharePoint "Units" list
- [ ] Find Unit 103
- [ ] Check "Occupant Household" = link to Mark's Current Residents record
- [ ] Check "Referral Household" = blank/empty
- [ ] Check "Has referral" = No
- [ ] Check "Is occupied" = Yes

**Expected:** Unit now marked as occupied by Mark

---

**Step 11: Verify Unit Browse View**
- [ ] Go to UnitBrowseScreen
- [ ] Toggle "Show Vacancies" ON
- [ ] Verify Unit 103 does NOT appear (occupied)
- [ ] Toggle OFF
- [ ] Verify Unit 103 appears as occupied

**Expected:** Unit 103 shows as occupied in all views

---

#### Edge Case Tests for Convert Workflow

**Test 3A: Referral with Minimal Data**
- [ ] Create referral with only name and unit
- [ ] No phone, email, or other optional fields
- [ ] Attempt to convert
- [ ] Verify resident created successfully
- [ ] Check optional fields are blank in resident record

**Expected:** Conversion works with minimal data

---

**Test 3B: Referral with No Service Providers**
- [ ] Create referral with no associated providers
- [ ] Don't select any in ComboBox during conversion
- [ ] Verify conversion succeeds
- [ ] Check resident has empty Associated Providers

**Expected:** Conversion works without providers

---

**Test 3C: Referral with All Fields Populated**
- [ ] Create referral with every field filled
- [ ] Multiple disabilities, benefits
- [ ] Multiple service providers
- [ ] Convert to resident
- [ ] Verify all data transfers

**Expected:** All fields copy correctly, including multi-select

---

#### Cleanup
- [ ] Delete Mark from "Current Residents" list
- [ ] Update Unit 103: clear Occupant, set Is occupied = No

---

### Exit Resident Workflow

**Test ID:** WF-004
**Description:** Exiting a resident who is moving out
**Prerequisites:**
- [ ] Resident "Test Resident Sarah" exists
- [ ] Sarah occupies Unit 104
- [ ] Unit 104 has "Occupant Household" = Sarah
- [ ] Unit 104 has "Is occupied" = Yes

#### Test Steps

**Step 1: Navigate to Resident**
- [ ] Go to ResBrowseScreen
- [ ] Find "Test Resident Sarah"
- [ ] Click on Sarah's card
- [ ] Verify ResInfoScreen displays

**Expected:** Sarah's resident info displays

---

**Step 2: Initiate Exit**
- [ ] Click "Exit Resident" or "Move Out" button
- [ ] Verify confirmation dialog appears (if implemented)

**Expected:** Exit action initiated

---

**Step 3: Confirm Exit**
- [ ] If confirmation dialog: Click "Yes" or "Confirm"
- [ ] Watch for success/error messages
- [ ] Verify navigation away from resident

**Expected:** Exit completes

---

**Step 4: Verify Resident Status Updated**
- [ ] Open SharePoint "Current Residents" list
- [ ] Find Sarah's record
- [ ] Check "Household Status" = "Former Occupant"

**Expected:** Status changed to "Former"

---

**Step 5: Verify Archive Created**
- [ ] Open SharePoint "Archived Records" list
- [ ] Find record for Sarah
- [ ] Check "Archive Date" = today
- [ ] Check "Archive Reason" populated
- [ ] Check "Linked Resident" = Sarah's record

**Expected:** Archive record created

---

**Step 6: Verify Resident Removed from Browse View**
- [ ] Go to ResBrowseScreen
- [ ] Verify "Test Resident Sarah" does NOT appear

**Expected:** Former residents filtered out

---

**Step 7: Verify Unit Updated** ⚠️ **KNOWN ISSUE**
- [ ] Open SharePoint "Units" list
- [ ] Find Unit 104
- [ ] Check "Occupant Household" = blank/empty **← Expected but may fail**
- [ ] Check "Is occupied" = No **← Expected but may fail**
- [ ] Check "Date Available" = today **← Expected but may fail**

**Expected Result:** Unit cleared and available

**Current Behavior:** ❌ Unit may not be updated (known bug)

**Status:** NEEDS FIX - Exit workflow doesn't update unit

---

**Step 8: Verify Unit Appears as Vacant**
- [ ] Go to UnitBrowseScreen
- [ ] Toggle "Show Vacancies" ON
- [ ] Verify Unit 104 appears **← May fail if unit not updated**
- [ ] Click Unit 104
- [ ] Verify "Receive Referral" button appears **← May fail**

**Expected:** Unit 104 available for new referral

**Current Behavior:** ❌ May still show as occupied

---

#### Cleanup
- [ ] Delete Sarah from "Current Residents" (if testing complete)
- [ ] Delete archive record for Sarah
- [ ] Manually update Unit 104 if needed:
  - [ ] Clear "Occupant Household"
  - [ ] Set "Is occupied" = No
  - [ ] Set "Date Available" = Today

---

### Intake Workflow

**Test ID:** WF-005
**Description:** Completing intake for an applicant
**Prerequisites:**
- [ ] Referral "Test Referral Tom" exists
- [ ] Tom linked to Unit 105 with specific unit type
- [ ] Unit type has eligibility requirements configured

#### Test Steps

**Step 1: Navigate to Referral**
- [ ] Go to RNABrowseScreen
- [ ] Find "Test Referral Tom"
- [ ] Click Tom's card

**Expected:** RNAInfoScreen displays

---

**Step 2: Open Intake Workflow**
- [ ] Click "Intake" workflow button
- [ ] Verify navigation to IntakeScreen

**Expected:** IntakeScreen loads

---

**Step 3: Review Eligibility Requirements**
- [ ] Verify left panel displays:
  - [ ] Household Composition (min/max occupancy)
  - [ ] Income Restriction (limits and basis)
  - [ ] Other Eligibility (homelessness, disability)
- [ ] Verify data matches Tom's unit type

**Expected:** Requirements display correctly from unit type

---

**Step 4: Complete Intake Form** ⚠️ **NOT IMPLEMENTED**
- [ ] Look for intake data entry form
- [ ] Check if form exists with fields

**Expected Result:** Form with intake fields

**Current Behavior:** ❌ Form is placeholder, not functional

**Status:** IN PROGRESS - Form needs to be built

---

#### Cannot Complete Test - Workflow Not Finished

**Remaining Steps (Once Implemented):**
5. Fill in intake fields
6. Check off documentation items
7. Save intake form
8. Verify referral status updates
9. Verify "Intake Completed" flag set
10. Verify navigation back to referral

---

### Phone Screening Workflow

**Test ID:** WF-006
**Description:** Completing phone screening for a referral
**Prerequisites:**
- [ ] Referral exists
- [ ] Phone screening form exists

#### Test Steps

**Cannot Test:** Phone screening screen not in current export

**Status:** UNKNOWN - May exist in different version

---

## Integration Testing

### Integration Test Overview

Integration tests verify that data stays consistent when multiple related records are updated through workflows.

---

### Integration Test 1: Receive Referral → Close Referral → Receive New Referral

**Test ID:** INT-001
**Purpose:** Verify unit can cycle through referrals correctly

#### Test Steps

**Step 1: Start with Vacant Unit**
- [ ] Unit 110 is vacant, no referral

**Step 2: Receive First Referral**
- [ ] Create "Referral A" for Unit 110
- [ ] Verify Unit 110 shows Referral A
- [ ] Verify "Has referral" = Yes

**Step 3: Close First Referral**
- [ ] Close Referral A with reason "Declined"
- [ ] Verify Referral A status = "Former Referral"
- [ ] Verify Unit 110 cleared
- [ ] Verify "Has referral" = No

**Step 4: Receive Second Referral**
- [ ] Create "Referral B" for Unit 110
- [ ] Verify Unit 110 shows Referral B (not A)
- [ ] Verify "Has referral" = Yes

**Expected:** Unit correctly tracks current referral, clears when closed

---

### Integration Test 2: Referral → Convert → Exit → New Referral

**Test ID:** INT-002
**Purpose:** Complete participant lifecycle

#### Test Steps

**Step 1: Referral Created**
- [ ] Create "Referral C" for Unit 111
- [ ] Unit 111: Referral Household = C, Has referral = Yes, Is occupied = No

**Step 2: Convert to Resident**
- [ ] Convert Referral C to Current Resident
- [ ] Verify Unit 111: Occupant Household = C, Referral Household = blank, Has referral = No, Is occupied = Yes
- [ ] Verify Referral C deleted from Referrals list
- [ ] Verify Resident C appears in Current Residents list

**Step 3: Exit Resident**
- [ ] Exit Resident C
- [ ] Verify Resident C status = "Former Occupant"
- [ ] Verify Unit 111: Occupant Household = blank, Is occupied = No, Date Available = today
- [ ] Verify archive created for Resident C

**Step 4: New Referral**
- [ ] Create "Referral D" for Unit 111
- [ ] Verify Unit 111 accepts new referral
- [ ] Verify Unit 111: Referral Household = D, Has referral = Yes

**Expected:** Complete lifecycle works, unit ready for reuse

**Known Issue:** Step 3 may fail - exit workflow doesn't update unit

---

### Integration Test 3: Service Provider Association Continuity

**Test ID:** INT-003
**Purpose:** Verify service providers carry from referral to resident

#### Test Steps

**Step 1: Create Referral with Providers**
- [ ] Create referral with 2 associated service providers
- [ ] Verify providers display on RNAInfoScreen

**Step 2: Convert to Resident**
- [ ] During conversion, select same 2 providers in ComboBox
- [ ] Complete conversion

**Step 3: Verify Provider Association**
- [ ] Open resident's ResInfoScreen
- [ ] Navigate to scrResidentAssociations
- [ ] Verify both providers appear in Service Providers section

**Step 4: Add Provider to Resident**
- [ ] Add a 3rd service provider to resident
- [ ] Verify all 3 now display

**Expected:** Providers persist through conversion, can be managed independently

---

### Integration Test 4: Unit Type Eligibility Display

**Test ID:** INT-004
**Purpose:** Verify unit type requirements display correctly throughout

#### Test Steps

**Step 1: Configure Unit Type**
- [ ] Select a unit type (e.g., "LTH Housing Support")
- [ ] Note its requirements: income limit, homelessness, disability

**Step 2: Create Unit with Type**
- [ ] Create/select Unit 112 with "LTH Housing Support" type
- [ ] Open UnitInfoScreen for Unit 112
- [ ] Verify eligibility section shows correct requirements

**Step 3: Create Referral for Unit**
- [ ] Create referral for Unit 112
- [ ] Open IntakeScreen for referral
- [ ] Verify eligibility requirements match Unit 112's type

**Step 4: Change Unit Type**
- [ ] Change Unit 112's type to "Section 8 PBV"
- [ ] Reopen UnitInfoScreen
- [ ] Verify eligibility requirements updated
- [ ] Open IntakeScreen for existing referral
- [ ] Verify requirements updated there too

**Expected:** Unit type requirements dynamically displayed, always current

---

## Edge Case Testing

### Edge Case Test Overview

Edge cases test unusual or boundary conditions that might cause errors.

---

### Edge Case 1: Null/Blank Fields

**Test ID:** EDGE-001

**Test 1A: Referral with Blank Optional Fields**
- [ ] Create referral with only name and unit
- [ ] Leave phone, email, preferences blank
- [ ] View on RNAInfoScreen
- [ ] Verify no errors with blank fields
- [ ] Convert to resident
- [ ] Verify conversion succeeds

**Expected:** App handles blank optional fields gracefully

---

**Test 1B: Unit with No Unit Type**
- [ ] Create unit without assigning unit type
- [ ] Open UnitInfoScreen
- [ ] Check eligibility section
- [ ] Create referral for this unit
- [ ] Open IntakeScreen

**Expected:** App handles missing unit type (shows N/A or error message)

---

**Test 1C: Referral with No Unit Assignment**
- [ ] Create referral without unit number
- [ ] View on RNAInfoScreen
- [ ] Attempt to convert to resident

**Expected:** Conversion should fail or require unit selection

---

### Edge Case 2: Deleted Lookup References

**Test ID:** EDGE-002

**Test 2A: Referral with Deleted Unit**
- [ ] Create referral linked to Unit 120
- [ ] Delete Unit 120 from Units list
- [ ] Open RNABrowseScreen
- [ ] Click referral
- [ ] Check unit display

**Expected:** App handles missing lookup gracefully (shows "N/A" or error)

---

**Test 2B: Resident with Deleted Unit**
- [ ] Create resident in Unit 121
- [ ] Delete Unit 121
- [ ] Open ResBrowseScreen
- [ ] Click resident
- [ ] Check unit display

**Expected:** Resident displays, unit shows as unavailable

---

**Test 2C: Account with Deleted Provider**
- [ ] Create account linked to Account Provider X
- [ ] Delete Provider X from Account Providers list
- [ ] Open scrResidentAccounts
- [ ] View account

**Expected:** Account displays, provider shows as unavailable

---

### Edge Case 3: Concurrent Users

**Test ID:** EDGE-003

**Test 3A: Two Users Viewing Same Referral**
- [ ] User A opens Referral M on RNAInfoScreen
- [ ] User B opens Referral M on RNAInfoScreen
- [ ] User A edits phone number, saves
- [ ] User B edits email, saves
- [ ] Refresh both screens
- [ ] Check final values

**Expected:** Last save wins, both changes may not persist

---

**Test 3B: Two Users Converting Same Referral**
- [ ] User A starts converting Referral N
- [ ] User B starts converting Referral N (simultaneously)
- [ ] User A completes conversion first
- [ ] User B attempts to convert
- [ ] Check result

**Expected:** User B's conversion should fail (referral deleted)

**Likely Behavior:** May create duplicate resident or error

---

**Test 3C: Two Users Receiving Referral for Same Unit**
- [ ] Unit 130 is vacant, no referral
- [ ] User A clicks "Receive Referral" for Unit 130
- [ ] User B clicks "Receive Referral" for Unit 130 (before A saves)
- [ ] Both enter different names
- [ ] Both click Save

**Expected:** One should succeed, one should fail or show warning

**Likely Behavior:** May create two referrals for same unit

---

### Edge Case 4: Special Characters in Names

**Test ID:** EDGE-004

**Test 4A: Apostrophe in Name**
- [ ] Create referral: "O'Brien, John"
- [ ] Verify display in gallery
- [ ] Open detail screen
- [ ] Edit and save
- [ ] Convert to resident

**Expected:** Apostrophe handled correctly throughout

---

**Test 4B: Hyphenated Names**
- [ ] Create referral: "Smith-Jones, Mary-Anne"
- [ ] Test all workflows

**Expected:** Hyphens preserved correctly

---

**Test 4C: Non-English Characters**
- [ ] Create referral: "Müller, François José"
- [ ] Test display and workflows

**Expected:** Unicode characters handled correctly

---

### Edge Case 5: Maximum Field Lengths

**Test ID:** EDGE-005

**Test 5A: Very Long Name**
- [ ] Create referral with 100+ character name
- [ ] Check display in gallery (truncation?)
- [ ] Check detail view

**Expected:** Name displays or truncates gracefully

---

**Test 5B: Very Long Notes**
- [ ] Enter 5000+ characters in Close-out Notes
- [ ] Save
- [ ] Reopen and verify

**Expected:** Long text saved and retrieved correctly

---

### Edge Case 6: Date Boundaries

**Test ID:** EDGE-006

**Test 6A: Future Dates**
- [ ] Set "Lease Signing Date" to 10 years in future
- [ ] Complete conversion
- [ ] Check if accepted

**Expected:** Define whether future dates allowed

---

**Test 6B: Past Dates**
- [ ] Set "Close-out Date" to 10 years in past
- [ ] Save
- [ ] Check if accepted

**Expected:** Past dates likely acceptable

---

**Test 6C: Date Format Consistency**
- [ ] Create records with various date fields
- [ ] Check display format across screens
- [ ] Verify consistent format (MM/DD/YYYY or DD/MM/YYYY)

**Expected:** Consistent date formatting

---

## Performance Testing

### Performance Test Overview

Performance tests verify the app remains responsive with realistic data volumes.

---

### Performance Test 1: Large Dataset Load Times

**Test ID:** PERF-001

**Test 1A: 100+ Units**
- [ ] Create or ensure 100+ units in Units list
- [ ] Navigate to UnitBrowseScreen
- [ ] Time how long gallery takes to load
- [ ] Toggle "Show Vacancies"
- [ ] Time filter application

**Expected:** Load time < 5 seconds, filter < 2 seconds

**Metric:** ___ seconds to load, ___ seconds to filter

---

**Test 1B: 100+ Referrals**
- [ ] Ensure 100+ referrals in list
- [ ] Navigate to RNABrowseScreen
- [ ] Time gallery load

**Expected:** Load time < 5 seconds

**Metric:** ___ seconds to load

---

**Test 1C: 100+ Residents**
- [ ] Ensure 100+ residents
- [ ] Navigate to ResBrowseScreen
- [ ] Time gallery load

**Expected:** Load time < 5 seconds

**Metric:** ___ seconds to load

---

### Performance Test 2: Delegation Warnings

**Test ID:** PERF-002

**Test 2A: Check for Delegation Warnings**
- [ ] In Power Apps Studio, open app
- [ ] Enable formula bar
- [ ] Look for yellow warning triangles on formulas
- [ ] Document any delegation warnings found

**Expected:** No delegation warnings (or only on acceptable formulas)

**Warnings Found:**
- Screen: ___________
- Control: ___________
- Formula: ___________

---

**Test 2B: Test Delegation with Large Dataset**
- [ ] Create 600+ records (beyond delegation limit)
- [ ] Test filters and sorts
- [ ] Verify all records accessible

**Expected:** All records accessible despite delegation limit

**Result:** ___ records accessible

---

### Performance Test 3: Lookup Performance

**Test ID:** PERF-003

**Test 3A: Gallery with Inline Lookups**
- [ ] Open ResBrowseScreen (lookups up unit for each resident)
- [ ] With 50+ residents
- [ ] Time gallery render

**Expected:** Acceptable render time (< 3 seconds)

**Metric:** ___ seconds

---

**Test 3B: Repeated Screen Navigation**
- [ ] Navigate UnitBrowseScreen → UnitInfoScreen
- [ ] Back to UnitBrowseScreen
- [ ] Repeat 10 times
- [ ] Check for slowdown

**Expected:** No cumulative slowdown

**Observation:** ___________

---

### Performance Test 4: Form Save Performance

**Test ID:** PERF-004

**Test 4A: Simple Form Save**
- [ ] Edit text field on resident
- [ ] Time save operation

**Expected:** Save < 2 seconds

**Metric:** ___ seconds

---

**Test 4B: Complex Form Save**
- [ ] Edit multiple fields including lookups
- [ ] Time save operation

**Expected:** Save < 3 seconds

**Metric:** ___ seconds

---

**Test 4C: Workflow Completion Time**
- [ ] Time complete Convert to Resident workflow
- [ ] From button click to new resident screen load

**Expected:** < 5 seconds

**Metric:** ___ seconds

---

## User Acceptance Testing

### UAT Overview

User Acceptance Testing involves actual staff members using the app in realistic scenarios.

---

### UAT Test 1: Daily Workflow Simulation

**Test ID:** UAT-001
**Participant:** Staff Member 1
**Scenario:** Process a new referral received today

#### Steps
1. [ ] Participant receives referral information (provide sample data)
2. [ ] Ask participant to create referral in app
3. [ ] Observer watches without helping (unless stuck)
4. [ ] Participant completes phone screening
5. [ ] Participant reviews unit eligibility requirements
6. [ ] Participant initiates intake (if available)

#### Observations
- Time to complete: ___ minutes
- Errors encountered: ___________
- Usability issues noted: ___________
- Participant feedback: ___________

---

### UAT Test 2: Move-In Process

**Test ID:** UAT-002
**Participant:** Staff Member 2
**Scenario:** Convert approved applicant to resident

#### Steps
1. [ ] Participant receives move-in details (provide sample)
2. [ ] Ask participant to convert applicant to resident
3. [ ] Participant completes conversion
4. [ ] Participant verifies new resident record
5. [ ] Participant checks unit now shows as occupied

#### Observations
- Time to complete: ___ minutes
- Data accuracy: ___________
- Errors encountered: ___________
- Participant feedback: ___________

---

### UAT Test 3: Bill Pay Management

**Test ID:** UAT-003
**Participant:** Staff Member 3 (who manages bill pay)
**Scenario:** Add and track utility account for resident

#### Steps
1. [ ] Participant selects resident
2. [ ] Navigates to accounts section
3. [ ] Adds new utility account
4. [ ] Enters billing details
5. [ ] Marks as payable account
6. [ ] Reviews account information

#### Observations
- Time to complete: ___ minutes
- Ease of use rating (1-10): ___
- Participant feedback: ___________

---

### UAT Test 4: Exploration and Discovery

**Test ID:** UAT-004
**Participant:** Staff Member 4 (new to app)
**Scenario:** Explore app without specific task

#### Steps
1. [ ] Give participant 15 minutes to explore freely
2. [ ] Ask them to narrate what they're doing
3. [ ] Observe what they click, where they go
4. [ ] Note any confusion or questions

#### Observations
- Navigation path: ___________
- Features discovered easily: ___________
- Features not found: ___________
- Questions asked: ___________
- Overall impression: ___________

---

### UAT Test 5: Error Recovery

**Test ID:** UAT-005
**Participant:** Staff Member 5
**Scenario:** Intentionally cause error, observe recovery

#### Steps
1. [ ] Ask participant to create referral
2. [ ] Have them leave required field blank
3. [ ] Attempt to save
4. [ ] Observe how they respond to error
5. [ ] Note if error message is clear

#### Observations
- Did error message appear? ___
- Was error message clear? ___
- Did participant understand how to fix? ___
- Time to recover: ___ minutes

---

## Regression Testing

### Regression Test Overview

Regression tests verify that new changes don't break existing functionality.

**Run regression tests after:**
- Any code changes
- Formula modifications
- New feature additions
- Bug fixes

---

### Regression Test Checklist

**Run this checklist after every significant change:**

#### Core Navigation
- [ ] UnitBrowseScreen → UnitInfoScreen
- [ ] RNABrowseScreen → RNAInfoScreen
- [ ] ResBrowseScreen → ResInfoScreen
- [ ] All "Back" buttons work
- [ ] All navigation preserves context (variables set)

#### Data Display
- [ ] Unit gallery displays with correct colors
- [ ] Referral gallery displays with correct data
- [ ] Resident gallery displays with correct data
- [ ] Forms load with correct data
- [ ] Lookup fields display properly (show Value, not Id)

#### Filters and Sorts
- [ ] "Show Vacancies" toggle works
- [ ] "Former" records filtered from browse screens
- [ ] Galleries sorted correctly (unit number, etc.)
- [ ] Search/filter controls work (if implemented)

#### Form Operations
- [ ] Create new records works
- [ ] Edit existing records works
- [ ] Delete records works (if applicable)
- [ ] Form validation works
- [ ] Required fields enforced

#### Workflows
- [ ] Receive Referral complete workflow
- [ ] Close Referral complete workflow
- [ ] Convert to Resident complete workflow
- [ ] Exit Resident workflow (partial - unit update may fail)

#### Data Consistency
- [ ] Creating referral updates unit
- [ ] Closing referral clears unit
- [ ] Converting creates resident and updates unit
- [ ] Exiting resident archives (but may not clear unit)

---

### Regression Test: After Adding OnSuccess Handlers

**Test ID:** REG-001
**Run After:** Adding OnSuccess handlers to forms

#### Verify
- [ ] RNAInfoScreen form save shows notification
- [ ] ResInfoScreen form save shows notification
- [ ] UnitInfoScreen form save shows notification
- [ ] Data refreshes after save (changes visible immediately)
- [ ] Variables refresh after save
- [ ] No errors introduced by OnSuccess code

---

### Regression Test: After Fixing Exit Resident Workflow

**Test ID:** REG-002
**Run After:** Adding unit update logic to exit workflow

#### Verify
- [ ] Exit resident still archives resident
- [ ] Exit resident now clears unit's Occupant Household
- [ ] Exit resident now sets Is occupied = false
- [ ] Exit resident now sets Date Available
- [ ] Unit appears in vacancy list after exit
- [ ] Unit's "Receive Referral" button now appears
- [ ] No errors introduced

---

### Regression Test: After Building Intake Form

**Test ID:** REG-003
**Run After:** Completing IntakeScreen form implementation

#### Verify
- [ ] Eligibility requirements still display correctly
- [ ] New form fields appear
- [ ] Form save works
- [ ] Referral status updates
- [ ] Navigation returns to RNAInfoScreen
- [ ] Existing IntakeScreen variables still set correctly
- [ ] No impact on other screens

---

### Regression Test: After Adding Household Members

**Test ID:** REG-004
**Run After:** Building household members section

#### Verify
- [ ] scrResidentAssociations loads correctly
- [ ] Accounts section still works
- [ ] Providers section still works
- [ ] New household members section displays
- [ ] Add/edit/delete household members works
- [ ] No errors on screen load

---

## Test Data Management

### Test Data Best Practices

1. **Use Consistent Naming**
   - Prefix all test records: "TEST - "
   - Example: "TEST - John Doe", "TEST - Unit 999"

2. **Document Test Data**
   - Keep list of test records created
   - Note which tests use which data

3. **Clean Up After Testing**
   - Delete test records when done
   - Reset modified production data

4. **Separate Test Environment**
   - Ideally, use separate SharePoint site for testing
   - If using production, be extra careful

---

### Test Data Creation Script

**Creating Complete Test Dataset:**

1. **Units (create 5 test units):**
   - TEST - Unit 901: Vacant, no referral (LTH Housing Support)
   - TEST - Unit 902: Vacant, has referral (Section 8 PBV)
   - TEST - Unit 903: Occupied (Ramsey CoC)
   - TEST - Unit 904: Vacant, given notice (HUD-VASH)
   - TEST - Unit 905: Offline (LTH Housing Support)

2. **Referrals (create 3 test referrals):**
   - TEST - Referral Alice: Unit 902, Status "Referral, New"
   - TEST - Applicant Bob: Unit 901, Status "Applicant, Approved"
   - TEST - Former Charlie: No unit, Status "Former Referral"

3. **Residents (create 2 test residents):**
   - TEST - Resident Diana: Unit 903, Status "Occupant, Current"
   - TEST - Former Evan: No unit, Status "Former Occupant"

4. **Service Providers (create 2):**
   - TEST - Provider Sam: Agency "TEST Agency X"
   - TEST - Provider Taylor: Agency "TEST Agency Y"

5. **Organizations (create 1):**
   - TEST - Agency X: Services "Mental Health, Case Management"

6. **Accounts (create 1):**
   - TEST - Xcel Energy Account: For Diana, Is Payable = Yes

---

### Test Data Cleanup Checklist

**After completing test session:**

- [ ] Delete all records starting with "TEST - " from:
  - [ ] Referrals & Applicants
  - [ ] Current Residents
  - [ ] Units
  - [ ] Service Providers
  - [ ] Agencies
  - [ ] Account Information
  - [ ] Archived Records

- [ ] Verify no orphaned lookup references

- [ ] Reset any modified production records

---

## Bug Reporting Template

### Bug Report Template

**Use this template when reporting issues:**

---

**Bug ID:** BUG-[DATE]-[NUMBER]
**Reported By:** [Your Name]
**Date:** [Date]
**Severity:** Critical / High / Medium / Low

#### Summary
Brief description of the issue (1 sentence)

#### Environment
- App Version: [if known]
- Browser: [Chrome/Edge/Firefox] Version [X.X]
- User: [test account or production]
- Device: Desktop / Tablet / Phone

#### Steps to Reproduce
1. Step 1
2. Step 2
3. Step 3

#### Expected Behavior
What should happen

#### Actual Behavior
What actually happens

#### Screenshots/Video
[Attach if available]

#### Workaround
[If known workaround exists]

#### Additional Notes
Any other relevant information

---

### Example Bug Report

**Bug ID:** BUG-2025-10-24-001
**Reported By:** Jane Tester
**Date:** October 24, 2025
**Severity:** High

#### Summary
Exit resident workflow does not update unit record, unit remains marked as occupied

#### Environment
- App Version: Latest (post-convert workflow completion)
- Browser: Edge Version 118
- User: test@organization.com
- Device: Desktop

#### Steps to Reproduce
1. Navigate to ResBrowseScreen
2. Select resident "TEST - Resident Diana" (occupies Unit 903)
3. Click resident card to open ResInfoScreen
4. Click "Exit Resident" button
5. Confirm exit (if prompted)
6. Open SharePoint Units list
7. Check Unit 903

#### Expected Behavior
- Unit 903 "Occupant Household" field should be blank/empty
- Unit 903 "Is occupied" should be No
- Unit 903 "Date Available" should be today's date
- Unit 903 should appear in vacancy toggle on UnitBrowseScreen

#### Actual Behavior
- Unit 903 "Occupant Household" still shows Diana
- Unit 903 "Is occupied" still Yes
- Unit 903 "Date Available" not updated
- Unit 903 does not appear as vacant

#### Screenshots/Video
[Screenshot of Unit 903 record in SharePoint showing Diana still linked]

#### Workaround
Manually edit Unit 903 in SharePoint:
1. Clear "Occupant Household" field
2. Set "Is occupied" to No
3. Set "Date Available" to today

#### Additional Notes
Resident record correctly updated to "Former Occupant" and archive created. Issue is only with unit update. See Technical Documentation section on Exit Resident Workflow for recommended fix.

---

## Test Session Summary Template

### Test Session Summary

**Date:** [Date]
**Tester:** [Name]
**Duration:** [Hours]
**Focus:** [What was tested]

#### Tests Executed
- [ ] Test ID: [XXX-000] - [Result: Pass/Fail]
- [ ] Test ID: [XXX-000] - [Result: Pass/Fail]
- [ ] Test ID: [XXX-000] - [Result: Pass/Fail]

#### Summary
- Total Tests Run: [#]
- Tests Passed: [#]
- Tests Failed: [#]
- Bugs Found: [#]

#### Bugs Discovered
- BUG-[ID]: [Brief description] - Severity: [Level]
- BUG-[ID]: [Brief description] - Severity: [Level]

#### Notes
[Any additional observations or recommendations]

#### Next Steps
[What should be tested next or what fixes are needed]

---

## Conclusion

This testing guide provides comprehensive coverage for the Upper Post Clearview application. Use the progressive debugging approach throughout - start small, build up, isolate problems quickly.

**Testing Priority Order:**
1. Core workflows (Receive, Close, Convert)
2. Browse and display screens
3. Form operations
4. Edge cases
5. Performance
6. User acceptance

**Remember:** Testing is iterative. Run regression tests after every change. Document everything. Good testing now saves hours of troubleshooting later.

---

**Document Version:** 1.0
**Last Updated:** October 24, 2025
**Next Review:** After Intake workflow completion
