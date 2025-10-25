---
{"dg-publish":true,"permalink":"/readme/","tags":["gardenEntry"]}
---

# UP Clearview Documentation

**UP Clearview** is CommonBond Communities' operations hub for supportive housing. The app centralizes unit availability, referral/resident pipelines, service provider coordination, and Housing Support bill-pay—providing a single source of truth so staff can keep people housed without jumping between spreadsheets.

**Current Status:** ~85% complete | Core workflows operational | Ready for small-site pilot testing

**Documentation Status:** Recently refactored! Core reference docs and all screen/workflow docs are complete. Some guides still need to be streamlined from originals. See [RESTRUCTURING_COMPLETE.md](RESTRUCTURING_COMPLETE.md) for details.

---

## Quick Start

**New to UP Clearview?**
- Start with [Introduction](01-Introduction.md) for overview and key concepts
- See [Current Status](02-Current-Status.md) for what's working and what's next

**Using the App?**
- [Data Model](03-Data-Model.md) - Understanding the SharePoint lists
- Browse [Screens](screens/) or [Workflows](workflows/) for specific features
- [Testing Guide](UP%20Clearview%20-%20Testing%20Guide.md) - Comprehensive testing procedures

**Deploying to a New Site?**
- [Implementation Guide](UP%20Clearview%20-%20Implementation%20Guide.md) - Complete deployment playbook
- Review [screens](screens/) and [workflows](workflows/) documentation

**Building or Troubleshooting?**
- Browse [individual screen docs](screens/) for technical details
- See [individual workflow docs](workflows/) for formulas and implementation
- [Known Issues](09-Known-Issues.md) - Current bugs and workarounds
- [Roadmap](10-Roadmap.md) - Development priorities

---

## Documentation Map

### Getting Started
- **[01 - Introduction](01-Introduction.md)** - What is UP Clearview, who it's for, key capabilities
- **[02 - Current Status](02-Current-Status.md)** - What's done, what's in progress, deployment readiness
- **[03 - Data Model](03-Data-Model.md)** - SharePoint schema, lists, and relationships

### Developer Reference
*These detailed technical docs are planned - for now see individual screen/workflow docs and the original [Testing Guide](UP%20Clearview%20-%20Testing%20Guide.md)*
- **04 - Architecture** *(planned)* - Power Apps structure, design patterns, philosophy
- **05 - Progressive Debugging** *(planned)* - How to troubleshoot when things break
- **06 - Component Reference** *(planned)* - Controls, formulas, common patterns

### Screens (Detailed Reference)
- **[07a - UnitBrowseScreen](screens/07a-UnitBrowseScreen.md)** - Browse vacant and upcoming units
- **[07b - RNABrowseScreen](screens/07b-RNABrowseScreen.md)** - Browse referrals and applicants
- **[07c - ResBrowseScreen](screens/07c-ResBrowseScreen.md)** - Browse current residents
- **[07d - UnitInfoScreen](screens/07d-UnitInfoScreen.md)** - View unit details, receive referrals
- **[07e - RNAInfoScreen](screens/07e-RNAInfoScreen.md)** - View referral/applicant details
- **[07f - ResInfoScreen](screens/07f-ResInfoScreen.md)** - View resident details, exit resident
- **[07g - scrConvert](screens/07g-scrConvert.md)** - Convert referral to resident
- **[07h - IntakeScreen](screens/07h-IntakeScreen.md)** - Intake workflow (⚠️ form incomplete)
- **[07i - RNACloseScreen](screens/07i-RNACloseScreen.md)** - Close-out referral/applicant
- **[07j - scrResidentAssociations](screens/07j-scrResidentAssociations.md)** - Manage providers and accounts
- **[07k - scrResidentAccounts](screens/07k-scrResidentAccounts.md)** - Bill pay account management
- **[07l - OrgScreen](screens/07l-OrgScreen.md)** - Manage agencies
- **[07m - scrServiceProviders](screens/07m-scrServiceProviders.md)** - Manage service providers

### Workflows (Step-by-Step)
- **[08a - Receive Referral](workflows/08a-Receive-Referral.md)** - Create new referral record
- **[08b - Phone Screening](workflows/08b-Phone-Screening.md)** - Screen referrals by phone
- **[08c - Intake](workflows/08c-Intake.md)** - Complete intake process (⚠️ incomplete)
- **[08d - Convert to Resident](workflows/08d-Convert-to-Resident.md)** - Transition applicant to resident
- **[08e - Exit Resident](workflows/08e-Exit-Resident.md)** - Process resident move-out
- **[08f - Close-Out Referral](workflows/08f-Close-Out-Referral.md)** - Close unsuccessful referrals

### Issues & Planning
- **[09 - Known Issues](09-Known-Issues.md)** - Current bugs, blockers, and workarounds
- **[10 - Roadmap](10-Roadmap.md)** - Development priorities and future features

### User & Testing Guides
*Streamlined focused guides are planned - for now use the comprehensive originals below*
- **[UP Clearview - Testing Guide](UP%20Clearview%20-%20Testing%20Guide.md)** - Comprehensive testing procedures (original 60 KB guide)
- **11 - User Guide** *(planned)* - Focused day-in-the-life walkthrough
- **12 - Testing Philosophy** *(planned)* - Progressive debugging approach
- **13 - Testing Procedures** *(planned)* - Screen and workflow test cases
- **14 - Test Data Setup** *(planned)* - Creating test data in SharePoint

### Implementation & Deployment
*Streamlined focused guides are planned - for now use the comprehensive original below*
- **[UP Clearview - Implementation Guide](UP%20Clearview%20-%20Implementation%20Guide.md)** - Complete deployment playbook (original 82 KB guide)
- **21 - Implementation Guide** *(planned)* - Focused deployment procedures
- **22 - Testing Checklist** *(planned)* - Pre-deployment validation
- **23 - Field Customization** *(planned)* - Adapting for different site types
- **24 - Rollout Strategy** *(planned)* - Phased deployment plan

---

## Key Concepts

### Core Workflows
UP Clearview tracks residents through their entire journey:
1. **Receive Referral** - Partner refers someone for housing
2. **Phone Screening** - Initial eligibility check
3. **Intake** - Gather detailed information and documents
4. **Convert to Resident** - Approved applicant signs lease
5. **Ongoing Management** - Track services, accounts, household
6. **Exit** - Resident moves out or referral closes

### Data Model
Eight core SharePoint lists power the application:
- **Units** - Housing inventory with eligibility and status
- **Referrals & Applicants** - Pre-housing pipeline
- **Current Residents** - Active tenants
- **Exits** - Historical exit/close-out records
- **Service Providers** - Case managers and support staff
- **Agencies** - Organizations providing services
- **Account Information** - Bill pay tracking (Housing Support)
- **Unit Types** - Eligibility requirement templates

See [Data Model](03-Data-Model.md) for complete schema.

---

## Navigation Tips

- **Looking for specific functionality?** Check the [screens](screens/) or [workflows](workflows/) directories
- **Troubleshooting an issue?** See [Known Issues](09-Known-Issues.md) for bugs and workarounds
- **Need to know if something works?** See [Current Status](02-Current-Status.md)
- **Planning a deployment?** Use the [Implementation Guide](UP%20Clearview%20-%20Implementation%20Guide.md)
- **Can't find what you need?** Search all docs for keywords or check [RESTRUCTURING_COMPLETE.md](RESTRUCTURING_COMPLETE.md) for what changed

---

**Last Updated:** October 24, 2025
**Questions or Issues?** See [Known Issues](09-Known-Issues.md) or contact the development team
