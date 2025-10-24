---
{"dg-publish":true,"permalink":"/screen-status/","dgShowToc":true}
---

# Screen Status

**Last Updated:** October 24, 2025

Detailed status of all 13 application screens with testing results and known issues.

---

## Main Browse Screens

### UnitBrowseScreen ✅ Working Great
**Status:** Fully functional, extensively tested

**Features:**
- Gallery of all units with occupancy status
- Toggle to filter current and upcoming vacancies (shows vacant OR has given notice)
- Color-coded by unit type (LTH Housing Support, CoC, Section 8, HUD-VASH)
- Displays occupancy status with detailed subtitles
- Search and sort functionality

**Variable Used:** `varActiveUnit`

**Testing Notes:** Filter logic updated to include units where resident has given notice. Working as expected.

---

### RNABrowseScreen ✅ Working Great
**Status:** Fully functional, extensively tested

**Features:**
- List of all referrals and applicants
- Search and sort functionality
- Status-based filtering (excludes "former" referrals)
- Color-coded by unit type
- Navigation to RNAInfoScreen

**Variable Used:** `varSelectedReferral`

**Testing Notes:** No issues identified. Clean browse experience.

---

### ResBrowseScreen ✅ Working Great
**Status:** Fully functional, extensively tested

**Features:**
- Active residents gallery
- Sort by unit number
- Unit type color coding
- Filters out former residents
- Navigation to ResInfoScreen

**Variable Used:** `varSelectedResident`

**Testing Notes:** No issues identified. Performs well.

---

## Detail/Info Screens

### UnitInfoScreen ✅ Functional
**Status:** Working, minor enhancement needed

**Features:**
- Three-section viewer: Unit Info, Administrative Info, Eligibility Info
- Editable form for unit status tracking
- "Receive Referral" modal form
- Dynamic display based on occupancy status

**Known Issue:**
- Missing OnSuccess handler on Form1 (no user feedback after save)

**Enhancement Needed:** Add OnSuccess handler with Notify() message

---

### RNAInfoScreen ⚠️ Mostly Functional
**Status:** Working with known issue in phone screening section

**Features:**
- Comprehensive data display
- Workflow buttons (phone screening, intake, convert, close-out)
- Edit mode toggle
- Pre-loads unit data (varReferralUnit)

**Known Issues:**
- Phone screening section has some issues (main functionality works)
- Missing OnSuccess handler on main form

**Testing Status:** Main functionality validated, phone screening needs review

---

### ResInfoScreen ✅ Working Great
**Status:** Fully functional, extensively tested

**Features:**
- Household information display
- Service provider connections
- Exit resident functionality with confirmation modal
- Edit/view mode toggle
- Pre-loads unit data (varResidentUnit)

**Exit Workflow:**
- Creates Exit record before removing resident ✅
- Updates unit (clears occupant, sets vacant) ✅
- Success notification and navigation ✅

**Testing Notes:** Exit workflow extensively tested. All functionality working correctly.

**Variable Used:** `varSelectedResident`

---

## Workflow Screens

### IntakeScreen ❌ Critical Blocker
**Status:** 30% complete - displays requirements but lacks functionality

**Current State:**
- ✅ Screen loads, variables set correctly
- ✅ Eligibility requirements display from linked unit
- ❌ Form2 is placeholder, not configured
- ❌ No data entry fields
- ❌ No save functionality

**Blocker:** Cannot deploy without this workflow functional

**Estimated Fix:** 6-8 hours to build form, add fields, implement OnSuccess handler

