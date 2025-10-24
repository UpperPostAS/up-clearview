---
{"dg-publish":true,"permalink":"/up-clearview-development-roadmap/","dgShowToc":true}
---

**Complete Task List for Remaining Work**

---

## About This Roadmap

This document tracks outstanding development work—bug fixes, in-flight features, planned enhancements, and polish tasks—for the current UP Clearview build. Pair it with the [[UP Clearview - Project Overview & Current State\|project overview]] to understand context before diving into specific assignments.

**Current Status:** 80% Complete  
**Estimated Remaining Effort:** 21-31 hours

Need the technical underpinnings of an item? Follow the links into the [[UP Clearview – Technical Documentation\|technical manual]] or the [[UP Clearview – Data Sources\|data source reference]] as you work.

---

## Critical Path (Pre-Deployment)

These tasks MUST be completed before deploying to production.

---

### CRITICAL-001: Complete IntakeScreen Form

**Priority:** HIGH (Blocking deployment)
**Status:** 30% complete (displays requirements, form not built)
**Estimated Effort:** 6-8 hours
**Assigned To:** [Developer]

#### Current State
- ✅ Screen loads correctly
- ✅ Variables set (varIntakeReferral, varIntakeUnit, varUnitType)
- ✅ Eligibility requirements display from unit type
- ❌ Form2 is placeholder, not configured
- ❌ No data entry fields
- ❌ No save functionality
- ❌ No workflow logic

#### Tasks

**Task 1.1: Design Form Fields** (1 hour)
- [ ] Determine required vs optional fields
- [ ] Design field layout and grouping
- [ ] Create field list document

**Proposed Fields:**
```
- Intake Completion Date (Date, required, default: Today)
- Completed By (Person or Text, required, default: current user)
- Documentation Checklist (group of Yes/No):
  - [ ] State ID or Driver's License
  - [ ] Social Security Card
  - [ ] Birth Certificate
  - [ ] Income Verification Documents
  - [ ] Disability Documentation (conditional on unit type)
  - [ ] Homelessness Verification (conditional on unit type)
  - [ ] Professional Statement of Need (conditional on unit type)
  - [ ] Other Required Documents
- Verification Notes (Multi-line text, optional)
- Eligibility Confirmed (Yes/No, required)
- Ready for Move-In (Yes/No, required)
```

**Task 1.2: Build Form** (2-3 hours)
- [ ] Delete placeholder Form2
- [ ] Add new form control bound to 'Referrals & Applicants'
- [ ] Set form Item = varIntakeReferral
- [ ] Add data cards for each field
- [ ] Configure field types (date, text, checkboxes, toggle)
- [ ] Set required fields
- [ ] Test form layout

**Task 1.3: Add Conditional Field Visibility** (1 hour)
- [ ] Disability Documentation visible only if varUnitType.'Disability Requirement' is not "None"
- [ ] Homelessness Verification visible only if varUnitType.'Homelessness Requirement' = Yes
- [ ] Professional Statement visible only if varUnitType requires it

**Conditional Visibility Formula Example:**
```powerapps
DisabilityDocCard.Visible: =varUnitType.'Disability Requirement' <> "None"
```

**Task 1.4: Implement OnSuccess Handler** (1 hour)
- [ ] Add OnSuccess property to form
- [ ] Patch referral record to update status
- [ ] Set 'Intake Completed' date
- [ ] Update 'Household Status' to appropriate next stage
- [ ] Show success notification
- [ ] Refresh data
- [ ] Navigate back to RNAInfoScreen

**OnSuccess Formula:**
```powerapps
OnSuccess: |-
  =Patch(
      'Referrals & Applicants',
      varIntakeReferral,
      {
          'Intake Completed': IntakeForm.LastSubmit.'Intake Completion Date',
          'Household Status': {
              '@odata.type': "#Microsoft.Azure.Connectors.SharePoint.SPListExpandedReference",
              Value: "Applicant, Pending Approval"
          }
      }
  );
  Notify("Intake completed successfully.", NotificationType.Success);
  Refresh('Referrals & Applicants');
  Navigate(RNAInfoScreen, ScreenTransition.None)
```

**Task 1.5: Add Validation** (30 min)
- [ ] Verify required fields validated
- [ ] Add custom validation if needed
- [ ] Test validation errors display

**Task 1.6: Test Complete Workflow** (1 hour)
- [ ] Create test referral
- [ ] Navigate to IntakeScreen
- [ ] Complete all fields
- [ ] Save successfully
- [ ] Verify status updated
- [ ] Verify intake date set
- [ ] Verify navigation back to RNAInfoScreen
- [ ] Verify intake button disabled or hidden after completion

**Acceptance Criteria:**
- [ ] Form displays all required fields
- [ ] Conditional fields show/hide based on unit type
- [ ] Form saves successfully
- [ ] Referral status advances to "Applicant"
- [ ] Intake date recorded
- [ ] User receives success notification
- [ ] Navigation returns to referral detail screen

**Dependencies:** None

**Blockers:** None

---

### CRITICAL-002: Fix Exit Resident Workflow

