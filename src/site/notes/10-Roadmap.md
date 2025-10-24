---
{"dg-publish":true,"permalink":"/10-roadmap/","dgShowToc":true}
---

# Development Roadmap

**UP Clearview, Feature and Enhancement Planning**  
**Last Updated:** October 24, 2025

This version updates timing and phase structure. Work cannot resume before November 10, 2025. Earliest deployment to 1–2 sites is November 24, 2025.

---

## Timeline Overview

```
Now → Nov 10: No work
Nov 10 → Nov 22: Phase 1, Critical Path (pilot‑ready)
Nov 24: Initial deployment to 1–2 sites (earliest possible)
Nov 24 → ~Dec 9: Combined Pilot + Refinement loop
~Dec 10 → Ongoing: Broader Rollout
6+ months: Future Enhancements
```

**Earliest Work Start:** Monday, November 10, 2025  
**Earliest Initial Deployment:** Monday, November 24, 2025, 1–2 sites  
**Broader Rollout Window:** Begins about one month in, around December 10, 2025  
**UI Upgrades:** Hold for production launch milestone

---

## Phase 1: Critical Path, Must Do Before Pilot

**Goal:** Clear deployment blockers to reach pilot‑ready build.  
**Window:** Nov 10 → Nov 22  
**Effort Estimate:** 13–20 hours

### 1.1 IntakeScreen Form Implementation

**Priority:** CRITICAL  
**Effort:** 6–8 hours  
**Assigned:** TBD

**Tasks:**

- Design form fields and layout.
    
- Build form with data cards.
    
- Add conditional field visibility by unit type.
    
- Implement OnSuccess handler.
    
- Add validation.
    
- Test end‑to‑end workflow.
    

**Acceptance Criteria:**

- Form displays all required fields.
    
- Conditional fields show and hide correctly.
    
- Form saves and updates referral status.
    
- Intake date recorded.
    
- User receives success notification.
    

**Blocks:** Intake workflow, applicant progression.

---

### 1.2 Household Members Implementation

**Priority:** CRITICAL  
**Effort:** 4–6 hours  
**Assigned:** TBD

**Tasks:**

- Create Household Members SharePoint list.
    
- Design fields: Associated Resident, Relationship, DOB, Notes.
    
- Build gallery on `scrResidentAssociations`.
    
- Add create and edit form.
    
- Add delete functionality.
    
- Test CRUD operations.
    

**Acceptance Criteria:**

- View household members for a resident.
    
- Add, edit, and remove household members.
    
- Data persists correctly.
    

**Blocks:** Household composition tracking, HMIS compliance.

---

### 1.3 Convert to Resident Validation

**Priority:** HIGH  
**Effort:** 1–2 hours  
**Assigned:** TBD

**Tasks:**

- Create complete test referral.
    
- Execute conversion workflow.
    
- Verify field transfers, including lookups.
    
- Verify unit update and navigation.
    
- Document and fix issues.
    

**Acceptance Criteria:**

- All referral data transfers to resident record.
    
- Unit assignment works.
    
- Referral removed after conversion.
    
- No data loss or corruption.
    

**Blocks:** Applicant‑to‑resident progression.

---

### 1.4 OnSuccess Handlers

**Priority:** HIGH  
**Effort:** 1–2 hours  
**Assigned:** TBD

**Tasks:**

- Add OnSuccess to `RNAInfoScreen` `Form1`.
    
- Add OnSuccess to `UnitInfoScreen` `Form1`.
    
- Confirm notifications.
    
- Verify data refresh after save.
    

**Acceptance Criteria:**

- Success notification after save.
    
- Automatic refresh.
    
- Form reflects latest data.
    

**Blocks:** None.

---

### 1.5 Password Field Removal

**Priority:** HIGH, Security  
**Effort:** 1–2 hours  
**Assigned:** TBD

**Tasks:**

- Remove Password field from Account Information list.
    
- Remove password data card from `scrResidentAccounts`.
    
- Update documentation.
    
- Notify users and recommend browser password managers.
    

**Acceptance Criteria:**

- Password field removed from SharePoint and app forms.
    
- Docs updated and users informed.
    

**Blocks:** Security compliance.

---

**Phase 1 Output:** Pilot‑ready build by Nov 22 for possible deployment Nov 24.

---

## Phase 2/3: Combined Pilot and Refinement Loop

**Goal:** Validate with real users and iterate quickly in the same window.  
**Window:** Nov 24 → ~Dec 9  
**Sites:** 1–2 small sites, 6–10 units each

### Activities

- Deploy to pilot sites on or after Nov 24.
    
- Train coordinators, monitor usage.
    
- Weekly feedback collection.
    
- Fix critical issues as they arise.
    
- Implement prioritized enhancements during the same window.
    
- Maintain change log and quick release notes.
    

### Success Metrics

