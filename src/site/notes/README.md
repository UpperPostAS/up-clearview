---
{"dg-publish":true,"permalink":"/readme/","tags":["gardenEntry"]}
---

# UP Clearview Documentation

**UP Clearview** is CommonBond Communities' operations hub for supportive housing. The app centralizes unit availability, referral/resident pipelines, service provider coordination, and Housing Support bill-pay—providing a single source of truth so staff can keep people housed without jumping between spreadsheets.

**Current Status:** ~85% complete | Core workflows operational | Ready for small-site pilot testing

---

## Quick Start

**New to UP Clearview?**
- Start with [Introduction](01-Introduction.md) for overview and key concepts
- See [Current Status](02-Current-Status.md) for what's working and what's next

**Using the App?**
- [User Guide](11-User-Guide.md) - Day-in-the-life walkthrough for staff
- [Data Model](03-Data-Model.md) - Understanding the SharePoint lists

**Deploying to a New Site?**
- [Implementation Guide](21-Implementation-Guide.md) - Complete deployment playbook
- [Testing Checklist](22-Testing-Checklist.md) - Validation procedures

**Building or Troubleshooting?**
- [Architecture](04-Architecture.md) - How the app is structured
- [Progressive Debugging](05-Progressive-Debugging.md) - Troubleshooting methodology
- [Known Issues](09-Known-Issues.md) - Current bugs and workarounds

---

## Documentation Map

### Getting Started
- **[01 - Introduction](01-Introduction.md)** - What is UP Clearview, who it's for, key capabilities
- **[02 - Current Status](02-Current-Status.md)** - What's done, what's in progress, deployment readiness
- **[03 - Data Model](03-Data-Model.md)** - SharePoint schema, lists, and relationships

### Developer Reference
- **[04 - Architecture](04-Architecture.md)** - Power Apps structure, design patterns, philosophy
- **[05 - Progressive Debugging](05-Progressive-Debugging.md)** - How to troubleshoot when things break
- **[06 - Component Reference](06-Component-Reference.md)** - Controls, formulas, common patterns

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
- **[11 - User Guide](11-User-Guide.md)** - How to use UP Clearview (for staff)
- **[12 - Testing Philosophy](12-Testing-Philosophy.md)** - Progressive debugging approach to testing
- **[13 - Testing Procedures](13-Testing-Procedures.md)** - Screen and workflow test cases
- **[14 - Test Data Setup](14-Test-Data-Setup.md)** - Creating test data in SharePoint

### Implementation & Deployment
- **[21 - Implementation Guide](21-Implementation-Guide.md)** - Deploy UP Clearview to new sites
- **[22 - Testing Checklist](22-Testing-Checklist.md)** - Pre-deployment validation
- **[23 - Field Customization](23-Field-Customization.md)** - Adapting for different site types
- **[24 - Rollout Strategy](24-Rollout-Strategy.md)** - Phased deployment plan

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

- **Looking for specific functionality?** Check the screen or workflow guides
- **Troubleshooting an issue?** Start with [Progressive Debugging](05-Progressive-Debugging.md)
- **Need to know if something works?** See [Current Status](02-Current-Status.md)
- **Planning a deployment?** Follow the [Implementation Guide](21-Implementation-Guide.md)
- **Can't find what you need?** Search all docs for keywords

---

**Last Updated:** October 24, 2025
**Questions or Issues?** See [Known Issues](09-Known-Issues.md) or contact the development team
