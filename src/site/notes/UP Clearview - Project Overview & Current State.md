---
{"dg-publish":true,"permalink":"/up-clearview-project-overview-and-current-state/","tags":["gardenEntry"]}
---

**Version:** Wednesday Work - October 23, 2025
**Status:** Active Development - 75% Complete

---

## INTRODUCTION

Upper Post Clearview is a supportive housing operations management tool designed to make it easier for staff to have the information they need to do their jobs and potentially integrate with other systems in the future. The application consolidates scattered spreadsheet data into an integrated Power Apps solution, providing three core views—Units, Referrals & Applicants, and Current Residents—with comprehensive workflow support and service provider management.

### Purpose & Vision

UP Clearview replaces manual tracking across multiple spreadsheets with a centralized system that:
- Provides quick access to resident, unit, and referral information
- Supports key workflows (phone screening, intake, referrals, close-outs)
- Tracks service provider connections and account information
- Maintains housing unit inventory with eligibility requirements
- Enables data integration with HMIS and other systems

### How It Works

**Residents View**
The Current Residents tab displays all occupied units and their occupants, with color-coded unit types for quick visual identification. Staff can:
- Browse occupied units with resident names and unit types
- Dive into resident information pages showing key details called upon regularly
- Edit resident information directly from the page
- Exit residents from UP Clearview
- Access bill pay information for Housing Support program participants
- View connected service providers
- See household members for family units (upcoming feature)

**Referrals & Applicants View**
This view shows people who have been referred or have applied for current or upcoming vacant units. The layout is optimized for the referral process:
- Browse all active referrals and applicants (former referrals filtered out)
- Access detailed referral information with a different layout than residents
- Use workflow buttons for common tasks:
  - **Phone Screening Workflow** (complete) - Guides staff through phone screening questions
  - **Intake Workflow** (in progress) - Will process intake documentation and requirements
  - **Convert to Resident Workflow** (complete) - Converts referral to resident upon move-in with automatic unit updates
  - **Close-Out Referral Workflow** (complete) - Documents declined/withdrawn referrals with automatic status updates

**Units View**
The Units page can be filtered to show all units or only current and upcoming vacancies. Each unit displays:
- **Left panel (static information):**
  - Unit details (bedrooms, ADA status, unit type)
  - Administrative information (HMIS Program ID, subsidy administrator, referral source)
  - Eligibility information (income limits, occupancy requirements, homelessness criteria, disability documentation)
  - Everything needed when requesting a referral

- **Right panel (dynamic status):**
  - Current occupant (if occupied)
  - Referral information (if referral received)
  - "Receive Referral" workflow button (when applicable)
  - Date referral requested
  - Expected availability date (if offline)

**Service Provider & Account Management**
The associations interface connects residents with associated records:
- **Bill Pay Accounts:** Track usernames, billing information, and payment schedules for Housing Support participants
- **Service Providers:** Link residents to case managers and support organizations
  - Browse service providers with organizational affiliations
  - View provider details (name, role, program, contact information)
  - Access organization information (agency name, services provided, contacts)

### Current Completion Status

**Overall Progress:** 80%
- **Core Functionality:** 80% complete (browse, view, edit, navigate)
- **Workflows:** 75% complete (phone screening done, close-out done, convert to resident done, intake in progress)
- **Provider & Account Management:** 80% complete (basic CRUD working, some association features pending)

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

- ✅ **Archived Records** - Former resident storage
  - **Note:** Archiving workflow needs refinement

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

**Close-Out Workflow:**
- ✅ Form with close-out date, reason, and notes
- ✅ Automatic household status transition (adds "Former" prefix to current status)
- ✅ Unit cleanup: clears Referral Household, sets Has referral = false
- ✅ Data source refresh and navigation back to browse screen

**Convert to Resident Workflow:** ✅ **JUST COMPLETED (October 24, 2025)**
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

**Priority 2: Household Members Section**

**Current State:** Not yet started, planned for Friday

**Fix Required:**
- Create Participants list if not exists (or use existing)
- Link Participants to Current Residents (household composition)
- Build gallery and form for household members
- Add/edit/remove functionality

**Impact:** Cannot track household composition (affects occupancy calculations, HMIS reporting)
**Time Estimate:** 4-6 hours
**Priority:** HIGH - needed for compliance

---

**Priority 3: Missing OnSuccess Handlers**

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

**Priority 4: Exit Resident Workflow Completion**

**Current State:** Archives resident but doesn't clean up unit properly

**Missing:**
- Update resident's Household Status to "Former Occupant"
- Clear unit's Occupant Household field
- Set unit's Is occupied = false
- Update unit's Date Available to today

**Impact:** Units remain marked as occupied, data integrity issues
**Time Estimate:** 2-3 hours
**Priority:** HIGH

---

**Priority 5: Service Provider/Organization Creation**

**Issue:** Cannot add new providers or organizations through the app interface

**Likely Cause:** SharePoint permissions or Azure AD/Entra ID configuration

**Fix Required:**
- Check SharePoint list permissions (need "Add" permission)
- Verify Power Apps connection has proper delegated permissions
- May need Azure AD app registration updates
- Test with different user roles

**Impact:** Must use SharePoint directly to add providers (poor user experience)
**Time Estimate:** 2-4 hours (mostly troubleshooting permissions)
**Priority:** MEDIUM

---

**Priority 6: Password Management Solution**

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

**Priority 7: Archiving Refinement**

**Issue:** Archiving process needs improvement

**Current State:**
- Archived Records list exists
- Exit resident workflow archives data
- Format and completeness need review

**Fix Required:**
- Review what data gets archived
- Ensure archive format is useful for future reference
- Consider if former residents should be soft-deleted (status change) vs hard-deleted (removed from list)
- Verify referral close-out archiving works correctly

**Impact:** Historical data may not be properly preserved
**Time Estimate:** 2-3 hours
**Priority:** MEDIUM

---

**Priority 8: Hide Empty Fields in View Mode**

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

### Current State
Upper Post Clearview is a functional supportive housing management application at 80% completion. The core browse/view/edit functionality works well across all three main views (Units, Referrals & Applicants, Residents). Service provider and account management features are largely complete. Critical workflows like phone screening, referral close-out, and convert to resident are operational.

### Immediate Priorities (Before Deployment)
1. Complete IntakeScreen functionality
2. Implement household members section
3. Add OnSuccess handlers and user notifications
4. Resolve password management for account tracking
5. Complete exit resident workflow cleanup
6. Develop comprehensive testing suite

### Quick Win: Account Management
The account tracking interface can be deployed immediately once a password management solution is in place. Staff can use browser-built password managers (Chrome, Edge, Firefox all have this capability) while the password field is removed from the SharePoint list. This provides immediate value for Housing Support bill pay tracking.

### Timeline to Deployment
With focused development effort:
- **Critical workflow (Intake):** 6-8 hours
- **Missing features (Household members, OnSuccess handlers):** 6-8 hours
- **Exit workflow fix:** 2-3 hours
- **Testing suite development:** 4-8 hours
- **Polish (notifications, empty field hiding):** 3-4 hours

**Total estimated effort:** 21-31 hours of development + testing time

### Contact
While I may no longer be working for CommonBond come Friday, I would be happy to provide any support leading to implementation of this platform. Feel free to contact me at 971-716-4690 or jb_miles@berkeley.edu

---

**Document Version:** 1.0
**Date:** October 23, 2025
**Next Review:** After completing critical priorities