- Coordinators complete all workflows without workarounds.
    
- Data accuracy maintained.
    
- Measured time savings vs spreadsheets.
    
- User satisfaction survey results.
    
- Issue frequency and severity tracked.
    

**Deliverable:** Pilot report with prioritized enhancements, green‑light for broader rollout.

**Note on UI:** Hold major UI upgrades for production launch. Limit this loop to functional fixes, minor UX polish, and stability.

---

## Phase 4: Broader Rollout

**Goal:** Scale to additional sites.  
**Start:** ~Dec 10, 2025  
**Cadence:**

- Month 1 of rollout: 3–5 additional small sites.
    
- Months 2–3 of rollout: 2–3 medium sites, 10–20 units.
    
- Months 4–12: Evaluate, refine, and complete remaining sites including large sites.
    

### Training and Support

- Coordinator training sessions and office hours.
    
- Quick reference guides.
    
- Ongoing bug fix support.
    

**Deliverable:** Application in use across all target sites.

---

## Production Launch Milestone

**Scope:**

- Major user interface upgrades and visual refresh.
    
- Finalized documentation and training materials.
    
- Comprehensive regression test.
    

**Timing:** Align with first stable wave of broader rollout, post‑pilot, not earlier than mid‑December 2025. Exact date driven by pilot outcomes and stability.

---

## Future Enhancements, 6+ Months

Lower priority, after stable rollout and production launch.

### 5.1 Service Provider Inline Creation

**Priority:** MEDIUM  
**Effort:** 2–3 hours

### 5.2 Phone Screening Enhancements

**Priority:** MEDIUM  
**Effort:** 2–4 hours after investigation

### 5.3 Advanced Reporting and Analytics

**Priority:** MEDIUM  
**Effort:** 8–16 hours

### 5.4 ETO Integration

**Priority:** HIGH, Strategic  
**Effort:** 20–40 hours

### 5.5 Document Management

**Priority:** LOW  
**Effort:** 8–12 hours

### 5.6 Automated Notifications

**Priority:** LOW  
**Effort:** 6–10 hours

### 5.7 Mobile Optimization

**Priority:** LOW  
**Effort:** 12–20 hours

### 5.8 Bulk Import and Export

**Priority:** LOW  
**Effort:** 4–8 hours

---

## Prioritization Framework

1. Impact
    
2. Urgency
    
3. Effort
    
4. Strategic value
    
5. Dependencies
    
6. User feedback
    

**Priority Levels:** CRITICAL, HIGH, MEDIUM, LOW

---

## Change Request Process

1. Document request, include business case and urgency.
    
2. Submit to product team for backlog and initial estimate.
    
3. Prioritization review, quarterly and ad hoc for critical items.
    
4. Development and testing, then deploy.
    

---

## Roadmap Updates

- **Monthly:** Adjust based on pilot and rollout feedback.
    
- **Quarterly:** Major planning and reprioritization.
    
- **As needed:** Critical issues or strategic shifts.
    

**Last Review:** October 24, 2025  
**Next Review:** November 24, 2025

---

## Related Documentation

- Current Status, 02-Current-Status.md
    
- Known Issues, 09-Known-Issues.md
    
- Implementation Guide, 21-Implementation-Guide.md
    
- Testing Checklist, 22-Testing-Checklist.md
    

**Questions about roadmap priorities?** Contact the product team or development lead.
# Development Roadmap

**UP Clearview - Feature & Enhancement Planning**
**Last Updated:** October 24, 2025

This document outlines the development priorities for UP Clearview, from critical deployment blockers through future enhancements.

---

## Timeline Overview

```
Current State (85% complete)
    ↓
Critical Path (2-3 weeks)
    ↓
Small Site Pilot (4-6 weeks)
    ↓
Refinement & Polish (2-3 weeks)
    ↓
Broader Rollout (ongoing)
    ↓
Future Enhancements (6+ months)
```

**Estimated Time to Pilot-Ready:** 2-3 weeks
**Estimated Time to Production-Ready:** 8-12 weeks (including pilot feedback)

---

## Phase 1: Critical Path (Must Do Before Pilot)

**Goal:** Complete deployment blockers
**Timeline:** 2-3 weeks
**Effort Estimate:** 13-20 hours

### 1.1 IntakeScreen Form Implementation
**Priority:** CRITICAL
**Effort:** 6-8 hours
**Assigned:** TBD

**Tasks:**
- Design form fields and layout
- Build form with data cards
- Add conditional field visibility based on unit type
- Implement OnSuccess handler
- Add validation
- Test complete workflow

**Acceptance Criteria:**
- Form displays all required fields
- Conditional fields show/hide correctly
- Form saves and updates referral status
- Intake date recorded
- User receives success notification

**Blocks:** Intake workflow, applicant progression

---

