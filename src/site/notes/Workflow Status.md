---
{"dg-publish":true,"permalink":"/workflow-status/","dgShowToc":true}
---

# Workflow Status

**Last Updated:** October 24, 2025

Detailed status of all major workflows with testing results and implementation notes.

---

## Complete Workflows ✅

### Phone Screening Workflow
**Status:** ✅ Operational
**Screen:** RNAInfoScreen → Phone Screening Modal

**Features:**
- Complete modal form with all screening questions
- Accessible from RNAInfoScreen workflow buttons
- Pre-populated with referral information
- Saves screening data to Referrals & Applicants list

**Known Issue:** Some issues reported in phone screening section (needs review)

**Testing Status:** Functional but needs additional validation

---

### Receive Referral Workflow
**Status:** ✅ Operational, Ready for Trial Use
**Screen:** UnitInfoScreen → Receive Referral Modal

**Features:**
- "Receive Referral" button on vacant units
- Modal form for creating new referrals linked to specific units
- Automatic unit linkage when receiving referral
- Display logic (disabled when occupied or has existing referral)
- Date Referred auto-populated to today's date
- Unit Type/subsidy information saved correctly

**Ready for:** Begin using and gathering feedback on what data should be captured for ETO integration

**Testing Status:** Functional, ready for real-world use

---

### Close-Out Workflow (Referral/Applicant)
**Status:** ✅ Fully Functional, Extensively Tested
**Screen:** RNACloseScreen

**Features:**
- Form with close-out date, reason, and notes
- **Exit Logging:** Creates Exit record capturing all close-out data
  - Parses household status to determine Stage (Applicant vs Referral)
  - Uses `StartsWith()` logic: checks if status begins with "Applicant" or "Referral"
  - Stores close-out date, reason, and notes for historical tracking
- Unit cleanup: clears Referral Household, sets Has referral = false, clears Referral Requested
- Removes referral/applicant record after archiving to Exit list
- Success notification and navigation back to browse screen

**Testing Results:**
- ✅ Stage parsing working correctly (Applicant vs Referral detection)
- ✅ Exit record created with all data
- ✅ Unit cleanup validated (all fields cleared correctly)
- ✅ Referral deletion confirmed
- ✅ Navigation working properly

**Production Ready:** Yes

---

### Exit Resident Workflow
**Status:** ✅ Fully Functional, Extensively Tested
**Screen:** ResInfoScreen → Exit Confirmation Modal

