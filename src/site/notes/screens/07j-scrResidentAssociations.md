---
{"dg-publish":true,"permalink":"/screens/07j-scr-resident-associations/","dgShowToc":true}
---

# scrResidentAssociations

**Purpose:** Three-column view showing resident's bill pay accounts, service providers, and household members.
**Status:** Partial (65% complete)
**Data Sources:** Current Residents, Account Information, Service Providers

## Overview

scrResidentAssociations provides a comprehensive view of all entities associated with a resident across three key categories: bill pay accounts (for Housing Support residents), service providers (case managers, therapists, etc.), and household members (family composition).

The screen uses a three-column layout for parallel viewing of related information. The left column displays linked bill pay accounts, the center column shows associated service providers, and the right column is intended for household members but is not yet implemented.

This screen serves as a hub for managing relationships between residents and the various accounts and providers involved in their housing stability. Users can navigate to detail screens for adding or editing accounts and provider associations.

## Key Components

### Main Controls
- **Header Section** - Displays resident name and unit
- **Gallery_Accounts** - Left column: Displays resident's bill pay accounts
- **Gallery_Providers** - Center column: Displays associated service providers
- **Gallery_HouseholdMembers** - Right column: PLACEHOLDER (not built)
- **Navigation Buttons** - Links to add/edit accounts and providers

### Data Flow
- **OnVisible:** Ensures `varSelectedResident` is set for context
- **Gallery_Accounts.Items:** Filters Account Information list by selected resident
- **Gallery_Providers.Items:** Displays associated providers from resident's multi-lookup field
- **Gallery_HouseholdMembers:** Not yet implemented

## Features

### Bill Pay Accounts Display
**Status:** Complete
**Description:** Shows all bill pay accounts linked to the selected resident, including payee, account type, and payment details.

**Gallery Items Filter:**
```powerapps
Items: =Filter('Account Information', 'Linked Resident'.Id = varSelectedResident.ID)
```

**Information Displayed:**
- Account provider/payee name
- Account type
- Username/Account number
- Payment frequency
- Monthly cost
- Payment notes

**User Actions:**
- Click account to view details
- Navigate to scrResidentAccounts to add new account
- Navigate to scrResidentAccounts to edit existing account

### Service Providers Display
**Status:** Complete
**Description:** Shows all service providers associated with the resident (case managers, therapists, medical providers, etc.).

**Gallery Items Source:**
```powerapps
Items: =varSelectedResident.'Associated Providers'
```

**Information Displayed:**
- Provider name
- Organization/Agency
- Role (case manager, therapist, etc.)
- Contact information

**Design Notes:**
- Associated Providers is a multi-lookup field on Current Residents
- Gallery displays the array of provider records
- Can be managed through ResInfoScreen form or potentially inline

### Household Members Display
**Status:** Not Built
**Description:** Intended to show all household members (family composition) for the resident.

**What Needs to Be Built:**

**Option 1: New SharePoint List**
Create a "Household Members" list with fields:
- Title (member name)
- Linked Resident (lookup to Current Residents)
- Relationship (choice: Spouse, Child, Parent, Sibling, Other)
- Date of Birth (date)
- Notes (multi-line text)

**Option 2: Use Existing List**
Investigate if household composition is already tracked in another list or field.

**Gallery Configuration Needed:**
```powerapps
Gallery_HouseholdMembers.Items: =Filter('Household Members', 'Linked Resident'.Id = varSelectedResident.ID)
```

**Information to Display:**
- Member name
- Relationship to head of household
- Age or date of birth
- Any special notes

**User Actions Needed:**
- Click member to view/edit details
- Add new household member button
- Remove household member button

**Estimated Effort:** 4-6 hours
- 1 hour: Create or configure Household Members list
- 2 hours: Build gallery and form for household members
- 1 hour: Implement add/edit/delete functionality
- 1 hour: Testing

## Known Issues

### Household Members Section Missing
**Severity:** Medium (Feature Gap)
**Impact:** Cannot track family composition within the app
**Symptoms:** Right column is empty placeholder
**Cause:** Not yet built - requires either new list or integration with existing data
**Workaround:** Track household composition in notes field or external spreadsheet
**Fix Required:** Build complete household members feature
**Priority:** HIGH - Important for understanding household dynamics and eligibility

### No Inline Add/Edit/Delete
**Severity:** Low (User Experience)
**Impact:** Must navigate to separate screens to manage accounts/providers
**Symptoms:** No quick add buttons on association screen
**Workaround:** Navigate to detail screens
**Enhancement:** Add modal forms or inline editing capabilities
**Priority:** LOW - Current navigation pattern is acceptable

## Testing Notes

**Current Features Testing:**
- [ ] Navigate to ResInfoScreen
- [ ] Click Associations button
- [ ] Verify navigation to scrResidentAssociations
- [ ] Verify resident name displays in header
- [ ] Verify bill pay accounts display in left column
- [ ] Verify accounts filtered to selected resident only
- [ ] Verify associated providers display in center column
- [ ] Verify provider details (name, org, role) are readable
- [ ] Click account item and verify navigation/action
- [ ] Click provider item and verify navigation/action
- [ ] Test with resident who has no accounts (should show empty)
- [ ] Test with resident who has no providers (should show empty)
- [ ] Test with resident who has multiple accounts
- [ ] Test with resident who has multiple providers

**Future Testing (After Household Members Built):**
- [ ] Verify household members display in right column
- [ ] Verify members filtered to selected resident only
- [ ] Add new household member
- [ ] Verify new member appears in gallery
- [ ] Edit household member details
- [ ] Verify changes persist
- [ ] Remove household member
- [ ] Verify member removed from gallery
- [ ] Test with single-person household (no additional members)
- [ ] Test with multi-person household (spouse + children)

## Development Roadmap

**Phase 1: Household Members List (1 hour)**
- [ ] Create Household Members SharePoint list (or identify existing list)
- [ ] Add required fields (name, relationship, DOB, linked resident)
- [ ] Connect list to Power App

**Phase 2: Gallery Implementation (2 hours)**
- [ ] Configure Gallery_HouseholdMembers data source
- [ ] Design gallery template layout
- [ ] Add fields to display (name, relationship, age)
- [ ] Test gallery display with sample data

**Phase 3: Add/Edit Functionality (2 hours)**
- [ ] Create form for adding new household member
- [ ] Create form for editing existing household member
- [ ] Implement save logic
- [ ] Add navigation between association screen and member forms

**Phase 4: Delete Functionality (1 hour)**
- [ ] Add remove button or icon to gallery items
- [ ] Implement confirmation dialog
- [ ] Implement delete logic
- [ ] Test deletion and gallery refresh

**Priority:** HIGH - Schedule for completion within next sprint.

## Related Documentation

- [ResInfoScreen](07f-ResInfoScreen.md) - Parent screen for resident details
- [scrResidentAccounts](07k-scrResidentAccounts.md) - Form for managing bill pay accounts
- [scrServiceProviders](07m-scrServiceProviders.md) - Directory of service providers
- [Data Model - Account Information List](../03-Data-Model.md#account-information-list) - Schema for accounts
- [Data Model - Service Providers List](../03-Data-Model.md#service-providers-list) - Schema for providers