### 1.2 Household Members Implementation
**Priority:** CRITICAL
**Effort:** 4-6 hours
**Assigned:** TBD

**Tasks:**
- Create Household Members SharePoint list
- Design fields: Associated Resident, Relationship, DOB, Notes
- Build gallery on scrResidentAssociations
- Add create/edit form
- Add delete functionality
- Test CRUD operations

**Acceptance Criteria:**
- Can view household members for a resident
- Can add new household member
- Can edit existing household member
- Can remove household member
- Data persists correctly

**Blocks:** Household composition tracking, HMIS compliance

---

### 1.3 Convert to Resident Validation
**Priority:** HIGH
**Effort:** 1-2 hours
**Assigned:** TBD

**Tasks:**
- Create complete test referral
- Execute conversion workflow
- Verify all fields transfer correctly
- Verify lookup fields handled properly
- Verify unit updates correctly
- Verify navigation works
- Document any issues found
- Fix issues if discovered

**Acceptance Criteria:**
- All referral data transfers to resident record
- Unit assignment works correctly
- Referral removed after conversion
- No data loss or corruption
- Workflow completes successfully

**Blocks:** Applicant-to-resident progression

---

### 1.4 OnSuccess Handlers
**Priority:** HIGH
**Effort:** 1-2 hours
**Assigned:** TBD

**Tasks:**
- Add OnSuccess handler to RNAInfoScreen Form1
- Add OnSuccess handler to UnitInfoScreen Form1
- Test confirmation notifications
- Test data refresh after save

**Acceptance Criteria:**
- Users see success notification after save
- Data refreshes automatically
- Form updates with latest data
- No manual refresh needed

**Blocks:** None, but significantly improves UX

---

### 1.5 Password Field Removal
**Priority:** HIGH (Security)
**Effort:** 1-2 hours
**Assigned:** TBD

**Tasks:**
- Remove Password field from Account Information list
- Remove password data card from scrResidentAccounts
- Update documentation
- Communicate change to users
- Recommend browser password managers

**Acceptance Criteria:**
- Password field removed from SharePoint
- Form no longer shows password field
- Documentation updated
- Users informed of change

**Blocks:** Security compliance

---

**Phase 1 Total:** 13-20 hours over 2-3 weeks

---

## Phase 2: Small Site Pilot

**Goal:** Validate application with real users and gather feedback
**Timeline:** 4-6 weeks
**Sites:** 2-3 small sites (6-10 units each)

### 2.1 Pilot Site Selection
**Criteria:**
- Small size (6-10 units)
- Single coordinator per site
- Mix of unit types
- Willing to provide feedback
- Can tolerate minor issues

### 2.2 Pilot Activities
- Deploy to pilot sites
- Train coordinators
- Monitor usage
- Gather feedback weekly
- Fix critical issues as they arise
- Document pain points
- Identify desired enhancements

### 2.3 Success Metrics
- Coordinators can complete all workflows
- Data accuracy maintained
- Time savings vs. spreadsheets quantified
- User satisfaction survey results
- Issue frequency and severity tracked

**Deliverable:** Pilot feedback report with prioritized enhancement list

---

## Phase 3: Refinement & Polish

**Goal:** Address pilot feedback and prepare for broader rollout
**Timeline:** 2-3 weeks
**Effort Estimate:** Variable based on pilot feedback

### 3.1 Pilot Feedback Implementation
**Priority:** HIGH
**Effort:** TBD

**Tasks:**
- Review pilot feedback
- Prioritize enhancement requests
- Implement high-priority changes
- Fix bugs discovered during pilot
- Re-test affected workflows

### 3.2 UX Polish
**Priority:** MEDIUM
**Effort:** 4-8 hours

**Tasks:**
- Improve visual consistency
- Enhance navigation clarity
- Add helpful tooltips/instructions
- Improve error messages
- Optimize form layouts

### 3.3 Comprehensive Testing
**Priority:** HIGH
**Effort:** 4-6 hours

**Tasks:**
- Build complete test plan
- Create test data set
- Execute full regression test
- Document test results
- Fix any issues found

### 3.4 Documentation Updates
**Priority:** MEDIUM
**Effort:** 2-4 hours

**Tasks:**
- Update all docs based on changes
- Create user training materials
- Record video walkthroughs
- Build FAQ document

**Deliverable:** Production-ready application with complete documentation

---

## Phase 4: Broader Rollout

**Goal:** Deploy to additional sites
**Timeline:** Ongoing (months 4-12)

### 4.1 Site Prioritization
**Criteria:**
- Site size (small first, then medium, then large)
- Coordinator capacity
- Current system pain points
- Strategic value

### 4.2 Rollout Cadence
- Month 4-5: 3-5 additional small sites
- Month 6-7: 2-3 medium sites (10-20 units)
- Month 8-9: Evaluate and refine
- Month 10-12: Large sites or remainder