**Priority:** HIGH (Data integrity issue)
**Status:** 70% complete (archives resident, doesn't update unit)
**Estimated Effort:** 2-3 hours
**Assigned To:** [Developer]

#### Current State
- ✅ Exit button exists on ResInfoScreen
- ✅ Archives resident to Archived Records
- ✅ Updates resident's Household Status to "Former Occupant"
- ✅ Navigates away from resident
- ❌ Does NOT clear unit's Occupant Household
- ❌ Does NOT set unit's Is occupied = false
- ❌ Does NOT set unit's Date Available

#### Tasks

**Task 2.1: Locate Exit Button Formula** (15 min)
- [ ] Open ResInfoScreen in Power Apps Studio
- [ ] Find "Exit Resident" or "Move Out" button
- [ ] Review current OnSelect formula

**Task 2.2: Add Unit Update Logic** (1 hour)
- [ ] Add Patch for Units after resident archive
- [ ] Clear Occupant Household
- [ ] Set Is occupied = false
- [ ] Set Date Available = Today()
- [ ] Add Refresh(Units) call

**Updated Formula:**
```powerapps
OnSelect: |-
  =// Archive resident
  Patch(
      'Archived Records',
      Defaults('Archived Records'),
      {
          'Linked Resident': {
              '@odata.type': "#Microsoft.Azure.Connectors.SharePoint.SPListExpandedReference",
              Id: varSelectedResident.ID,
              Value: varSelectedResident.Title
          },
          'Archive Date': Today(),
          'Archive Reason': "Moved Out",
          'Unit': varResidentUnit
      }
  );

  // Update resident status
  Patch(
      'Current Residents',
      varSelectedResident,
      {
          'Household Status': {
              '@odata.type': "#Microsoft.Azure.Connectors.SharePoint.SPListExpandedReference",
              Value: "Former Occupant"
          }
      }
  );

  // NEW: Update unit
  Patch(
      Units,
      varResidentUnit,
      {
          'Occupant Household': Blank(),
          'Is occupied': false,
          'Date Available': Today()
      }
  );

  Refresh('Current Residents');
  Refresh(Units);
  Notify("Resident exited successfully.", NotificationType.Success);
  Navigate(ResBrowseScreen, ScreenTransition.None)
```

**Task 2.3: Test Exit Workflow** (1 hour)
- [ ] Create test resident in test unit
- [ ] Perform exit workflow
- [ ] Verify resident archived
- [ ] Verify resident status = "Former Occupant"
- [ ] **NEW: Verify unit cleared**
- [ ] **NEW: Verify unit Is occupied = false**
- [ ] **NEW: Verify unit Date Available = today**
- [ ] Verify unit appears in vacancy toggle
- [ ] Verify "Receive Referral" button now appears on unit

**Task 2.4: Regression Test** (30 min)
- [ ] Verify archive still works
- [ ] Verify resident status still updates
- [ ] Verify navigation still works
- [ ] Test with multiple residents

**Acceptance Criteria:**
- [ ] Resident archived successfully
- [ ] Resident status updated to "Former Occupant"
- [ ] Unit's Occupant Household cleared
- [ ] Unit's Is occupied set to false
- [ ] Unit's Date Available set to today
- [ ] Unit appears as vacant in all views
- [ ] Unit's "Receive Referral" button functional
- [ ] Success notification displayed

**Dependencies:** None

**Blockers:** None

---

### CRITICAL-003: Implement Household Members Section

**Priority:** HIGH (Needed for compliance)
**Status:** 0% complete (placeholder exists)
**Estimated Effort:** 4-6 hours
**Assigned To:** [Developer]

#### Current State
- ✅ scrResidentAssociations screen exists
- ✅ Three-column layout in place
- ✅ Accounts section complete
- ✅ Providers section complete
- ❌ Household Members section is empty placeholder

#### Tasks

**Task 3.1: Create/Verify Household Members SharePoint List** (30 min)
- [ ] Check if "Household Members" or "Participants" list exists
- [ ] If not, create new SharePoint list
- [ ] Configure columns:
  - [ ] Title (Member Name) - Single line text
  - [ ] Household (Lookup to Current Residents) - Lookup field
  - [ ] Relationship to HOH - Choice (Spouse, Child, Sibling, Parent, Other)
  - [ ] Date of Birth - Date
  - [ ] Age - Calculated (from DOB) or Number
  - [ ] Notes - Multiple lines of text

**Task 3.2: Add List to Power Apps Data Sources** (15 min)
- [ ] In Power Apps Studio, add Household Members list as data source
- [ ] Verify connection successful
- [ ] Test reading records

**Task 3.3: Build Gallery in Right Column** (1 hour)
- [ ] Locate right column container on scrResidentAssociations
- [ ] Add gallery control
- [ ] Set Items property to filter by current resident
- [ ] Configure gallery template: Name, Relationship, Age
- [ ] Style to match other columns

**Gallery Items Formula:**
```powerapps
Items: =Filter('Household Members', Household.Id = varSelectedResident.ID)
```

**Task 3.4: Add View/Edit Form** (1.5 hours)
- [ ] Add form control for Household Members
- [ ] Set up New, Edit, View modes
- [ ] Add data cards for all fields
- [ ] Default Household field to varSelectedResident
- [ ] Add Save and Cancel buttons

**Task 3.5: Implement Add Member Functionality** (1 hour)
- [ ] Add "+" or "Add Member" button
- [ ] OnSelect: NewForm(HouseholdMemberForm)
- [ ] Show form
- [ ] Test adding new member

**Task 3.6: Implement Edit/Delete** (1 hour)
- [ ] Gallery item OnSelect: show form in edit mode
- [ ] Add Delete button with confirmation
- [ ] Test editing existing member
- [ ] Test deleting member

**Task 3.7: Test Complete Functionality** (30 min)
- [ ] Create test resident
- [ ] Add 2-3 household members
- [ ] Edit a member
- [ ] Delete a member
- [ ] Verify members display correctly
- [ ] Verify members tied to correct resident

**Acceptance Criteria:**
- [ ] Household Members SharePoint list exists and configured
- [ ] Gallery displays members for selected resident
- [ ] Can add new household member
- [ ] Can edit existing household member
- [ ] Can delete household member
- [ ] Household automatically links to current resident
- [ ] Form validates required fields
- [ ] Display matches styling of other columns

**Dependencies:**
- May need SharePoint admin permissions to create list

**Blockers:**
- If Household Members list doesn't exist and permissions insufficient to create

---

### CRITICAL-004: Add OnSuccess Handlers to Forms

**Priority:** MEDIUM-HIGH (User experience, data refresh)
**Status:** Missing from 3 screens
**Estimated Effort:** 1-2 hours
**Assigned To:** [Developer]

#### Current State
Three main info screens have forms that save silently with no user feedback or data refresh.

#### Tasks

**Task 4.1: Add OnSuccess to RNAInfoScreen Form** (20 min)
- [ ] Open RNAInfoScreen in Power Apps Studio
- [ ] Locate Form1 (main referral form)
- [ ] Add OnSuccess property

**Formula:**
```powerapps
OnSuccess: |-
  =Set(varSelectedReferral, Form1.LastSubmit);
  Refresh('Referrals & Applicants');
  Notify("Referral updated successfully.", NotificationType.Success)
```

- [ ] Test: Edit referral, save, verify notification appears
- [ ] Test: Verify data refreshes after save

**Task 4.2: Add OnSuccess to ResInfoScreen Form** (20 min)
- [ ] Open ResInfoScreen in Power Apps Studio
- [ ] Locate Form1 (main resident form)
- [ ] Add OnSuccess property

**Formula:**
```powerapps
OnSuccess: |-
  =Set(varSelectedResident, Form1.LastSubmit);
  Refresh('Current Residents');
  Notify("Resident updated successfully.", NotificationType.Success)
```

- [ ] Test: Edit resident, save, verify notification
- [ ] Test: Verify data refreshes

**Task 4.3: Add OnSuccess to UnitInfoScreen Form** (20 min)
- [ ] Open UnitInfoScreen in Power Apps Studio
- [ ] Locate Form1 (unit status form)
- [ ] Add OnSuccess property

**Formula:**
```powerapps
OnSuccess: |-
  =Set(varActiveUnit, Form1.LastSubmit);
  Refresh(Units);
  Notify("Unit updated successfully.", NotificationType.Success)
```

- [ ] Test: Edit unit status, save, verify notification
- [ ] Test: Verify data refreshes

**Acceptance Criteria:**
- [ ] All three forms show success notification after save
- [ ] Data refreshes automatically after save
- [ ] Variables (varSelectedReferral, varSelectedResident, varActiveUnit) update with latest data
- [ ] No errors introduced

**Dependencies:** None

**Blockers:** None

---

### CRITICAL-005: Resolve Password Security Issue

**Priority:** HIGH (Security concern)
**Status:** Policy decision pending
**Estimated Effort:** 1-2 hours (implementation after decision)
**Assigned To:** [Project Lead + Developer]

#### Current State
- Account Information list contains password field (plain text)
- scrResidentAccounts may display password field
- Security risk if deployed as-is

#### Tasks

**Task 5.1: Make Decision** (Project Lead)
- [ ] Decide on approach:
  - Option A: Remove password field, use browser password managers
  - Option B: Integrate secure credential storage (e.g., Azure Key Vault)
  - Option C: Keep passwords but restrict permissions dramatically

**Recommended: Option A**

**Task 5.2: If Option A - Remove Password Field** (1 hour)
- [ ] Open SharePoint Account Information list
- [ ] Delete "Account Password" column
- [ ] Open scrResidentAccounts in Power Apps Studio
- [ ] Remove password data card from form (if present)
- [ ] Add instruction label: "Use your browser's password manager to store credentials"
- [ ] Test account creation without password field
- [ ] Update documentation

**Task 5.3: If Option B - Integrate Secure Storage** (8-12 hours)
- [ ] Research Azure Key Vault integration with Power Apps
- [ ] Set up Key Vault instance
- [ ] Configure Power Automate flow to store/retrieve passwords
- [ ] Modify form to call flow instead of storing in SharePoint
- [ ] Add encryption/decryption logic
- [ ] Test thoroughly
- [ ] Document process

**Task 5.4: If Option C - Restrict Permissions** (2 hours)
- [ ] Configure SharePoint list permissions
- [ ] Limit password column visibility to specific users
- [ ] Add warning message about password security
- [ ] Encrypt password field (if possible in SharePoint)
- [ ] Test with different user roles

**Task 5.5: Create User Documentation** (30 min)
- [ ] Document chosen approach
- [ ] If browser password managers: Create guide for staff
- [ ] If secure storage: Document retrieval process
- [ ] Add to user training materials

**Acceptance Criteria:**
- [ ] Decision made and documented
- [ ] Implementation complete based on chosen option
- [ ] Security risk mitigated
- [ ] Staff trained on new approach
- [ ] Ready for deployment

**Dependencies:**
- Decision from project lead/security team
- If Option B: Access to Azure Key Vault or similar

**Blockers:**
- Waiting for policy decision

---

### CRITICAL-006: Develop Comprehensive Test Suite

**Priority:** HIGH (Cannot deploy without testing)
**Status:** Testing Guide created, execution needed
**Estimated Effort:** 4-8 hours (test execution)
**Assigned To:** [QA Tester or Developer]

#### Current State
- Testing Guide document created (comprehensive)
- No systematic testing has been executed
- No test results documented

#### Tasks

**Task 6.1: Set Up Test Environment** (1 hour)
- [ ] Create test SharePoint site (or use production with caution)
- [ ] Create test data (units, referrals, residents, providers)
- [ ] Document test account credentials
- [ ] Prepare bug tracking spreadsheet/system

**Task 6.2: Execute Unit Tests** (2-3 hours)
- [ ] Test all 13 screens (UnitBrowseScreen through srcOrganizations)
- [ ] Follow test cases in Testing Guide
- [ ] Document results (pass/fail)
- [ ] Screenshot failures
- [ ] Log bugs found

**Task 6.3: Execute Workflow Tests** (2-3 hours)
- [ ] Test Receive Referral workflow (WF-001)
- [ ] Test Close Referral workflow (WF-002)
- [ ] Test Convert to Resident workflow (WF-003)
- [ ] Test Exit Resident workflow (WF-004) - will fail until CRITICAL-002 fixed
- [ ] Test Intake workflow (WF-005) - will fail until CRITICAL-001 complete
- [ ] Document results

**Task 6.4: Execute Integration Tests** (1 hour)
- [ ] Test INT-001: Receive → Close → Receive
- [ ] Test INT-002: Complete lifecycle
- [ ] Test INT-003: Service provider continuity
- [ ] Test INT-004: Unit type eligibility display

**Task 6.5: Execute Edge Case Tests** (1 hour)
- [ ] Test null/blank fields
- [ ] Test deleted lookup references
- [ ] Test special characters in names
- [ ] Test date boundaries
- [ ] Document behaviors

**Task 6.6: Compile Test Report** (30 min)
- [ ] Summarize test results
- [ ] List all bugs found with severity
- [ ] Calculate pass/fail percentage
- [ ] Identify critical issues blocking deployment
- [ ] Create bug fix priority list

**Task 6.7: Retest After Bug Fixes** (1-2 hours)
- [ ] After critical bugs fixed, rerun failed tests
- [ ] Execute regression tests
- [ ] Verify fixes work
- [ ] Document final results

**Acceptance Criteria:**
- [ ] All unit tests executed
- [ ] All workflow tests executed
- [ ] All integration tests executed
- [ ] Edge cases tested
- [ ] Test report created with pass/fail stats
- [ ] All critical bugs logged
- [ ] Regression testing complete after fixes

**Dependencies:**
- CRITICAL-001 and CRITICAL-002 should be fixed before final test run

**Blockers:**
- Intake and Exit workflows will fail tests until fixed

---

## Bug Fixes for Implemented Features

These are known issues in features that are otherwise complete.

---

### BUG-001: Remove() Only Clears Title Field

**Priority:** MEDIUM (Workaround exists)
**Status:** RESOLVED (already fixed in Convert workflow)
**Affected Feature:** Convert to Resident workflow
**Resolution:** Changed from `Remove()` to `RemoveIf()`

**Original Issue:**
```powerapps
Remove('Referrals & Applicants', varSelectedReferral)
// Only cleared Title field instead of deleting record
```

**Fix Applied:**
```powerapps
RemoveIf('Referrals & Applicants', ID = varSelectedReferral.ID)
// Properly deletes entire record
```

**No Action Needed** - Already resolved

---

### BUG-002: No User Feedback After Form Saves

**Priority:** MEDIUM
**Status:** Open
**Affected Screens:** RNAInfoScreen, ResInfoScreen, UnitInfoScreen

**See CRITICAL-004 for fix**

---

### BUG-003: Service Provider Creation Fails

**Priority:** MEDIUM
**Status:** Open (permission issue)
**Estimated Effort:** 2-4 hours (troubleshooting)
**Assigned To:** [Developer + SharePoint Admin]

#### Problem
- Users cannot create new service providers through scrServiceProviders
- Users cannot create new organizations through srcOrganizations
- Must create directly in SharePoint

#### Tasks

**Task B3.1: Check SharePoint List Permissions** (30 min)
- [ ] Open Service Providers list settings
- [ ] Check permissions for app users
- [ ] Verify "Add items" permission granted
- [ ] Do same for Agencies list

**Task B3.2: Check Power Apps Connection Permissions** (30 min)
- [ ] In Power Apps Studio, check data source connection
- [ ] Verify connection type (SharePoint direct or via connector)
- [ ] Check delegated permissions
- [ ] Test with different user account

**Task B3.3: Check Azure AD App Registration** (1 hour)
- [ ] Locate Azure AD app registration for Power Apps
- [ ] Verify API permissions include SharePoint write
- [ ] Check consent status
- [ ] Grant admin consent if needed

**Task B3.4: Test Create Operations** (30 min)
- [ ] Test creating provider with different user roles
- [ ] Check for error messages in browser console (F12)
- [ ] Document specific error received

**Task B3.5: Implement Fix Based on Findings** (1-2 hours)
- [ ] Apply appropriate fix (permissions, connection, etc.)
- [ ] Retest create operations
- [ ] Verify existing read/edit operations still work

**Acceptance Criteria:**
- [ ] Users can create new service providers through app
- [ ] Users can create new organizations through app
- [ ] No errors when saving new records
- [ ] New records appear immediately in galleries

**Dependencies:**
- May require SharePoint admin access
- May require Azure AD admin access

**Blockers:**
- Insufficient permissions to diagnose/fix

---

### BUG-004: Variable Naming Inconsistency

**Priority:** LOW (Code quality, not functional issue)
**Status:** Open
**Estimated Effort:** 2-3 hours
**Assigned To:** [Developer]

#### Problem
- Most screens use `varSelected[Entity]` pattern
- UnitBrowseScreen uses `varActiveUnit` instead of `varSelectedUnit`
- Inconsistent naming reduces code readability

#### Tasks

**Task B4.1: Find All References to varActiveUnit** (30 min)
- [ ] Use Power Apps Studio search
- [ ] List all screens/controls using varActiveUnit
- [ ] Document what needs to change

**Task B4.2: Rename to varSelectedUnit** (1 hour)
- [ ] UnitBrowseScreen gallery OnSelect
- [ ] UnitInfoScreen references
- [ ] Any other screens referencing this variable
- [ ] Test after each change

**Task B4.3: Verify All Functionality Still Works** (30 min)
- [ ] Test unit selection from browse
- [ ] Test navigation to unit info
- [ ] Test unit info display
- [ ] Test Receive Referral workflow

**Task B4.4: Search for Other Inconsistencies** (30 min)
- [ ] Review all variable names
- [ ] Document any other inconsistent naming
- [ ] Create list for future cleanup

**Acceptance Criteria:**
- [ ] All unit-related variables use `varSelectedUnit`
- [ ] All functionality works correctly after rename
- [ ] Code is more maintainable

**Dependencies:** None

**Blockers:** None

---

### BUG-005: Control Naming Not Descriptive

**Priority:** LOW (Code quality)
**Status:** Open
**Estimated Effort:** 4-6 hours
**Assigned To:** [Developer]

#### Problem
- Many controls have default names: Button1, Form1, Gallery1
- Hard to identify controls in formulas
- Reduces maintainability

#### Tasks

**Task B5.1: Create Naming Convention Document** (30 min)
- [ ] Define prefixes (btn, frm, gal, lbl, etc.)
- [ ] Define naming pattern: [prefix][ScreenName][Purpose]
- [ ] Example: btnRNAInfoClose, frmResidentEdit, galUnitBrowse

**Task B5.2: Rename Forms** (1 hour)
- [ ] Form1 → frmReferralEdit (RNAInfoScreen)
- [ ] Form1 → frmResidentEdit (ResInfoScreen)
- [ ] Form1 → frmUnitStatus (UnitInfoScreen)
- [ ] EditForm1 → frmReferralClose (RNACloseScreen)
- [ ] Update all formula references

**Task B5.3: Rename Galleries** (1 hour)
- [ ] Gallery names to galUnits, galReferrals, galResidents, etc.
- [ ] Update formula references

**Task B5.4: Rename Buttons** (1-2 hours)
- [ ] Workflow buttons to descriptive names
- [ ] Navigation buttons to descriptive names
- [ ] Update references

**Task B5.5: Rename Other Controls** (1 hour)
- [ ] Labels, icons, containers
- [ ] Focus on controls referenced in formulas
- [ ] Less critical controls can remain default names

**Task B5.6: Test After Each Screen** (1 hour)
- [ ] Test all functionality after renaming
- [ ] Verify no broken references

**Acceptance Criteria:**
- [ ] All major controls have descriptive names
- [ ] Naming convention documented
- [ ] All functionality works after renaming

**Dependencies:** None

**Blockers:** None

**Note:** This is cosmetic/maintainability. Can be done gradually or postponed post-deployment.

---

## In-Progress Features to Complete

Features that are started but not fully functional.

---

### INPROG-001: Complete IntakeScreen Form

**See CRITICAL-001** - Already covered above

---

### INPROG-002: Complete Household Members Section

**See CRITICAL-003** - Already covered above

---

### INPROG-003: Complete Exit Resident Unit Updates

**See CRITICAL-002** - Already covered above

---

## Planned Features to Build

Features that are mentioned in documentation but not yet started.

---

### PLANNED-001: Phone Screening Workflow Screen

**Priority:** MEDIUM (Listed as complete in some docs, but missing from export)
**Status:** Not in current export
**Estimated Effort:** Unknown (may exist in different version)
**Assigned To:** [Developer]

#### Investigation Needed

**Task P1.1: Determine Status** (30 min)
- [ ] Check if phone screening screen exists in production app
- [ ] Check if it's in different export/version
- [ ] Contact original developer about status
- [ ] Decide if needs to be built or just imported

**If Needs to Be Built:**

**Task P1.2: Design Phone Screening Form** (1 hour)
- [ ] List all phone screening questions
- [ ] Determine field types (text, yes/no, choice)
- [ ] Design layout

**Task P1.3: Build Phone Screening Screen** (2-3 hours)
- [ ] Create new screen
- [ ] Add form for Referrals & Applicants
- [ ] Add all phone screening fields
- [ ] Set Item = varSelectedReferral
- [ ] Style to match other screens

**Task P1.4: Implement Save Logic** (1 hour)
- [ ] OnSuccess handler
- [ ] Update referral record
- [ ] Set "Phone Screening Completed" flag
- [ ] Navigate back to RNAInfoScreen

**Task P1.5: Test Phone Screening Workflow** (1 hour)
- [ ] Navigate from RNAInfoScreen
- [ ] Complete phone screening
- [ ] Save successfully
- [ ] Verify data saved
- [ ] Verify flag set

**Acceptance Criteria:**
- [ ] Screen exists and is accessible
- [ ] All phone screening questions present
- [ ] Saves to referral record
- [ ] Marks screening complete
- [ ] Navigation works correctly

**Dependencies:**
- Determine if screen exists elsewhere first

**Blockers:**
- Unknown status

---

### PLANNED-002: Hide Empty Fields in View Mode

**Priority:** MEDIUM (User experience)
**Status:** Not started
**Estimated Effort:** 1-2 hours
**Assigned To:** [Developer]

#### Problem
- All form fields show on info screens even when blank
- Creates visual clutter
- Gives impression all fields required
- Designed as optional tool for staff

#### Tasks

**Task P2.1: Update RNAInfoScreen Form** (30 min)
- [ ] Open RNAInfoScreen Form1 in Power Apps Studio
- [ ] For each data card, add Visible property

**Formula:**
```powerapps
Visible: =!IsBlank(Parent.Default) || Parent.DisplayMode = DisplayMode.Edit
```

- [ ] Test: View mode shows only filled fields
- [ ] Test: Edit mode shows all fields

**Task P2.2: Update ResInfoScreen Form** (30 min)
- [ ] Same process for ResInfoScreen Form1
- [ ] Add Visible property to all data cards
- [ ] Test view/edit modes

**Task P2.3: Update UnitInfoScreen Form** (30 min)
- [ ] Same process for UnitInfoScreen Form1
- [ ] Add Visible property
- [ ] Test view/edit modes

**Acceptance Criteria:**
- [ ] In view mode, only populated fields display
- [ ] In edit mode, all fields display (even if blank)
- [ ] Toggle between modes works smoothly
- [ ] No fields permanently hidden that should be visible

**Dependencies:** None

**Blockers:** None

---

### PLANNED-003: Archiving Workflow Refinement

**Priority:** MEDIUM
**Status:** Basic archiving works, needs review
**Estimated Effort:** 2-3 hours
**Assigned To:** [Developer + Project Lead]

#### Current State
- Archived Records list exists
- Exit Resident creates archive record
- Close Referral does not create archive (just changes status)
- Format and completeness need review

#### Tasks

**Task P3.1: Review Current Archiving** (30 min)
- [ ] Check what data gets archived for residents
- [ ] Check what data is available in Archived Records list
- [ ] Determine if sufficient for reporting needs
- [ ] Decide if referral close-outs should also be archived

**Task P3.2: Decide on Archiving Strategy** (30 min)
- [ ] Hard delete vs. soft delete (status change)
- [ ] Should "former" records stay in main lists or move to archive?
- [ ] What reports/analytics need archived data?
- [ ] Document decision

**Task P3.3: Implement Referral Archiving (if decided)** (1 hour)
- [ ] Modify RNACloseScreen workflow
- [ ] Add Patch to create Archived Records entry
- [ ] Decide if referral deleted or just status changed
- [ ] Test close workflow with archiving

**Task P3.4: Review Archive Record Format** (30 min)
- [ ] Determine what fields should be in archive
- [ ] Currently: Record Name, Type, Linked records, Unit, Date, Reason, Notes
- [ ] Are there additional fields needed?
- [ ] Modify Archived Records list if needed

**Task P3.5: Test Archive Retrieval** (30 min)
- [ ] Practice retrieving archived records
- [ ] Verify useful for reporting
- [ ] Ensure no data loss

**Acceptance Criteria:**
- [ ] Archiving strategy documented and decided
- [ ] Implementation matches strategy
- [ ] Both resident and referral archives work (if decided)
- [ ] Archived data is retrievable and useful
- [ ] No data loss during archiving

**Dependencies:**
- Policy decision on archiving approach

**Blockers:**
- Awaiting decision

---

## Polish and User Experience

Improvements to make the app more polished and user-friendly.

---

### POLISH-001: Add User Notification Messages

**Priority:** MEDIUM
**Status:** Partially implemented (Convert workflow has notification)
**Estimated Effort:** 1-2 hours
**Assigned To:** [Developer]

#### Current State
- Convert to Resident shows success notification
- Most other operations are silent
- Users don't know if actions succeeded

#### Tasks

**Task POL1.1: Add Notifications to Workflows** (30 min)
- [ ] Receive Referral: "Referral created successfully."
- [ ] Close Referral: "Referral closed successfully."
- [ ] Exit Resident: "Resident exited successfully." (add to CRITICAL-002 fix)
- [ ] Intake: "Intake completed successfully." (add to CRITICAL-001)

**Task POL1.2: Add Error Notifications** (30 min)
- [ ] Add OnFailure handlers to critical forms
- [ ] Display First(Errors([List])).Message
- [ ] Test by forcing validation errors

**Example:**
```powerapps
OnFailure: |-
  =Notify(
      "Error: " & First(Errors('Current Residents')).Message,
      NotificationType.Error
  )
```

**Task POL1.3: Add Confirmation Dialogs** (30 min)
- [ ] Add confirmation before closing referral
- [ ] Add confirmation before exiting resident
- [ ] Add confirmation before deleting records

**Acceptance Criteria:**
- [ ] All major actions show success notifications
- [ ] Errors display user-friendly messages
- [ ] Destructive actions require confirmation
- [ ] Notifications display for 3-5 seconds and dismiss

**Dependencies:**
- CRITICAL-004 (OnSuccess handlers)

**Blockers:** None

---

### POLISH-002: Improve Form Layout and Styling

**Priority:** LOW
**Status:** Forms functional, could be prettier
**Estimated Effort:** 2-4 hours
**Assigned To:** [Developer]

#### Tasks

**Task POL2.1: Standardize Form Field Widths** (1 hour)
- [ ] Review all forms
- [ ] Set consistent widths for similar fields
- [ ] Align labels consistently
- [ ] Group related fields

**Task POL2.2: Add Section Headings** (1 hour)
- [ ] Add label headings to group fields
- [ ] Example: "Contact Information", "Demographics", "Program Details"
- [ ] Style headings consistently

**Task POL2.3: Improve Data Card Styling** (1 hour)
- [ ] Consistent font sizes
- [ ] Consistent colors
- [ ] Consistent spacing/padding
- [ ] Make required fields visually distinct

**Task POL2.4: Add Helpful Hints/Tooltips** (1 hour)
- [ ] Add hint text to fields where helpful
- [ ] Add tooltips explaining fields
- [ ] Add format examples (phone: 555-0100)

**Acceptance Criteria:**
- [ ] Forms look polished and professional
- [ ] Easy to scan and understand
- [ ] Consistent across all screens
- [ ] User-friendly hints provided

**Dependencies:** None

**Blockers:** None

---

### POLISH-003: Add Loading Indicators

**Priority:** LOW
**Status:** Not implemented
**Estimated Effort:** 1-2 hours
**Assigned To:** [Developer]

#### Tasks

**Task POL3.1: Add Loading Spinners to Galleries** (30 min)
- [ ] Show spinner while gallery loads
- [ ] Use LoadingSpinner property or custom control
- [ ] Test with slow connection

**Task POL3.2: Add Loading State for Workflows** (1 hour)
- [ ] Show spinner during Convert to Resident
- [ ] Show spinner during other long operations
- [ ] Disable buttons during processing (prevent double-click)

**Acceptance Criteria:**
- [ ] Users see loading indication during operations
- [ ] Buttons disabled during processing
- [ ] Loading indicators dismiss when complete

**Dependencies:** None

**Blockers:** None

---

## Testing and Quality Assurance

Testing-related tasks beyond the critical test suite.

---

### QA-001: Execute Comprehensive Test Suite

**See CRITICAL-006** - Already covered above

---

### QA-002: User Acceptance Testing with Staff

**Priority:** HIGH (Before deployment)
**Status:** Not started
**Estimated Effort:** 4-8 hours (staff time)
**Assigned To:** [Project Manager + Staff]

#### Tasks

**Task QA2.1: Recruit UAT Participants** (Project Manager)
- [ ] Identify 3-5 staff members from different roles
- [ ] Schedule UAT sessions (1 hour each)
- [ ] Prepare UAT scenarios

**Task QA2.2: Prepare UAT Environment** (Developer)
- [ ] Set up test environment
- [ ] Create sample data for realistic scenarios
- [ ] Prepare data sheets with test scenarios

**Task QA2.3: Conduct UAT Sessions** (Project Manager)
- [ ] Run 5 UAT tests from Testing Guide (UAT-001 through UAT-005)
- [ ] Observe staff using app
- [ ] Take notes on difficulties
- [ ] Collect feedback

**Task QA2.4: Compile UAT Feedback** (Project Manager)
- [ ] Summarize findings
- [ ] Categorize issues (critical, important, nice-to-have)
- [ ] Create task list for UAT findings
- [ ] Prioritize fixes

**Task QA2.5: Implement UAT Fixes** (Developer)
- [ ] Address critical UAT findings
- [ ] Implement important usability improvements
- [ ] Document nice-to-have items for future

**Task QA2.6: Conduct Follow-Up UAT** (Project Manager)
- [ ] Retest with staff after fixes
- [ ] Verify issues resolved
- [ ] Get final approval for deployment

**Acceptance Criteria:**
- [ ] At least 3 staff members complete UAT
- [ ] All critical usability issues resolved
- [ ] Staff feel comfortable using app
- [ ] Staff provide sign-off for deployment

**Dependencies:**
- CRITICAL items should be complete before UAT

**Blockers:**
- Staff availability

---

### QA-003: Create Automated Test Suite (Future)

**Priority:** LOW (Post-deployment enhancement)
**Status:** Not started
**Estimated Effort:** 8-16 hours
**Assigned To:** [Developer]

#### Tasks (Future)

- [ ] Research Power Apps automated testing tools
- [ ] Set up Test Studio or Power Apps Test Engine
- [ ] Create automated tests for critical workflows
- [ ] Schedule automated test runs
- [ ] Set up alerts for test failures

**Note:** This is a post-deployment enhancement, not required for initial launch.

---

## Performance and Optimization

Tasks to improve app performance and scalability.

---

### PERF-001: Optimize Gallery Performance

**Priority:** LOW (Only needed if performance issues arise)
**Status:** Current performance acceptable
**Estimated Effort:** 3-4 hours
**Assigned To:** [Developer]

#### Current State
- Galleries use inline LookUp() functions
- Works fine for current dataset size (<100 records)
- May slow down with larger datasets

#### Tasks (If Needed)

**Task PERF1.1: Identify Performance Bottlenecks** (1 hour)
- [ ] Time gallery load with 100+ records
- [ ] Identify which galleries are slowest
- [ ] Check for delegation warnings

**Task PERF1.2: Pre-Load Joined Data** (2 hours)
- [ ] In OnVisible, ClearCollect joined data
- [ ] Example: ResidentUnit collection with resident + unit data
- [ ] Update gallery to use collection instead of lookups
- [ ] Test performance improvement

**Example:**
```powerapps
OnVisible: |-
  =ClearCollect(
      colResidentsWithUnits,
      AddColumns(
          Filter('Current Residents', !StartsWith(Lower('Household Status'.Value), "former")),
          "UnitInfo",
          LookUp(Units, ID = 'Unit Number'.Id)
      )
  )
```

**Task PERF1.3: Test with Large Dataset** (1 hour)
- [ ] Create 500+ test records
- [ ] Test gallery performance
- [ ] Verify delegation still works
- [ ] Document results

**Acceptance Criteria:**
- [ ] Galleries load in < 3 seconds even with 500+ records
- [ ] No delegation warnings
- [ ] User experience smooth

**Dependencies:** None

**Blockers:** None

**Trigger:** Only implement if users report slowness

---

### PERF-002: Implement Lazy Loading (Future)

**Priority:** LOW
**Status:** Not needed currently
**Estimated Effort:** 4-6 hours
**Assigned To:** [Developer]

#### Tasks (Future)

- [ ] Research lazy loading in Power Apps galleries
- [ ] Implement "Load More" button for large datasets
- [ ] Test with 1000+ records
- [ ] Document implementation

**Note:** Only needed if dataset grows very large

---

## Post-Deployment Enhancements

Features to add after initial deployment, based on user feedback and needs.

---

### FUTURE-001: Side Navigation Menu

**Priority:** MEDIUM (User experience improvement)
**Status:** Not started
**Estimated Effort:** 4-6 hours
**Assigned To:** [Developer]

#### Description
Add persistent side navigation menu for quick access to key sections without going through browse screens.

#### Tasks

**Task F1.1: Design Navigation Menu** (1 hour)
- [ ] Decide on menu structure
- [ ] Choose icons for each section
- [ ] Design collapsed/expanded states

**Proposed Menu:**
- Home (Dashboard - future)
- Units
- Referrals & Applicants
- Residents
- Service Providers
- Organizations
- Reports (future)

**Task F1.2: Create Navigation Component** (2 hours)
- [ ] Build GroupContainer with navigation buttons
- [ ] Add icons and labels
- [ ] Style to match app theme
- [ ] Make collapsible

**Task F1.3: Add to All Screens** (2 hours)
- [ ] Copy navigation component to each screen
- [ ] Position consistently (left side)
- [ ] Adjust screen layouts to accommodate menu
- [ ] Wire up navigation (Navigate functions)

**Task F1.4: Test Navigation** (1 hour)
- [ ] Test from every screen
- [ ] Verify current screen highlighted
- [ ] Test collapsed/expanded states

**Acceptance Criteria:**
- [ ] Navigation menu appears on all screens
- [ ] One-click access to main sections
- [ ] Current location indicated
- [ ] Responsive and user-friendly

**Dependencies:** None

**Blockers:** None

---

### FUTURE-002: Dashboard View

**Priority:** MEDIUM
**Status:** Not started
**Estimated Effort:** 8-12 hours
**Assigned To:** [Developer]

#### Description
Create dashboard with key metrics and quick insights.

#### Proposed Dashboard Elements

**Task F2.1: Design Dashboard Layout** (1 hour)
- [ ] Sketch dashboard design
- [ ] Decide on key metrics to display

**Suggested Metrics:**
- Total units: Occupied / Vacant
- Referrals: Active / Pending Intake / Approved
- Residents: Current count
- Recent activity: Last 5 conversions, exits, referrals

**Task F2.2: Build Dashboard Screen** (4-6 hours)
- [ ] Create new screen
- [ ] Add cards/containers for each metric
- [ ] Use CountRows, Filter, etc. to calculate metrics
- [ ] Add charts (if Power Apps supports)
- [ ] Style professionally

**Task F2.3: Add Navigation to Dashboard** (1 hour)
- [ ] Make dashboard home screen
- [ ] Add to side navigation menu
- [ ] Test navigation

**Task F2.4: Add Drill-Down Links** (2-3 hours)
- [ ] Click on metric → go to related browse screen
- [ ] Example: Click "5 Pending Intake" → RNABrowseScreen filtered to pending intake
- [ ] Test all drill-downs

**Acceptance Criteria:**
- [ ] Dashboard displays key metrics
- [ ] Metrics accurate and update real-time
- [ ] Drill-down navigation works
- [ ] Dashboard is useful and informative

**Dependencies:** None

**Blockers:** None

---

### FUTURE-003: Add Provider from Association Screen

**Priority:** MEDIUM
**Status:** Not started
**Estimated Effort:** 2-3 hours
**Assigned To:** [Developer]

#### Description
Allow adding new service provider directly from scrResidentAssociations without navigating away.

#### Tasks

**Task F3.1: Add "New Provider" Button** (30 min)
- [ ] Add "+" button to Service Providers section
- [ ] Style to match design

**Task F3.2: Create Modal Form** (1 hour)
- [ ] Create GroupContainer with provider form
- [ ] Add fields: Name, Organization, Role, Contact
- [ ] Add Save/Cancel buttons
- [ ] Initially hidden (Visible = false)

**Task F3.3: Wire Up Functionality** (1 hour)
- [ ] Button click → show modal
- [ ] Save → create provider, auto-associate with resident
- [ ] Cancel → hide modal
- [ ] Refresh provider gallery after save

**Task F3.4: Test Workflow** (30 min)
- [ ] Open resident associations
- [ ] Click "+"
- [ ] Enter new provider
- [ ] Save
- [ ] Verify provider created
- [ ] Verify auto-associated with resident

**Acceptance Criteria:**
- [ ] Can add provider without leaving associations screen
- [ ] New provider automatically associated with current resident
- [ ] Smooth user experience
- [ ] No navigation required

**Dependencies:**
- BUG-003 must be fixed (provider creation permissions)

**Blockers:**
- Cannot implement until provider creation works

---

### FUTURE-004: Account Provider Batch View

**Priority:** MEDIUM
**Status:** Not started
**Estimated Effort:** 2-3 hours
**Assigned To:** [Developer]

#### Description
View all accounts for a specific provider (e.g., "Xcel Energy") for batch payment processing.

#### Tasks

**Task F4.1: Design Two-Pane Layout** (30 min)
- [ ] Sketch layout
- [ ] Left pane: Account Providers list
- [ ] Right pane: Accounts for selected provider

**Task F4.2: Build Provider Selection Gallery** (1 hour)
- [ ] Create gallery of Account Providers
- [ ] OnSelect: Set varSelectedAccountProvider
- [ ] Highlight selected provider

**Task F4.3: Build Accounts Gallery** (1 hour)
- [ ] Create gallery filtered by selected provider
- [ ] Display: Resident, Account Number, Amount Due, Next Due Date
- [ ] Add checkboxes for "paid" marking
- [ ] Add "Mark All Paid" button

**Task F4.4: Test Batch Processing** (30 min)
- [ ] Select "Xcel Energy"
- [ ] View all residents' Xcel accounts
- [ ] Mark as paid
- [ ] Verify updates

**Acceptance Criteria:**
- [ ] Can view all accounts by provider
- [ ] Easy to see which accounts need payment
- [ ] Can mark multiple accounts as paid
- [ ] Useful for monthly bill pay routine

**Dependencies:** None

**Blockers:** None

---

### FUTURE-005: ETO Integration

**Priority:** LOW (Future phase)
**Status:** Research needed
**Estimated Effort:** Unknown (likely 20-40 hours)
**Assigned To:** [Developer + Integration Specialist]

#### Description
Integrate with ETO (Efforts to Outcomes) system to reduce duplicate data entry.

#### Tasks (High-Level)

**Task F5.1: Research ETO API** (4-8 hours)
- [ ] Determine if ETO has API
- [ ] Review API documentation
- [ ] Assess feasibility
- [ ] Determine licensing/permission requirements

**Task F5.2: Design Integration Points** (2-4 hours)
- [ ] What data should sync?
- [ ] When should sync occur?
- [ ] One-way or two-way sync?

**Task F5.3: Build Integration** (12-20 hours)
- [ ] Create Power Automate flows
- [ ] Handle authentication
- [ ] Map fields between systems
- [ ] Error handling

**Task F5.4: Test Thoroughly** (4-8 hours)
- [ ] Test data flow
- [ ] Test error scenarios
- [ ] Verify data integrity

**Note:** This is a major undertaking. Requires stakeholder buy-in, budget, and extended timeline.

---

### FUTURE-006: Mobile Optimization

**Priority:** LOW
**Status:** Not started
**Estimated Effort:** 8-16 hours
**Assigned To:** [Developer]

#### Description
Optimize app for mobile/tablet use in the field.

#### Tasks

**Task F6.1: Test on Mobile Devices** (2 hours)
- [ ] Test on phone (iOS and Android)
- [ ] Test on tablet
- [ ] Document usability issues

**Task F6.2: Adjust Layouts for Mobile** (4-8 hours)
- [ ] Make galleries scrollable
- [ ] Adjust font sizes for touch
- [ ] Make buttons larger for touch
- [ ] Test responsive behavior

**Task F6.3: Optimize for Touch** (2-4 hours)
- [ ] Increase tap targets
- [ ] Remove hover-only interactions
- [ ] Test gestures (swipe, pinch)

**Task F6.4: Test Mobile Performance** (2 hours)
- [ ] Test on slow connections
- [ ] Optimize image sizes
- [ ] Reduce unnecessary data loading

**Note:** Power Apps has built-in responsiveness, but specific optimizations may be needed.

---

## Summary

### Critical Path (Must Do Before Deployment)

1. **CRITICAL-001:** Complete IntakeScreen Form (6-8 hours)
2. **CRITICAL-002:** Fix Exit Resident Workflow (2-3 hours)
3. **CRITICAL-003:** Implement Household Members Section (4-6 hours)
4. **CRITICAL-004:** Add OnSuccess Handlers to Forms (1-2 hours)
5. **CRITICAL-005:** Resolve Password Security Issue (1-2 hours)
6. **CRITICAL-006:** Develop Comprehensive Test Suite (4-8 hours)

**Total Critical Path:** 18-29 hours

### Bug Fixes (Should Do Before Deployment)

- **BUG-003:** Service Provider Creation Permissions (2-4 hours)
- **BUG-004:** Variable Naming Inconsistency (2-3 hours) - Optional
- **BUG-005:** Control Naming Not Descriptive (4-6 hours) - Optional

### Planned Features (Should Complete Soon)

- **PLANNED-001:** Phone Screening Screen (if missing) (3-5 hours)
- **PLANNED-002:** Hide Empty Fields in View Mode (1-2 hours)
- **PLANNED-003:** Archiving Refinement (2-3 hours)

### Polish (Nice to Have)

- **POLISH-001:** Add User Notification Messages (1-2 hours)
- **POLISH-002:** Improve Form Layout (2-4 hours)
- **POLISH-003:** Add Loading Indicators (1-2 hours)

### Post-Deployment Enhancements

- **FUTURE-001:** Side Navigation Menu (4-6 hours)
- **FUTURE-002:** Dashboard View (8-12 hours)
- **FUTURE-003:** Add Provider from Association Screen (2-3 hours)
- **FUTURE-004:** Account Provider Batch View (2-3 hours)
- **FUTURE-005:** ETO Integration (20-40 hours)
- **FUTURE-006:** Mobile Optimization (8-16 hours)

---

**Total Estimated Effort to Deployment:**
- **Critical:** 18-29 hours
- **Important bugs:** 2-4 hours
- **Planned features:** 4-8 hours
- **Polish:** 4-8 hours

**Grand Total:** 28-49 hours (depending on scope decisions)

**Recommended Minimum for Deployment:** Complete all CRITICAL items = 18-29 hours

---

**Document Version:** 1.0
**Date:** October 24, 2025
**Last Updated:** After Convert to Resident workflow completion
**Next Review:** After critical items completed
