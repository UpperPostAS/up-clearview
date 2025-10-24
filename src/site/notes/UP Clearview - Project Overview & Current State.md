---
{"dg-publish":true,"permalink":"/up-clearview-project-overview-and-current-state/","dgShowToc":true}
---

**Version:** Final Development Session - October 24, 2025
**Status:** Active Development - 85% Complete

---

## About This Report

This summary captures the current health of UP Clearview—what already works, what still needs attention, and the priorities driving the next build cycle. For navigation across the rest of the documentation set, head to the [[UP Clearview - Welcome & Index\|documentation hub]].

## Platform Snapshot

UP Clearview consolidates scattered spreadsheet data into an integrated Power Apps solution spanning three core views—Units, Referrals & Applicants, and Current Residents—with comprehensive workflow support and service provider management.

### Purpose & Vision

UP Clearview replaces manual tracking across multiple tools with a centralized system that:
- Provides quick access to resident, unit, and referral information
- Supports key workflows (phone screening, intake, referrals, close-outs)
- Tracks service provider connections and account information
- Maintains housing unit inventory with eligibility requirements
- Lays groundwork for integration with HMIS and other systems

### How It Works

**Residents View** — Shows occupied units and resident profiles with color-coded unit types. Staff can edit records, exit residents, review bill-pay data, and manage service provider associations. Household member tracking is next on deck.

**Referrals & Applicants View** — Optimized for the referral pipeline with workflow buttons for Phone Screening, Intake (in progress), Convert to Resident, and Close-Out. Former referrals stay archived but out of the active gallery.

**Units View** — Combines static eligibility details with real-time status. Staff can filter for vacancies, trigger "Receive Referral," and see who lives in or has been matched to each unit without leaving the screen.

**Service Provider & Account Management** — Brings provider contacts, organizational affiliations, and bill-pay accounts into one interface so staff can coordinate services and handle Housing Support billing on schedule.

## Current Completion Status

**Overall Progress:** 85%
- **Core Functionality:** 90% complete (browse, view, edit, navigate all working well)
- **Workflows:** 80% complete (phone screening done, close-out done, convert to resident done, exit resident with logging done, intake form still needed)
- **Provider & Account Management:** 80% complete (basic CRUD working, inline provider creation from associations still pending)
- **Exit Logging:** ✅ **NEW - COMPLETE** (Exits list captures all resident and referral departures)

---

## 1. WHAT IS COMPLETED ✅

### 1.1 Data Infrastructure

**SharePoint Lists (8 lists):**
- ✅ **Units** - Housing unit inventory with new Unit Type field replacing legacy Subsidy field
  - Boolean status fields: Is occupied, Has given notice, Has referral, Has move-in scheduled, Is offline
  - Rich eligibility criteria: income limits, homelessness requirements, disability documentation
  - Complete schema including: HMIS Program ID, referral sources, administrators

- ✅ **Referrals & Applicants** - Intake tracking with unit linkage
  - Household lifecycle from referral through applicant stages
  - Unit number and subsidy tracking
  - Date referred, household status, close-out fields

- ✅ **Current Residents** - Active occupant management
  - Unit assignments, lease dates, move-out tracking
  - Service provider linkages
  - Demographic and benefit information

- ✅ **Unit Types** - Configuration templates for eligibility requirements
  - Income restrictions, occupancy requirements
  - Homelessness criteria, disability documentation options
  - Linked to Units for automated eligibility display

- ✅ **Service Providers** - Case manager/agency contact directory
  - Individual provider records with organizational affiliations
  - Contact information, program assignments, active status

- ✅ **Agencies** - Organizations providing services
  - Organization contact details
  - Multi-select linking to service providers
  - Services provided descriptions

- ✅ **Account Information** - Bill pay tracking for Housing Support program
  - Provider accounts with usernames and account numbers
  - Billing frequency and recurring costs
  - Active/inactive status toggles
  - **Note:** Password storage needs to be addressed before full deployment

- ✅ **Exits** - Exit and close-out logging (formerly "Archived Records")
  - **NEW:** Comprehensive tracking of all departures
  - Fields: Title (name), Stage (Resident/Applicant/Referral), Exit/Close-Out Date, Close-Out Reason, Notes
  - Automatically populated when residents exit or referrals are closed out
  - Supports future ETO integration and historical reporting
  - **Status:** Fully implemented and tested (October 24, 2025)

### 1.2 Application Screens (13 screens)