### 4.3 Training & Support
- Coordinator training sessions
- Office hours for questions
- Quick reference guides
- Ongoing bug fix support

**Deliverable:** Application in use across all sites

---

## Future Enhancements (6+ Months)

**Goal:** Advanced features and integrations
**Priority:** LOW (After successful rollout)

### 5.1 Service Provider Inline Creation
**Priority:** MEDIUM
**Effort:** 2-3 hours

**Description:**
Add ability to create new service providers directly from scrResidentAssociations without navigating away.

**Benefits:**
- Improved workflow efficiency
- Better user experience
- Fewer context switches

---

### 5.2 Phone Screening Enhancements
**Priority:** MEDIUM
**Effort:** 2-4 hours (after investigation)

**Description:**
Address issues with phone screening section and improve functionality based on user feedback.

**Tasks:**
- Document specific issues
- Gather user requirements
- Implement fixes and enhancements
- Test thoroughly

---

### 5.3 Advanced Reporting & Analytics
**Priority:** MEDIUM
**Effort:** 8-16 hours

**Description:**
Build reporting capabilities for program management and compliance.

**Potential Reports:**
- Occupancy rates over time
- Turnover analysis
- Average time from referral to move-in
- Exit reasons breakdown
- Unit type utilization
- Service provider connections by program

**Approach:**
- Power BI integration, OR
- Built-in Power Apps dashboards

---

### 5.4 ETO Integration
**Priority:** HIGH (Strategic)
**Effort:** 20-40 hours (complex)

**Description:**
Integrate with existing ETO (Efforts to Outcomes) system for bi-directional data sync.

**Benefits:**
- Eliminate duplicate data entry
- Sync case management data
- Improve data accuracy
- Enable cross-system reporting

**Requires:**
- ETO API access
- Data mapping
- Sync logic
- Error handling
- Testing with both systems

**Dependencies:**
- Exits list operational (✅ Complete)
- Stable application (after rollout)
- IT/vendor support

---

### 5.5 Document Management
**Priority:** LOW
**Effort:** 8-12 hours

**Description:**
Add ability to upload and attach documents to resident/referral records.

**Features:**
- Upload ID copies
- Store lease documents
- Attach verification documents
- Organize by category

**Approach:**
- SharePoint document library integration
- Link to resident/referral records
- Version control
- Access permissions

---

### 5.6 Automated Notifications
**Priority:** LOW
**Effort:** 6-10 hours

**Description:**
Send automated email/text notifications for key events.

**Notification Triggers:**
- New referral received
- Intake completed
- Lease signing scheduled
- Resident notice given (vacancy upcoming)
- Exit completed

**Approach:**
- Power Automate flows
- Template-based messages
- User preference settings

---

### 5.7 Mobile Optimization
**Priority:** LOW
**Effort:** 12-20 hours

**Description:**
Optimize interface for mobile devices (phones/tablets).

**Tasks:**
- Review current mobile experience
- Redesign forms for mobile
- Test on various devices
- Optimize navigation for touch
- Consider separate mobile screens

**Benefits:**
- Field access for coordinators
- Flexibility for remote work
- Better accessibility

---

### 5.8 Bulk Import/Export
**Priority:** LOW
**Effort:** 4-8 hours

**Description:**
Add ability to bulk import residents or export data for reporting.

**Features:**
- CSV import for residents
- Excel export of current data
- Template downloads
- Validation on import
- Error reporting

**Use Cases:**
- Initial data migration for new sites
- Backup/archival
- External reporting needs

---

## Backlog (Unprioritized Ideas)

Ideas captured for future consideration:

- **Multi-Site Dashboard** - Executive view across all sites
- **Automated Compliance Checks** - Flag missing documentation
- **Historical Trends** - Year-over-year comparisons
- **Custom Field Configurability** - Let admins add custom fields without dev work

---

## Prioritization Framework

Issues and features are prioritized based on:

1. **Impact** - How many users/sites affected?
2. **Urgency** - Deployment blocker or nice-to-have?
3. **Effort** - Hours required to implement
4. **Strategic Value** - Alignment with organizational goals
5. **Dependencies** - Blocks other work?
6. **User Feedback** - Requested by coordinators?

**Priority Levels:**
- **CRITICAL** - Deployment blocker, must fix before pilot
- **HIGH** - Should fix before pilot, significant impact
- **MEDIUM** - Fix after pilot, moderate impact
- **LOW** - Future enhancement, minor impact

---

## Related Documentation

- [Current Status](02-Current-Status.md) - Where we are today
- [Known Issues](09-Known-Issues.md) - Detailed issue tracking
- [Implementation Guide](21-Implementation-Guide.md) - Deployment procedures
- [Testing Checklist](22-Testing-Checklist.md) - Validation requirements

---

**Questions about roadmap priorities?** Contact the product team or development lead.
