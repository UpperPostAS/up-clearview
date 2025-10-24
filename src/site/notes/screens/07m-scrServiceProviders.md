---
{"dg-publish":true,"permalink":"/screens/07m-scr-service-providers/","dgShowToc":true}
---

# scrServiceProviders

**Purpose:** Browse and manage service providers directory (case managers, therapists, medical providers, etc.).
**Status:** Complete (with permission limitation)
**Data Sources:** Service Providers, Agencies

## Overview

scrServiceProviders maintains a comprehensive directory of individual service providers who work with Upper Post Clearview residents. This includes case managers, therapists, mental health professionals, medical providers, and other support staff from various partner organizations.

The screen follows the gallery-and-form pattern: a gallery on the left displays all providers, and selecting a provider loads their details in a form on the right for viewing or editing. This centralized directory makes it easy to find provider contact information, track which organization they work for, and associate them with residents.

The screen is fully functional for browsing and editing existing providers, but shares the same permission issue as OrgScreen, preventing creation of new provider records through the app interface.

## Key Components

### Main Controls
- **ProvidersGallery** - Left panel: List of all service providers
- **ProviderForm** - Right panel: Details form for selected provider
- **Header Section** - Screen title and navigation

### Data Flow
- **Gallery.Items:** Displays all records from Service Providers list
- **Gallery.OnSelect:** Sets selected provider and loads into form
- **Form.Item:** Bound to selected provider record
- **Form.DefaultMode:** View or Edit depending on user action
- **Form.OnSuccess:** Saves changes and refreshes gallery

## Features

### Service Providers Gallery
**Status:** Complete
**Description:** Displays all service providers in a scrollable list.

**Gallery Items:**
```powerapps
Items: ='Service Providers'
```

**Information Displayed:**
- Provider name (Title)
- Organization/Agency (lookup)
- Role/Specialty (case manager, therapist, etc.)

**User Actions:**
- Click provider to view details
- Search/filter functionality (if implemented)
- Sort by name or organization (if implemented)

### Provider Details Form
**Status:** Complete
**Description:** Form for viewing and editing individual provider information.

**Form Fields:**
- **Title** - Provider name (text, required)
- **Organization** - Linked agency (lookup to Agencies list)
- **Role/Specialty** - Provider type (choice: Case Manager, Therapist, Medical Provider, Psychiatrist, etc.)
- **Phone** - Contact phone number (text)
- **Email** - Contact email address (text)
- **Extension** - Phone extension (text)
- **Office Location** - Building/office number (text)
- **Availability** - Scheduling notes or hours (multi-line text)
- **Languages** - Languages spoken (multi-line text or multi-choice)
- **Notes** - Additional information (multi-line text)

**Form Modes:**
- View Mode: Display information (read-only)
- Edit Mode: Modify existing provider details
- New Mode: Create new provider (BLOCKED by permission issue)

### Edit Provider
**Status:** Complete
**Description:** Users can edit existing provider records to update contact information, role, or organizational affiliation.

**OnSuccess Logic:**
```powerapps
OnSuccess: |-
  =Notify("Provider information updated successfully.", NotificationType.Success);
  Refresh('Service Providers')
```

### Link Providers to Residents
**Status:** Complete (via ResInfoScreen and RNAInfoScreen forms)
**Description:** Providers can be associated with residents and referrals through the "Associated Providers" multi-lookup field.

**Where Associations Are Made:**
- ResInfoScreen Form1 - Edit resident's associated providers
- RNAInfoScreen Form1 - Edit referral's associated providers
- scrConvert ComboBox1 - Select providers when converting referral to resident

**Association Benefits:**
- Residents can be linked to multiple providers
- Providers can be linked to multiple residents
- Enables reporting on caseloads and provider engagement
- Displays in scrResidentAssociations for quick reference

## Known Issues

### Cannot Create New Providers Through App
**Severity:** Medium (Feature Limitation)
**Impact:** New service providers must be created directly in SharePoint
**Symptoms:** "New" button doesn't work or is hidden, new record creation fails
**Cause:** SharePoint list permissions or Power Apps connection configuration issue (same as OrgScreen)
**Workaround:** Create new provider records in SharePoint list, then edit in app
**Investigation Needed:** Same troubleshooting steps as OrgScreen