**Main Browse Screens:**
- ✅ **UnitBrowseScreen** - Gallery of all units with occupancy status
  - Toggle to filter current and upcoming vacancies
  - Color-coded by unit type (LTH Housing Support, CoC, Section 8, HUD-VASH)
  - Displays occupancy status with detailed subtitles

- ✅ **RNABrowseScreen** - List of all referrals and applicants
  - Search and sort functionality
  - Status-based filtering (excludes "former" referrals)
  - Color-coded by unit type

- ✅ **ResBrowseScreen** - Active residents gallery
  - Sort by unit number
  - Unit type color coding
  - Filters out former residents

**Detail/Info Screens:**
- ✅ **UnitInfoScreen** - Detailed unit profile
  - Three-section viewer: Unit Info, Administrative Info, Eligibility Info
  - Editable form for unit status tracking
  - "Receive Referral" modal form
  - Dynamic display based on occupancy status

- ✅ **RNAInfoScreen** - Detailed referral/applicant record viewer/editor
  - Comprehensive data display
  - Workflow buttons (phone screening, intake, close-out)
  - Edit mode toggle

- ✅ **ResInfoScreen** - Individual resident detail screen
  - Household information
  - Service provider connections
  - Exit resident functionality
  - Edit/view mode toggle
  - **Planned enhancement:** Hide empty fields in view mode to reduce visual clutter

**Workflow Screens:**
- ⚠️ **IntakeScreen** - Intake workflow (IN PROGRESS)
  - Displays eligibility requirements from linked unit
  - Organized sections: household composition, income restrictions, other eligibility
  - Form implementation pending

- ✅ **RNACloseScreen** - Close-out workflow for declined/withdrawn referrals
  - Date closed, reason, notes fields
  - Automatic status transition to "Former Referral" or "Former Applicant"
  - Unit cleanup (clears referral household, resets flags)
  - Navigation back to browse screen

**Association/Management Screens:**
- ⚠️ **scrResidentAssociations** - Three-column associations view
  - Bill pay accounts section (complete)
  - Service providers section (complete)
  - Household members section (not yet started, planned for this week)

- ✅ **scrResidentAccounts** - Account information management
  - Form for Account Information list
  - Tracks provider accounts with usernames and billing
  - Conditional visibility based on "Is Active" and "Is Payable Account" toggles
  - New/Edit/View modes
  - **Ready for deployment** once password management solution is in place

- ✅ **OrgScreen** - Organizations/agencies management
  - Browse and edit Agencies list
  - Organization details, contact info, services provided
  - Multi-select service providers field

- ✅ **scrServiceProviders** - Service provider management
  - Browse and edit Service Providers list
  - Provider details (name, role, program, organization)
  - Links to organization via lookup field

### 1.2.1 Detailed Screen Status (Testing Results - October 24, 2025)

**Fully Functional Screens:**
- ✅ **ResBrowseScreen** - Tested extensively, working great
- ✅ **ResInfoScreen** - No issues, includes working exit workflow with exit record creation
- ✅ **UnitBrowseScreen** - Working great, vacancy filter now includes units with notice given
- ✅ **RNABrowseScreen** - Working great (Referrals & Applicants browse)
- ✅ **RNACloseScreen** - Extensive testing completed, fully functional
  - Creates Exit record with correct Stage parsing (Applicant vs Referral)
  - Properly clears unit referral data
  - Deletes referral record after archiving

**Screens with Known Issues:**
- ⚠️ **RNAInfoScreen** - Phone screening section has some issues
  - Main functionality works
  - Phone screening form needs review
  - Otherwise operational

**Screens Needing Completion:**
- ⚠️ **IntakeScreen** - Displays eligibility but lacks data entry form
  - Shows requirements correctly
  - Form implementation still required
  - Critical blocker for deployment

**Screens Needing Integration:**
- ⚠️ **scrResidentAssociations** - Service provider addition interface not integrated
  - Can view and select existing providers
  - Cannot add new service provider from this screen (must use SharePoint)
  - Workaround functional but not ideal user experience

**Convert to Resident Workflow:**
- ✅ **scrConvert** - Not yet tested after recent updates
  - Recently completed implementation
  - Needs testing to verify all fields transfer correctly
  - Should be ready but requires validation

### 1.3 Key Features Implemented

**Data Quality Improvements:**
- ✅ Referral form Unit Type data binding (saves subsidy correctly)
- ✅ Legacy "Unit Subsidy" field references migrated to new "Unit Type" field across 6 screens
- ✅ "Former" records filtered from browse screens (cleaner views of active data)
- ✅ Delegation warnings resolved for better performance at scale
  - Changed Sort() to SortByColumns() with internal field names
  - Improved filter operators for SharePoint delegation

