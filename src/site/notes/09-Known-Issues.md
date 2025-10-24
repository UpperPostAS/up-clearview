---
{"dg-publish":true,"permalink":"/09-known-issues/","dgShowToc":true}
---

# Known Issues

**Bug Tracking & Workarounds**
**Last Updated:** October 24, 2025

This document tracks all known issues in UP Clearview, organized by severity. Each issue includes the problem description, impact, workaround (if available), and proposed fix.

---

## Critical Blockers (Prevent Deployment) 🔴

### ISSUE-001: IntakeScreen Form Not Built
**Component:** IntakeScreen
**Status:** Open
**Priority:** CRITICAL
**Estimated Fix:** 6-8 hours

**Problem:**
IntakeScreen displays eligibility requirements but has no data entry form. Form2 is a placeholder with no fields, save functionality, or workflow logic.

**Impact:**
- Cannot complete intake workflow
- Blocks referral-to-applicant progression
- Core functionality unavailable

**Workaround:**
None - workflow cannot be completed without this form.

**Proposed Fix:**
1. Build form with required fields:
   - Intake Completion Date
   - Completed By
   - Documentation checklist (ID, SS card, income verification, etc.)
   - Verification notes
   - Eligibility confirmation
2. Add conditional field visibility based on unit type requirements
3. Implement OnSuccess handler to update referral status
4. Add validation
5. Test complete workflow

**Related Documentation:**
- [IntakeScreen](screens/07h-IntakeScreen.md)
- [Intake Workflow](workflows/08c-Intake.md)

---

### ISSUE-002: Household Members Section Missing
**Component:** scrResidentAssociations
**Status:** Open
**Priority:** CRITICAL
**Estimated Fix:** 4-6 hours

**Problem:**
The Household Members section on scrResidentAssociations is a placeholder. No SharePoint list exists, no gallery is built, and there's no add/edit/remove functionality.

**Impact:**
- Cannot track household composition
- Affects occupancy calculations
- Missing HMIS reporting requirement
- Compliance issue for some programs

**Workaround:**
Track household members in notes or external system.

**Proposed Fix:**
1. Create Household Members SharePoint list
2. Add fields: Associated Resident (lookup), Relationship, Date of Birth, Notes
3. Build gallery on scrResidentAssociations to display household members
4. Add form for creating/editing household members
5. Add delete functionality
6. Test CRUD operations

