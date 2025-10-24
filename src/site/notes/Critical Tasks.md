---
{"dg-publish":true,"permalink":"/critical-tasks/","dgShowToc":true}
---

# Critical Tasks

**Pre-Deployment Work**
**Last Updated:** October 24, 2025

These tasks MUST be completed before deploying to production.

**Estimated Total Effort:** 14-21 hours

---

## CRITICAL-001: Complete IntakeScreen Form

**Priority:** HIGH (Blocking deployment)
**Status:** 30% complete (displays requirements, form not built)
**Estimated Effort:** 6-8 hours

### Current State
- ✅ Screen loads correctly
- ✅ Variables set (varIntakeReferral, varIntakeUnit, varUnitType)
- ✅ Eligibility requirements display from unit type
- ❌ Form2 is placeholder, not configured
- ❌ No data entry fields
- ❌ No save functionality
- ❌ No workflow logic

### Tasks

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

---

## CRITICAL-002: Validate Convert to Resident Workflow

**Priority:** HIGH (Core workflow)
**Status:** Implementation complete, testing pending
**Estimated Effort:** 1-2 hours

### Current State
- ✅ scrConvert screen built
- ✅ FormViewer displays referral data
- ✅ Input fields for lease date and service providers
- ✅ Patch formula creates resident with all fields
- ✅ Unit update logic implemented
- ✅ Referral deletion implemented
- ✅ Navigation to new resident info screen
- ⚠️ Not yet tested end-to-end with real data

### Tasks

**Task 2.1: Prepare Test Data** (15 min)
- [ ] Create test referral with complete data
- [ ] Assign to specific unit
- [ ] Set household status to "Applicant, Approved"
- [ ] Fill in demographics, contact info, benefits

**Task 2.2: Execute Convert Workflow** (30 min)
- [ ] Navigate to test referral
- [ ] Click "Convert to Current Resident" button
- [ ] Review data in FormViewer (verify all fields showing)
- [ ] Select lease signing date
- [ ] Select service provider(s)
- [ ] Click "Convert to Current Resident" button
- [ ] Observe for errors

**Task 2.3: Verify Results** (30 min)
- [ ] New resident created in Current Residents list
- [ ] All fields transferred correctly:
  - [ ] Name, Preferred Name, Preferred Language
  - [ ] Contact info (phone, email)
  - [ ] Demographics (DOB, Veteran Status, Disability)
  - [ ] Benefits and income fields
  - [ ] HMIS and County Case Numbers
  - [ ] Household Size
  - [ ] Lease Signing Date
- [ ] Unit Number lookup populated correctly
- [ ] Associated Providers populated correctly
- [ ] Unit record updated:
  - [ ] Occupant Household points to new resident
  - [ ] Referral Household cleared
  - [ ] Has referral = false
  - [ ] Is occupied = true
- [ ] Original referral deleted from Referrals & Applicants
- [ ] Navigation to ResInfoScreen with new resident loaded
- [ ] Form in edit mode

**Task 2.4: Test Edge Cases** (15 min)
- [ ] Test with no service providers selected
- [ ] Test with multiple service providers
- [ ] Test with minimal referral data (only required fields)

**Acceptance Criteria:**
- [ ] Workflow completes without errors
- [ ] All data transfers accurately
- [ ] Lookup fields handled correctly
- [ ] Unit properly marked as occupied
- [ ] Referral properly removed
- [ ] User ends up on correct screen viewing new resident

---

## CRITICAL-003: Implement Household Members Section

**Priority:** HIGH (Compliance requirement)
**Status:** Not started
**Estimated Effort:** 4-6 hours

### Current State
- ✅ scrResidentAssociations has placeholder for household members
- ✅ Layout includes three-column design
- ❌ No Participants/Household Members list created or linked
- ❌ No gallery built
- ❌ No form built
- ❌ No add/edit/remove functionality

### Tasks

**Task 3.1: Create or Link Participants List** (1 hour)
- [ ] Check if Participants list already exists in SharePoint
- [ ] If not, create new list "Household Members"
- [ ] Define fields:
  - Title (Person name)
  - Relationship to Head of Household (Choice: Spouse, Child, Parent, Other)
  - Date of Birth (Date)
  - Associated Resident (Lookup to Current Residents)
- [ ] Add data connection to Power Apps
- [ ] Test connection

**Task 3.2: Build Household Members Gallery** (1 hour)
- [ ] Add gallery to scrResidentAssociations household members section
- [ ] Set Items to filter by varSelectedResident
- [ ] Display: Name, Relationship, Age (calculated from DOB)
- [ ] Add "+" button to add new member
- [ ] Add edit/delete icons on each gallery item

**Gallery Items Formula:**
```powerapps
Items: =Filter('Household Members', 'Associated Resident'.Id = varSelectedResident.ID)
```

**Task 3.3: Build Household Member Form** (1-2 hours)
- [ ] Create modal form for add/edit
- [ ] Add data cards: Name, Relationship, Date of Birth
- [ ] Set Associated Resident default to varSelectedResident
- [ ] Configure form for New/Edit modes
- [ ] Style consistently with other forms