**Features:**
- Exit confirmation modal
- **Exit Logging:** Creates Exit record before removing resident
  - Captures resident name
  - Sets stage to "Resident"
  - Records exit date (today's date)
  - Provides archive for historical tracking and ETO integration
- Updates unit record:
  - Clears Occupant Household lookup
  - Sets Is occupied = false
- Removes resident from Current Residents list
- Success notification and navigation to browse screen

**Testing Results:**
- ✅ Exit record creation validated
- ✅ Unit update confirmed (cleared and marked vacant)
- ✅ Resident deletion working
- ✅ Navigation correct

**Production Ready:** Yes

---

### Convert to Resident Workflow
**Status:** ✅ Implementation Complete, ⚠️ Testing Pending
**Screen:** scrConvert

**Features:**
- Referral data review using FormViewer
- Input fields for:
  - Lease signing date (DatePicker)
  - Service providers (ComboBox multi-select)
- Creates Current Resident record with all referral data:
  - Title, Preferred Name, Preferred Language
  - Date of Birth, Phone, Email
  - Veteran Status, Disability, Benefits
  - Other Source of Income, Monthly Income
  - HMIS Record Number, County Case Number
  - Household Size, Lease Signing Date
- Handles lookup fields correctly:
  - Unit Number (passes entire lookup object)
  - Associated Providers (passes multi-select array)
- Updates Unit record:
  - Sets Occupant Household (with @odata.type)
  - Clears Referral Household
  - Clears Referral Requested
  - Sets Has referral = false
  - Sets Is occupied = true
- Removes referral record using `RemoveIf()` with ID match
- Navigates to new resident's info page in edit mode
- Success notification displayed

**Implementation Approach:**
- Built using **progressive debugging** methodology
- Started with minimal fields (Title + Household Status only)
- Added fields one at a time to identify issues
- Discovered field type mismatch: Veteran Status was Choice in one list, Text in another
- Fixed by ensuring field types match exactly

**Testing Status:**
- Code complete and validated in isolation
- Needs end-to-end testing with real data to verify:
  1. All fields transfer correctly
  2. Lookup fields handled properly
  3. Unit updates work as expected
  4. Navigation goes to correct screen with correct data

**Estimated Testing Time:** 1-2 hours

---

## Incomplete Workflows ❌

### Intake Workflow
**Status:** ❌ Critical Blocker - No Form Implementation
**Screen:** IntakeScreen

**Current State:**
- ✅ Screen loads, variables set correctly
- ✅ Eligibility requirements display from linked unit
- ✅ Organized sections: household composition, income restrictions, other eligibility
- ❌ Form2 is placeholder, not configured
- ❌ No data entry fields
- ❌ No save functionality
- ❌ No workflow progression logic

**Required for Deployment:** YES - critical workflow

**Implementation Plan:**
See [[Critical Tasks#CRITICAL-001\|Critical Tasks#CRITICAL-001]] for detailed 6-8 hour implementation plan including:
- Form field design
- Form build with data cards
- Conditional field visibility
- OnSuccess handler implementation
- Testing procedures

---

## Exit Logging System ✅

**Status:** Fully Implemented (October 24, 2025)

### Exits List
**Purpose:** Archive all resident and referral departures for historical tracking and ETO integration

**Fields:**
- **Title** - Person's name
- **Stage** - Choice field: "Resident", "Applicant", or "Referral"
- **Exit / Close-Out Date** - Date of exit/close-out
- **Close-Out Reason** - Choice field (optional, populated for referral close-outs)
- **Notes** - Multi-line text (optional, populated for referral close-outs)

### When Exit Records Are Created

**Resident Exit:**
- Triggered by: Exit button on ResInfoScreen
- Stage: "Resident"
- Date: Today's date
- Reason: Not captured (blank)
- Notes: Not captured (blank)

**Referral/Applicant Close-Out:**
- Triggered by: Close-out form on RNACloseScreen
- Stage: Parsed from Household Status ("Applicant" or "Referral")
- Date: User-selected close-out date
- Reason: User-selected from dropdown
- Notes: User-entered text

### Integration Readiness

The Exits list is now ready for:
- Historical reporting
- ETO integration planning
- Trend analysis (turnover rates, close-out reasons)
- Audit trails

**Next Step:** Determine what additional data points should be captured in Receive Referral workflow to support ETO integration.

---

## Workflow Summary Table

| Workflow | Status | Testing | Blocker | Screen |
|----------|--------|---------|---------|--------|
| Phone Screening | ✅ Operational | Needs review | No | RNAInfoScreen |
| Receive Referral | ✅ Operational | Validated | No | UnitInfoScreen |
| Intake | ❌ Not built | Not started | **YES** | IntakeScreen |
| Convert to Resident | ⚠️ Needs testing | Pending | No | scrConvert |
| Close-Out (Referral) | ✅ Fully functional | Extensive | No | RNACloseScreen |
| Exit (Resident) | ✅ Fully functional | Extensive | No | ResInfoScreen |
| Exit Logging | ✅ Complete | Validated | No | N/A (system) |

---

**Related Documentation:**
- [[Screen Status\|Screen Status]] - Detailed screen information
- [[Current Status\|Current Status]] - Overall project snapshot
- [[Critical Tasks\|Critical Tasks]] - Task breakdown for incomplete workflows
- [[UP Clearview – Technical Documentation\|UP Clearview – Technical Documentation]] - Technical implementation details