**Referral Workflow:**
- ✅ "Receive Referral" button on vacant units
- ✅ Modal form for creating new referrals linked to specific units
- ✅ Automatic unit linkage when receiving referral
- ✅ Display logic (disabled when occupied or has existing referral)
- ✅ Date Referred auto-populated to today's date
- ✅ Unit Type/subsidy information saved correctly

**Phone Screening Workflow:**
- ✅ Complete modal form with all screening questions
- ✅ Accessible from RNAInfoScreen workflow buttons
- ✅ Pre-populated with referral information
- ✅ Saves screening data to Referrals & Applicants list

**Close-Out Workflow:** ✅ **COMPLETED WITH EXIT LOGGING (October 24, 2025)**
- ✅ Form with close-out date, reason, and notes
- ✅ **NEW:** Creates Exit record capturing all close-out data
  - Parses household status to determine Stage (Applicant vs Referral)
  - Stores close-out date, reason, and notes for historical tracking
- ✅ Unit cleanup: clears Referral Household, sets Has referral = false, clears Referral Requested
- ✅ Removes referral/applicant record after archiving to Exit list
- ✅ Success notification and navigation back to browse screen
- **Testing Status:** Extensively tested, fully operational

**Convert to Resident Workflow:** ✅ **COMPLETED (October 24, 2025)**
- ✅ scrConvert screen with referral data review
- ✅ Input fields for lease date, coordinator, and service providers
- ✅ Creates Current Resident record with all referral data
- ✅ Copies demographics, contact info, benefits, disability, veteran status
- ✅ Handles lookup fields (Unit Number, Associated Providers) correctly
- ✅ Updates Unit record (sets Occupant Household, clears Referral, marks occupied)
- ✅ Removes referral record from list
- ✅ Navigates to new resident's info page in edit mode
- ✅ Success notification displayed
- **Key Learning:** Built using progressive debugging approach - started with minimal fields, added one at a time to identify field type mismatches
- **Testing Status:** Implementation complete, validation testing pending

**Exit Resident Workflow:** ✅ **COMPLETED WITH EXIT LOGGING (October 24, 2025)**
- ✅ Exit confirmation modal on ResInfoScreen
- ✅ **NEW:** Creates Exit record before removing resident
  - Captures resident name, stage ("Resident"), exit date (today)
  - Provides archive for historical tracking and ETO integration
- ✅ Updates unit record (clears Occupant Household, sets Is occupied = false)
- ✅ Removes resident from Current Residents list
- ✅ Success notification and navigation to browse screen
- **Testing Status:** Extensively tested, fully operational