**Related Documentation:**
- [scrResidentAssociations](screens/07j-scrResidentAssociations.md)
- [Data Model](03-Data-Model.md#household-members)

---

## High Priority (Should Fix Before Deployment) 🟠

### ISSUE-003: OnSuccess Handlers Missing
**Components:** RNAInfoScreen, UnitInfoScreen
**Status:** Open
**Priority:** HIGH
**Estimated Fix:** 1-2 hours total

**Problem:**
Main forms on RNAInfoScreen and UnitInfoScreen (Form1) lack OnSuccess handlers. After users save edits, they receive no confirmation and data doesn't refresh automatically.

**Impact:**
- Poor user experience
- Users unsure if save succeeded
- Must manually refresh to see changes
- Could lead to duplicate edits

**Workaround:**
Forms do save correctly. Users can refresh the page or navigate away and back to see updates.

**Proposed Fix:**

For RNAInfoScreen:
```powerapps
Form1.OnSuccess: |-
  =Notify("Referral information updated successfully.", NotificationType.Success);
  Refresh('Referrals & Applicants');
  Set(varSelectedRNA, Form1.LastSubmit)
```

For UnitInfoScreen:
```powerapps
Form1.OnSuccess: |-
  =Notify("Unit information updated successfully.", NotificationType.Success);
  Refresh(Units);
  Set(varSelectedUnit, Form1.LastSubmit)
```

**Related Documentation:**
- [RNAInfoScreen](screens/07e-RNAInfoScreen.md)
- [UnitInfoScreen](screens/07d-UnitInfoScreen.md)

---

### ISSUE-004: Password Field Security Risk
**Component:** scrResidentAccounts, Account Information list
**Status:** Open
**Priority:** HIGH
**Estimated Fix:** 1-2 hours

**Problem:**
Account Information list stores passwords in plain text. This is a security vulnerability.

**Impact:**
- Passwords visible to anyone with SharePoint access
- Security compliance risk
- Data breach potential

**Workaround:**
Leave password field empty; use browser-based password managers.

**Proposed Fix:**
1. Remove Password field from Account Information list in SharePoint
2. Remove password data card from scrResidentAccounts form
3. Update documentation to recommend browser password managers
4. Communicate change to users

**Alternative Options:**
- Implement third-party password vault integration
- Build encrypted storage (complex, not recommended)

**Related Documentation:**
- [scrResidentAccounts](screens/07k-scrResidentAccounts.md)
- [Data Model](03-Data-Model.md#account-information)

---

### ISSUE-005: Convert to Resident Workflow Untested
**Component:** scrConvert
**Status:** Open (code complete, testing pending)
**Priority:** HIGH
**Estimated Fix:** 1-2 hours validation testing

**Problem:**
Convert to Resident workflow implementation is complete but has not been tested end-to-end with real data.

**Impact:**
- Unknown if all fields transfer correctly
- Lookup field handling unvalidated
- Unit update logic unconfirmed
- Could fail in production

**Workaround:**
N/A - workflow should not be used until validated.

**Testing Plan:**
1. Create complete test referral with all fields populated
2. Assign to specific unit
3. Set status to "Applicant, Approved"
4. Execute conversion
5. Verify all fields transferred to resident record
6. Verify lookup fields (Unit, Associated Providers) correct
7. Verify unit updated (occupant set, referral cleared)
8. Verify referral deleted
9. Verify navigation to correct resident detail screen
10. Check for any errors or data inconsistencies

**Related Documentation:**
- [scrConvert](screens/07g-scrConvert.md)
- [Convert to Resident Workflow](workflows/08d-Convert-to-Resident.md)

---

## Medium Priority (UX Issues, Non-Blocking) 🟡

### ISSUE-006: Service Provider Inline Creation Not Available
**Component:** scrResidentAssociations
**Status:** Open
**Priority:** MEDIUM
**Estimated Fix:** 2-3 hours

**Problem:**
Cannot create new service providers from the scrResidentAssociations screen. Users can only select existing providers from a dropdown.

**Impact:**
- Poor user experience when new provider needs to be added
- Forces context switch to SharePoint or scrServiceProviders screen
- Slows down workflow

**Workaround:**
1. Go to SharePoint and create service provider directly, OR
2. Navigate to scrServiceProviders screen, create provider, then return

**Proposed Fix:**
Add modal form on scrResidentAssociations for creating new service providers:
1. Add "New Provider" button next to provider selection
2. Show modal with provider creation form
3. On save, add new provider to database
4. Refresh provider list
5. Auto-select newly created provider

**Related Documentation:**
- [scrResidentAssociations](screens/07j-scrResidentAssociations.md)
- [scrServiceProviders](screens/07m-scrServiceProviders.md)

---

### ISSUE-007: Phone Screening Section Issues
**Component:** RNAInfoScreen
**Status:** Open (needs investigation)
**Priority:** MEDIUM
**Estimated Fix:** 2-4 hours (TBD after investigation)

**Problem:**
Phone screening section on RNAInfoScreen has some issues. Main functionality works but edge cases may have problems.

**Impact:**
- Unknown - needs specific documentation
- Main phone screening workflow appears functional

**Workaround:**
Use phone screening modal for primary workflow.

**Next Steps:**
1. Document specific issues observed
2. Reproduce issues
3. Identify root causes
4. Estimate fix effort
5. Prioritize based on impact

**Related Documentation:**
- [RNAInfoScreen](screens/07e-RNAInfoScreen.md)
- [Phone Screening Workflow](workflows/08b-Phone-Screening.md)

---

## Low Priority (Minor Issues) 🟢

### ISSUE-008: Missing Comprehensive Test Suite
**Component:** All
**Status:** Open
**Priority:** LOW
**Estimated Effort:** 4-6 hours

**Problem:**
No automated or comprehensive manual test suite exists. Testing is ad-hoc.

**Impact:**
- Regressions could be introduced without detection
- Difficult to validate changes
- Time-consuming to test before deployment

**Workaround:**
Manual testing of critical workflows before each release.

**Proposed Fix:**
1. Create comprehensive test plan document
2. Build test data set in SharePoint
3. Document test procedures for each screen
4. Document test procedures for each workflow
5. Create regression test checklist
6. Consider Power Apps Test Studio for automated tests

**Related Documentation:**
- [Testing Philosophy](12-Testing-Philosophy.md)
- [Testing Procedures](13-Testing-Procedures.md)

---

## Resolved Issues ✅

### RESOLVED-001: Exit Resident Doesn't Update Unit
**Resolution Date:** October 24, 2025

**Problem (Original):**
When resident exited, unit record wasn't updated. Occupant Household wasn't cleared and Is occupied wasn't set to false.

**Fix Applied:**
Updated Exit Resident workflow to:
- Clear unit's Occupant Household lookup
- Set Is occupied = false
- Update all relevant unit flags

**Status:** ✅ Complete and extensively tested

---

### RESOLVED-002: Exit Logging Not Implemented
**Resolution Date:** October 24, 2025

**Problem (Original):**
No historical tracking of exits and close-outs. Data lost when residents/referrals removed from system.

**Fix Applied:**
- Created Exits SharePoint list
- Added exit record creation to Exit Resident workflow
- Added exit record creation to Close-Out Referral workflow
- Captures: Name, Stage, Date, Reason, Notes

**Status:** ✅ Complete and extensively tested

---

### RESOLVED-003: Close-Out Workflow Missing
**Resolution Date:** October 24, 2025

**Problem (Original):**
No way to close out unsuccessful referrals/applicants.

**Fix Applied:**
- Built RNACloseScreen with close-out form
- Implemented exit logging
- Added unit cleanup logic
- Added referral record removal

**Status:** ✅ Complete and extensively tested

---

## Issue Summary

| Priority | Open Issues | Estimated Total Effort |
|----------|-------------|------------------------|
| Critical (🔴) | 2 | 10-14 hours |
| High (🟠) | 3 | 3-5 hours |
| Medium (🟡) | 2 | 4-7 hours |
| Low (🟢) | 1 | 4-6 hours |
| **Total** | **8** | **21-32 hours** |

**Critical Path to Deployment:** 10-14 hours (Critical issues only)

---

## Reporting New Issues

When reporting a new issue, include:

1. **Component/Screen** - Where the issue occurs
2. **Priority** - Critical/High/Medium/Low
3. **Problem Description** - What's wrong
4. **Steps to Reproduce** - How to see the issue
5. **Expected Behavior** - What should happen
6. **Actual Behavior** - What actually happens
7. **Impact** - Who/what is affected
8. **Workaround** - Temporary solution (if any)
9. **Screenshots** - Visual evidence (if applicable)

---

**Related Documentation:**
- [Current Status](02-Current-Status.md) - Overall project status
- [Roadmap](10-Roadmap.md) - Development priorities
- [Testing Procedures](13-Testing-Procedures.md) - How to test for issues

---

**Questions?** Contact the development team or add comments to this document.