**Troubleshooting Steps:**
1. Navigate to Service Providers SharePoint list
2. Check list permissions settings
3. Verify Power Apps connection has Contribute permission or higher
4. Test manual creation in SharePoint to confirm list allows new items
5. Check Power Apps data source connection permissions
6. Review app sharing settings
7. Compare with OrgScreen issue (likely same root cause)

**Note:** Fixing this issue for Service Providers and Organizations screens simultaneously may be more efficient, as they likely share the same permission configuration problem.

**Estimated Effort:** 2-4 hours (troubleshooting and resolution, combined with OrgScreen fix)

## Testing Notes

**Test Cases - Browse and View:**
- [ ] Navigate to scrServiceProviders screen
- [ ] Verify gallery displays all service providers
- [ ] Verify provider names and organizations are readable
- [ ] Click provider in gallery
- [ ] Verify form loads with provider details
- [ ] Verify all fields display correctly
- [ ] Verify Organization lookup displays correctly
- [ ] Test scrolling through long provider list

**Test Cases - Edit Functionality:**
- [ ] Select a provider
- [ ] Switch form to Edit mode
- [ ] Modify provider name
- [ ] Update phone number
- [ ] Change role/specialty
- [ ] Update organization affiliation
- [ ] Click Save
- [ ] Verify success notification
- [ ] Verify changes persist
- [ ] Verify gallery refreshes with updated information

**Test Cases - Organization Lookup:**
- [ ] Edit provider
- [ ] Change organization via lookup field
- [ ] Verify lookup search works
- [ ] Verify selected organization saves correctly
- [ ] Test provider with no organization (blank lookup)

**Test Cases - Provider Associations:**
- [ ] Navigate to ResInfoScreen
- [ ] Edit resident's associated providers
- [ ] Add provider from scrServiceProviders to resident
- [ ] Save resident record
- [ ] Navigate to scrResidentAssociations
- [ ] Verify provider appears in center column
- [ ] Verify provider details display correctly

**Test Cases - New Provider (after permission fix):**
- [ ] Click "New" or "Add Provider" button
- [ ] Verify blank form appears
- [ ] Enter provider name
- [ ] Select organization
- [ ] Select role/specialty
- [ ] Enter contact information
- [ ] Click Save
- [ ] Verify new provider appears in gallery
- [ ] Verify new provider can be associated with residents

**Test Cases - Search/Filter (if implemented):**
- [ ] Search for provider by name
- [ ] Filter providers by organization
- [ ] Filter providers by role/specialty
- [ ] Verify filtered results are accurate

## Development Roadmap

**Immediate (Bug Fix):**
- [ ] Investigate and resolve new provider creation permission issue (2-4 hours)
- [ ] Coordinate fix with OrgScreen (same issue)
- [ ] Test fix with multiple user roles
- [ ] Document resolution for future reference

**Post-Deployment Enhancements:**
- [ ] Add search/filter functionality to gallery
- [ ] Add sorting options (alphabetical, by organization, by role)
- [ ] Add provider type icons for visual identification
- [ ] Show provider caseload (count of associated residents)
- [ ] Add inactive/archived provider status
- [ ] Link to provider's residents (show all residents associated with provider)
- [ ] Add contact history tracking (log interactions with providers)
- [ ] Add provider availability calendar integration

## Related Documentation

- [srcOrganizations](07l-OrgScreen.md) - Manage organizations (agencies)
- [scrResidentAssociations](07j-scrResidentAssociations.md) - View resident's associated providers
- [ResInfoScreen](07f-ResInfoScreen.md) - Edit resident's associated providers
- [RNAInfoScreen](07e-RNAInfoScreen.md) - Edit referral's associated providers
- [Data Model - Service Providers List](../03-Data-Model.md#service-providers-list) - Schema for providers
- [Data Model - Agencies List](../03-Data-Model.md#agencies-list) - Schema for organizations
- [Known Issues - Permission Problems](../09-Known-Issues.md#provider-creation-permissions) - Permission troubleshooting