See [[Critical Tasks#CRITICAL-001\|Critical Tasks#CRITICAL-001]] for detailed implementation plan.

---

### RNACloseScreen ✅ Fully Functional
**Status:** Extensively tested, production-ready

**Features:**
- Form with close-out date, reason, notes fields
- Creates Exit record with correct stage parsing (Applicant vs Referral)
- Updates unit: clears Referral Household, sets Has referral = false, clears Referral Requested
- Removes referral record after archiving
- Success notification and navigation

**Testing Notes:**
- Tested with multiple scenarios
- Stage parsing working correctly (StartsWith logic)
- Unit cleanup validated
- Exit record creation confirmed

**Variable Used:** `varSelectedReferral`

---

### scrConvert (Convert to Resident) ⚠️ Needs Testing
**Status:** Implementation complete, validation testing pending

**Features:**
- Referral data review (FormViewer)
- Input fields for lease date, coordinator, service providers
- Creates Current Resident record with all referral data
- Handles lookup fields correctly (Unit Number, Associated Providers)
- Updates Unit record (sets Occupant Household, clears Referral, marks occupied)
- Removes referral record
- Navigates to new resident's info page in edit mode

**Implementation Notes:**
- Built using progressive debugging approach
- Started with minimal fields, added incrementally
- Fixed field type mismatches (Choice vs Text)

**Testing Status:** Code complete but not yet validated with real data. Should test:
1. Field transfer accuracy
2. Lookup field handling
3. Unit update correctness
4. Navigation to correct screen

**Variable Used:** `varSelectedReferral`, creates `varNewResident`

---

## Association/Management Screens

### scrResidentAssociations ⚠️ Integration Pending
**Status:** Functional with workaround needed

**Features:**
- Three-column associations view
- Bill pay accounts section (complete)
- Service providers section (complete - can select existing)
- Household members section (not yet started)

**Known Issue:**
- Cannot add new service provider from this screen
- Must use SharePoint directly to create providers
- Then can select them in the app

**Workaround:** Functional but not ideal UX. Users must switch to SharePoint to add providers.

**Enhancement Needed:** Add modal form or inline creation for new service providers (2-3 hours)

See [[Critical Tasks#CRITICAL-003\|Critical Tasks#CRITICAL-003]] for household members implementation plan.

---

### scrResidentAccounts ✅ Ready for Deployment
**Status:** Fully functional, pending password management solution

**Features:**
- Form for Account Information list
- Tracks provider accounts with usernames and billing
- Conditional visibility based on "Is Active" and "Is Payable Account" toggles
- New/Edit/View modes
- Defaults to currently selected resident

**Blocker:** Password field needs to be removed from SharePoint list before deployment (security concern)

**Solution:** Staff use browser-built password managers (Chrome, Edge, Firefox)

**Testing Notes:** All functionality works correctly. Ready to deploy once password field removed.

---

### OrgScreen ✅ Functional
**Status:** Working well

**Features:**
- Browse and edit Agencies list
- Organization details, contact info, services provided
- Multi-select service providers field

**Testing Notes:** No issues identified.

---

### scrServiceProviders ✅ Functional
**Status:** Working well

**Features:**
- Browse and edit Service Providers list
- Provider details (name, role, program, organization)
- Links to organization via lookup field

**Testing Notes:** No issues identified.

---

## Summary Table

| Screen | Status | Testing | Blocker | Priority |
|--------|--------|---------|---------|----------|
| UnitBrowseScreen | ✅ Working Great | Extensive | No | - |
| RNABrowseScreen | ✅ Working Great | Extensive | No | - |
| ResBrowseScreen | ✅ Working Great | Extensive | No | - |
| UnitInfoScreen | ✅ Functional | Validated | No | Low - OnSuccess |
| RNAInfoScreen | ⚠️ Mostly Functional | Partial | No | Med - Phone screening |
| ResInfoScreen | ✅ Working Great | Extensive | No | - |
| IntakeScreen | ❌ Incomplete | Not started | **YES** | **HIGH** |
| RNACloseScreen | ✅ Fully Functional | Extensive | No | - |
| scrConvert | ⚠️ Needs Testing | Pending | No | Med - Validate |
| scrResidentAssociations | ⚠️ Integration Pending | Partial | No | Med - Provider add |
| scrResidentAccounts | ✅ Ready | Validated | Security | Med - Password |
| OrgScreen | ✅ Functional | Basic | No | - |
| scrServiceProviders | ✅ Functional | Basic | No | - |

---

**Related Documentation:**
- [[Current Status\|Current Status]] - Overall project status
- [[Workflow Status\|Workflow Status]] - Detailed workflow information
- [[Critical Tasks\|Critical Tasks]] - Pre-deployment task breakdown
- [[UP Clearview – Technical Documentation\|UP Clearview – Technical Documentation]] - Architecture and technical details
