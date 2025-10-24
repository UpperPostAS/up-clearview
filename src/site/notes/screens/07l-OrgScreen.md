---
{"dg-publish":true,"permalink":"/screens/07l-org-screen/","dgShowToc":true}
---

# srcOrganizations (OrgScreen)

**Purpose:** Browse and manage agencies/organizations that provide services or referrals.
**Status:** Complete (with permission limitation)
**Data Sources:** Agencies

## Overview

srcOrganizations (also referred to as OrgScreen) provides a directory view of agencies and organizations that Upper Post Clearview works with for referrals, services, and partnerships. This includes county social services, CoC agencies, healthcare organizations, and other community partners.

The screen uses a gallery-and-form layout pattern: a gallery on the left displays all organizations, and selecting an organization loads its details in a form on the right for viewing or editing. This allows staff to maintain current contact information, service descriptions, and organizational details for all partner agencies.

The screen is fully functional for browsing and editing existing organizations, but there is a known permission issue preventing creation of new organization records through the app interface.

## Key Components

### Main Controls
- **OrganizationsGallery** - Left panel: List of all agencies/organizations
- **OrganizationForm** - Right panel: Details form for selected organization
- **Header Section** - Screen title and navigation

### Data Flow
- **Gallery.Items:** Displays all records from Agencies list
- **Gallery.OnSelect:** Sets selected organization and loads into form
- **Form.Item:** Bound to selected organization record
- **Form.DefaultMode:** View or Edit depending on user action
- **Form.OnSuccess:** Saves changes and refreshes gallery

## Features

### Organizations Gallery
**Status:** Complete
**Description:** Displays all agencies and organizations in a scrollable list.

**Gallery Items:**
```powerapps
Items: =Agencies
```

**Information Displayed:**
- Organization name (Title)
- Organization type or primary service
- Contact person (if applicable)

**User Actions:**
- Click organization to view details
- Search/filter functionality (if implemented)
- Sort by name (if implemented)

### Organization Details Form
**Status:** Complete
**Description:** Form for viewing and editing organization information.

**Form Fields:**
- **Title** - Organization name (text, required)
- **Organization Type** - Category (choice: Government, Non-Profit, Healthcare, etc.)
- **Services Provided** - Description of services (multi-line text)
- **Primary Contact Name** - Main contact person (text)
- **Contact Title** - Role/position of contact (text)
- **Phone** - Contact phone number (text)
- **Email** - Contact email address (text)
- **Address** - Physical address (multi-line text)
- **Website** - Organization website URL (text)
- **Notes** - Additional information (multi-line text)

**Form Modes:**
- View Mode: Display information (read-only)
- Edit Mode: Modify existing organization details
- New Mode: Create new organization (BLOCKED by permission issue)

### Edit Organization
**Status:** Complete
**Description:** Users can edit existing organization records to update contact information, services, or other details.

**OnSuccess Logic:**
```powerapps
OnSuccess: |-
  =Notify("Organization information updated successfully.", NotificationType.Success);
  Refresh(Agencies)
```

## Known Issues

### Cannot Create New Organizations Through App
**Severity:** Medium (Feature Limitation)
**Impact:** New organizations must be created directly in SharePoint
**Symptoms:** "New" button doesn't work or is hidden, new record creation fails
**Cause:** SharePoint list permissions or Power Apps connection configuration issue
**Workaround:** Create new organization records in SharePoint list, then edit in app
**Investigation Needed:**
- Check SharePoint list permissions (verify app connection has "Add items" permission)
- Check Power Apps data connection settings
- Review Azure AD app registration (if using service principal)
- Test with different user roles to isolate permission level

**Troubleshooting Steps:**
1. Navigate to Agencies SharePoint list
2. Check list permissions settings
3. Verify Power Apps connection has Contribute permission or higher
4. Test manual creation in SharePoint to confirm list allows new items
5. Check Power Apps data source connection permissions
6. Review app sharing settings (check if shared users have appropriate role)

**Potential Solutions:**
1. Grant Power Apps connection higher permissions on SharePoint list
2. Reconfigure data connection in Power Apps
3. Update Azure AD app registration permissions
4. Share app with higher permission level

**Estimated Effort:** 2-4 hours (troubleshooting and resolution)

## Testing Notes

**Test Cases - Browse and View:**
- [ ] Navigate to srcOrganizations screen
- [ ] Verify gallery displays all organizations
- [ ] Verify organization names are readable
- [ ] Click organization in gallery
- [ ] Verify form loads with organization details
- [ ] Verify all fields display correctly
- [ ] Test scrolling through long organization list
- [ ] Test with no organization selected (form should be empty or show prompt)

**Test Cases - Edit Functionality:**
- [ ] Select an organization
- [ ] Switch form to Edit mode
- [ ] Modify organization name
- [ ] Update contact phone number
- [ ] Change services provided description
- [ ] Click Save
- [ ] Verify success notification
- [ ] Verify changes persist (select different org and come back)
- [ ] Verify gallery refreshes with updated information

**Test Cases - Search/Filter (if implemented):**
- [ ] Use search box to find organization by name
- [ ] Verify filtered results
- [ ] Clear search and verify all organizations return
- [ ] Test partial name matching

**Test Cases - New Organization (after permission fix):**
- [ ] Click "New" or "Add Organization" button
- [ ] Verify blank form appears
- [ ] Enter organization name
- [ ] Fill required fields
- [ ] Click Save
- [ ] Verify new organization appears in gallery
- [ ] Verify new organization can be selected and edited

**Test Cases - Permission Issues:**
- [ ] Attempt to create new organization
- [ ] Document exact error message (if any)
- [ ] Try different user accounts (admin vs standard user)
- [ ] Test direct creation in SharePoint list
- [ ] Verify edit permissions work for existing records

## Development Roadmap

**Immediate (Bug Fix):**
- [ ] Investigate and resolve new organization creation permission issue (2-4 hours)
- [ ] Test fix with multiple user roles
- [ ] Document resolution for future reference

**Post-Deployment Enhancements:**
- [ ] Add search/filter functionality to gallery
- [ ] Add sorting options (alphabetical, by type, etc.)
- [ ] Add organization type icons for visual identification
- [ ] Link organizations to referrals (show which referrals came from each org)
- [ ] Add contact history tracking (log interactions with organizations)
- [ ] Add inactive/archived organization status

## Related Documentation

- [scrServiceProviders](07m-scrServiceProviders.md) - Similar screen for individual service providers
- [Data Model - Agencies List](../03-Data-Model.md#agencies-list) - Schema for organizations
- [Known Issues - Permission Problems](../09-Known-Issues.md#provider-creation-permissions) - Permission troubleshooting