**Data Display:**
- ✅ Color-coded unit type palette system
  - LTH Housing Support: Light green (#C6EFCE)
  - Ramsey CoC: Light yellow (#FFF2CC)
  - Section 8 PBV: Light blue (#DDEBF7)
  - HUD-VASH: Light purple (#E5DFEC)

- ✅ Dynamic occupancy status rendering based on boolean fields
- ✅ Search and filter capabilities across browse screens
- ✅ Sort toggles (ascending/descending)

**Variable Standardization (In Progress):**
- ✅ varSelectedResident consistently used across Resident screens
- ✅ varSelectedReferral consistently used across RNA screens
- ⚠️ varActiveUnit used in Unit screens (considering rename to varSelectedUnit)
- ✅ Pre-loaded unit lookups (varResidentUnit, varReferralUnit) reduce redundant queries

**Account Management:**
- ✅ Conditional field visibility (shows billing fields only for payable accounts)
- ✅ Active/inactive toggle controls
- ✅ Form defaults to currently selected resident
- ✅ **Can be deployed immediately** with proper password management solution in place

---

## 2. WHAT NEEDS TO BE DONE BEFORE DEPLOYMENT 🔧

### 2.1 CRITICAL GAPS

**Priority 1: IntakeScreen Implementation**

**Current State:** Does not have functionality - displays eligibility requirements but has no data entry form

**Fix Required:**
- Build form with intake data fields
- Implement workflow progression logic
- Add validation and save handlers
- Create navigation to next steps

**Impact:** Intake workflow unavailable, requires manual workarounds
**Time Estimate:** 6-8 hours
**Priority:** HIGH - core workflow

---

**Priority 2: Household Members / Family Associations**

**Current State:** Not yet implemented

**Required:**
- Create or link to Participants/Household Members list
- Build gallery and form in scrResidentAssociations
- Link household members to Current Residents
- Add/edit/remove functionality

**Impact:** Cannot track household composition (affects occupancy calculations, HMIS reporting)
**Time Estimate:** 4-6 hours
**Priority:** HIGH - needed for compliance and accurate tracking

---

**Priority 3: Service Provider Addition from Association Screen**

**Issue:** Cannot add new service providers inline from scrResidentAssociations

**Current State:** Can view and select existing providers, but must use SharePoint to create new ones

**Fix Required:**
- Add modal form or inline creation interface
- Auto-select newly created provider
- Integrate into association workflow

**Impact:** Must use SharePoint directly to add providers (functional but not ideal UX)
**Time Estimate:** 2-3 hours
**Priority:** MEDIUM - workaround exists

---

**Priority 4: Missing OnSuccess Handlers**

**Issue:** Some forms lack save confirmation/refresh logic

**Affected Screens:**
- RNAInfoScreen main form - no OnSuccess handler
- UnitInfoScreen Form1 - no OnSuccess handler

**Fix Required:**
- Add OnSuccess handlers to refresh data and variables
- Add Notify() messages for user feedback
- Ensure variable consistency after saves

**Impact:** No user feedback after saves, no data refresh, potential stale data
**Time Estimate:** 1-2 hours
**Priority:** MEDIUM-HIGH

---

**Priority 5: Password Management Solution**

**Issue:** Account Information list contains plain-text passwords

**Current Recommendation:**
- Staff use browser-built password managers (Chrome, Edge, Firefox)
- Delete password field from Account Information list
- Provide guidance on using browser password tools
- Future enhancement: Integrate secure password storage

**Account Management Can Deploy Once Addressed:**
- The account tracking interface is ready for use
- All functionality works correctly
- Just needs password security solution before going live

**Impact:** Security risk if passwords stored in plain text
**Time Estimate:** 1-2 hours (documentation and field removal)
**Priority:** HIGH - security concern

---

**Priority 6: Data Capture Design for ETO Integration**

**Current State:** Exit logging and referral receive workflows are functional

**Planning Needed:**
- Determine what data should be captured in "Receive Referral" workflow
- Identify which fields in Exits list would facilitate ETO integration
- Consider what additional data points are needed for reporting

**Next Steps:**
- Begin trial use of receive referral workflow
- Collect feedback on data needs
- Plan integration points

**Impact:** Prepares system for external integrations
**Time Estimate:** Planning/discussion phase
**Priority:** MEDIUM - preparation for future work

---

**Priority 7: Hide Empty Fields in View Mode**

**Current State:** All fields show in info screens regardless of whether they contain data

**User Experience Issue:**
- Gives impression all fields must be filled out
- Creates visual clutter
- Designed as optional tool for staff use

**Fix Required:**
- Add Visible property to data cards: `Visible: =!IsBlank(Parent.Default) || Parent.DisplayMode = DisplayMode.Edit`
- Shows all fields in edit mode
- Hides empty fields in view mode

**Impact:** Improved user experience, clearer expectations about required vs optional data
**Time Estimate:** 1-2 hours
**Priority:** MEDIUM

### 2.2 HIGH PRIORITY (Should Complete)

**Variable Naming Consistency**

**Status:** In progress, continuing work tomorrow

**Current State:**
- Most variables use "varSelected..." pattern
- UnitBrowseScreen uses "varActiveUnit" (considering rename to varSelectedUnit)
- General cleanup and standardization ongoing

**Time:** Ongoing
**Priority:** MEDIUM (improves maintainability)

---

**Control and Component Naming Refactoring**

**Status:** Planned for tomorrow

**Goal:** Systematic renaming for consistency and clarity
- Form names (frmResidentEdit → frmResident, etc.)
- Gallery names
- Button and icon names
- Ensure consistency across all 13 screens

**Time:** Ongoing
**Priority:** MEDIUM (improves maintainability)

---

**Testing Suite Development**

**Current Gap:** No comprehensive testing plan or test cases written

**Needed:**
- Manual test scripts for each workflow
- Test cases for all forms (create, edit, delete)
- Edge case testing (null values, missing data, validation)
- Integration testing (workflow sequences)
- User acceptance testing scripts

**Impact:** Risk of bugs in production, no systematic validation process
**Time Estimate:** 4-8 hours to create comprehensive test suite
**Priority:** HIGH - needed before deployment

---

**User Notification Messages**

**Issue:** No user feedback after operations

**Current State:**
- Forms save silently
- No error messages on failures
- No success confirmations

**Fix Required:**
- Add Notify() calls to all OnSuccess handlers
- Add error handling with user-friendly messages
- Provide confirmation for destructive actions (delete, exit resident, close referral)

**Impact:** Users don't know if actions succeeded or failed
**Time Estimate:** 1-2 hours
**Priority:** MEDIUM

### 2.3 FUTURE ENHANCEMENTS

**Side Navigation for Provider-Centric Views**

**User Need:** Service provider view that bypasses unit/resident lists

**Current State:** Must navigate Residents → Associations → Providers

**Proposed Solution:**
- Add side navigation menu with icons/tabs
- Direct access to OrgScreen and scrServiceProviders
- Quick access to frequently used views
- Persistent across screens

**Implementation:**
- Add GroupContainer with navigation buttons to each screen
- Style with consistent colors/icons
- Could use Power Apps Component Framework (PCF) for advanced UI

**Time Estimate:** 4-6 hours
**Priority:** MEDIUM - improves user experience

---

**Add Service Provider from Association Screen**

**User Need:** Associate providers directly from scrResidentAssociations

**Current State:** Can only select existing providers

**Proposed Solution:**
- Add "+" button to add new provider
- Modal form for quick provider creation
- Auto-select newly created provider

**Time Estimate:** 2-3 hours
**Priority:** MEDIUM

---

**Account Provider Batch View**

**User Need:** View all accounts for a specific provider (e.g., Excel Energy) for batch payment processing

**Proposed Solution:**
- Two-pane layout on account screen
- Left pane: Account providers list
- Right pane: All accounts for selected provider
- Useful for monthly batch payments

**Status:** Planned, may be completed soon

**Time Estimate:** 2-3 hours
**Priority:** MEDIUM - workflow improvement

---

**Performance Optimization**

**Potential Issue:** Complex gallery formulas doing repeated LookUps

**Current Approach:**
- ResBrowseScreen: LookUp(Units...) for every resident
- RNABrowseScreen: LookUp(Units...) for every referral

**Optimization:**
- Use ClearCollect in OnVisible to pre-load joined data
- Store computed values in collections
- Reduce real-time lookup calls

**Time:** 3-4 hours
**Priority:** LOW (only needed if performance becomes an issue with larger datasets)

---

## SUMMARY & NEXT STEPS

### Current State (October 24, 2025)
Upper Post Clearview is a functional supportive housing management application at **85% completion**. The core browse/view/edit functionality works excellently across all three main views (Units, Referrals & Applicants, Residents). Service provider and account management features are largely complete.

**Major Milestone Achieved:** Exit logging is now fully implemented - both resident exits and referral close-outs create archive records in the Exits list, capturing name, stage, date, reason, and notes for historical tracking and future ETO integration.

**Tested & Working Well:**
- ResBrowseScreen, ResInfoScreen with exit workflow
- UnitBrowseScreen with vacancy filter (now includes units with notice)
- RNABrowseScreen
- RNACloseScreen (referral close-out) with exit logging
- Convert to Resident workflow (pending validation testing)

**Known Issues:**
- RNAInfoScreen phone screening section needs review
- IntakeScreen lacks data entry form (critical blocker)
- Service provider creation must be done in SharePoint (workaround functional)

### Immediate Priorities (Before Deployment)
1. **Complete IntakeScreen functionality** (6-8 hours) - Critical blocker
2. **Implement household members/family associations** (4-6 hours) - Compliance requirement
3. **Validate Convert to Resident workflow** (1-2 hours testing) - Just completed, needs verification
4. **Service provider inline creation** (2-3 hours) - UX improvement
5. **Add OnSuccess handlers** (1-2 hours) - User feedback
6. **Resolve password management** (1-2 hours) - Security
7. **Plan data capture for ETO integration** - Ongoing discussion

### Ready for Trial Use
- **Receive Referral workflow** - Begin using and gathering feedback on data needs
- **Exit tracking** - Fully functional, ready for ETO integration planning
- **Close-out workflow** - Extensively tested and operational

### Timeline to Deployment
With focused development effort:
- **Critical workflows (Intake, testing):** 8-10 hours
- **Household members implementation:** 4-6 hours
- **Service provider integration:** 2-3 hours
- **OnSuccess handlers and notifications:** 2-3 hours
- **Testing and validation:** 4-6 hours

**Estimated remaining effort:** 20-28 hours of development + comprehensive testing

### Contact
While I may no longer be working for CommonBond come Friday, I would be happy to provide any support leading to implementation of this platform. Feel free to contact me at 971-716-4690 or jb_miles@berkeley.edu

---

**Document Version:** 1.0
**Date:** October 23, 2025
**Next Review:** After completing critical priorities