**Task 3.4: Implement Add/Edit/Delete Logic** (1 hour)
- [ ] "+" button shows form in New mode
- [ ] Gallery edit icon shows form in Edit mode with selected item
- [ ] Delete icon removes member (with confirmation)
- [ ] OnSuccess refreshes gallery
- [ ] Add Notify() messages

**Task 3.5: Test Complete Functionality** (30 min)
- [ ] Add new household member
- [ ] Edit existing household member
- [ ] Delete household member
- [ ] Verify filtering works (only shows members for current resident)
- [ ] Verify age calculation accurate
- [ ] Test with multiple members

**Acceptance Criteria:**
- [ ] Can add new household members
- [ ] Can edit existing members
- [ ] Can delete members (with confirmation)
- [ ] Gallery shows only members for current resident
- [ ] Data saves correctly to SharePoint
- [ ] User receives appropriate notifications

---

## CRITICAL-004: Add OnSuccess Handlers

**Priority:** MEDIUM-HIGH (User feedback)
**Status:** Not started
**Estimated Effort:** 1-2 hours

### Current State
- ❌ RNAInfoScreen main form has no OnSuccess handler
- ❌ UnitInfoScreen Form1 has no OnSuccess handler
- Result: No user feedback after saves, no data refresh

### Tasks

**Task 4.1: Add OnSuccess to RNAInfoScreen Form** (30 min)
- [ ] Locate main form on RNAInfoScreen
- [ ] Add OnSuccess property
- [ ] Add Notify() success message
- [ ] Refresh varSelectedReferral
- [ ] Refresh varReferralUnit
- [ ] Refresh data source

**OnSuccess Formula:**
```powerapps
OnSuccess: |-
  =Notify("Referral updated successfully.", NotificationType.Success);
  Set(varSelectedReferral, ThisItem);
  Set(varReferralUnit, LookUp(Units, ID = varSelectedReferral.'Unit Number'.Id));
  Refresh('Referrals & Applicants')
```

**Task 4.2: Add OnSuccess to UnitInfoScreen Form1** (30 min)
- [ ] Locate Form1 on UnitInfoScreen
- [ ] Add OnSuccess property
- [ ] Add Notify() success message
- [ ] Refresh varActiveUnit
- [ ] Refresh data source

**OnSuccess Formula:**
```powerapps
OnSuccess: |-
  =Notify("Unit updated successfully.", NotificationType.Success);
  Set(varActiveUnit, ThisItem);
  Refresh(Units)
```

**Task 4.3: Test Both Forms** (30 min)
- [ ] Edit referral, save, verify notification appears
- [ ] Edit unit, save, verify notification appears
- [ ] Verify data refreshes after save
- [ ] Verify variables updated correctly

**Acceptance Criteria:**
- [ ] Users see success message after saving
- [ ] Data refreshes automatically
- [ ] Variables stay in sync with saved data

---

## CRITICAL-005: Password Management Solution

**Priority:** HIGH (Security)
**Status:** Not started
**Estimated Effort:** 1-2 hours

### Current State
- ✅ Account Information list tracks provider accounts
- ✅ scrResidentAccounts form fully functional
- ❌ Password field stores plain text in SharePoint (security risk)

### Tasks

**Task 5.1: Document Browser Password Manager Usage** (30 min)
- [ ] Create user guide for browser password managers
- [ ] Include instructions for Chrome, Edge, Firefox
- [ ] Add screenshots showing how to save/retrieve passwords
- [ ] Document best practices

**Task 5.2: Remove Password Field from SharePoint** (15 min)
- [ ] Navigate to Account Information list settings
- [ ] Delete Password column
- [ ] Verify deletion doesn't break existing records

**Task 5.3: Update Power Apps Form** (30 min)
- [ ] Remove Password data card from scrResidentAccounts form
- [ ] Add informational label: "Use your browser's password manager to store account passwords"
- [ ] Test form still saves correctly without password field

**Task 5.4: Communicate Change to Users** (15 min)
- [ ] Draft announcement about password management approach
- [ ] Explain browser password manager usage
- [ ] Set expectations for how to handle account passwords

**Acceptance Criteria:**
- [ ] Password field removed from SharePoint
- [ ] Password data card removed from form
- [ ] Form functions correctly without password field
- [ ] User documentation created
- [ ] Account tracking remains fully functional

---

## Summary

| Task | Priority | Effort | Status |
|------|----------|--------|--------|
| CRITICAL-001: IntakeScreen | HIGH | 6-8 hrs | Not started |
| CRITICAL-002: Test Convert | HIGH | 1-2 hrs | Code complete |
| CRITICAL-003: Household Members | HIGH | 4-6 hrs | Not started |
| CRITICAL-004: OnSuccess Handlers | MED-HIGH | 1-2 hrs | Not started |
| CRITICAL-005: Password Mgmt | HIGH | 1-2 hrs | Not started |

**Total Estimated Effort:** 14-21 hours

---

**After These Complete:** Ready for pilot rollout. See [[Deployment Strategy\|Deployment Strategy]] for rollout plan.

**Related Documentation:**
- [[Current Status\|Current Status]] - Overall project snapshot
- [[Screen Status\|Screen Status]] - Screen-by-screen details
- [[Workflow Status\|Workflow Status]] - Workflow implementation status
- [[Deployment Strategy\|Deployment Strategy]] - Pilot and rollout plan
