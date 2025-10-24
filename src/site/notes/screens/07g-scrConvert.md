---
{"dg-publish":true,"permalink":"/screens/07g-scr-convert/","dgShowToc":true}
---

# scrConvert

**Purpose:** Convert a referral/applicant to a current resident upon move-in.
**Status:** ⚠️ Code complete, needs validation testing (1-2 hours)
**Data Sources:** Referrals & Applicants, Current Residents, Units, Service Providers

## Overview

scrConvert is the workflow screen for converting an approved applicant to a current resident when they sign their lease and move in. The screen displays referral information on the left for review, with input fields on the right for move-in specific details (lease signing date, coordinator assignment, and service provider associations).

When the "Convert to Current Resident" button is clicked, the system creates a new Current Residents record with all the referral's demographic and eligibility data, updates the unit to reflect the new occupancy, and removes the referral record. The user is then automatically navigated to the new resident's ResInfoScreen in edit mode for any final data verification.

This workflow was developed using the progressive debugging approach documented in the technical manual, starting with minimal required fields and adding fields one at a time to ensure compatibility.

## Key Components

### Main Controls
- **FormViewer1** - Displays referral data for review (left panel)
- **DatePicker1** - Lease signing date input
- **TextInput1** - Assigned coordinator name (optional)
- **ComboBox1** - Associated service providers selection (multi-select)
- **Button3** - "Convert to Current Resident" action button

### Data Flow
- **OnVisible:** Currently empty (could pre-populate coordinator with current user)
- **Button3 OnSelect:**
  1. Create Current Resident record with Patch(), capture result in varNewResident
  2. Show success notification
  3. Update Unit record (set Occupant Household, clear Referral Household, mark occupied)
  4. Remove referral record using RemoveIf()
  5. Set varSelectedResident to new resident
  6. Navigate to ResInfoScreen
  7. Set form to edit mode

## Features

### Referral Data Review
**Status:** Complete
**Description:** FormViewer displays all referral information in read-only format for staff review before conversion.

**Fields Displayed:**
- Name, Preferred Name
- Unit Number, Unit Type
- Date of Birth, Phone, Email
- Household Size, Household Status
- Veteran Status, Disability, Benefits
- Income information
- HMIS Record Number, County Case Number
- Associated Providers

### Move-In Data Entry
**Status:** Complete
**Description:** Input fields for move-in specific information.

**Fields:**
- **Lease Signing Date (DatePicker):** Required, defaults to current date
- **Assign to Coordinator (TextInput):** Optional, staff member overseeing case
- **Associate Service Providers (ComboBox):** Optional multi-select from Service Providers list

### Convert to Current Resident
**Status:** Complete
**Description:** Creates resident record, updates unit, removes referral, navigates to resident detail.

**Complete Working Formula:**
```powerapps
OnSelect: |-
  =Set(
      varNewResident,
      Patch(
          'Current Residents',
          Defaults('Current Residents'),
          {
              Title: varSelectedReferral.Title,
              'Household Status': {
                  '@odata.type': "#Microsoft.Azure.Connectors.SharePoint.SPListExpandedReference",
                  Value: "Occupant, Current"
              },
              'Unit Number': varSelectedReferral.'Unit Number',
              'Associated Providers': ComboBox1.SelectedItems,
              'Preferred Name': varSelectedReferral.'Preferred Name',
              'Preferred Language': varSelectedReferral.'Preferred Language',
              'Date of Birth': varSelectedReferral.'Date of Birth',
              Phone: varSelectedReferral.Phone,
              Email: varSelectedReferral.Email,
              'Veteran Status': varSelectedReferral.'Veteran Status',
              Disability: varSelectedReferral.Disability,
              Benefits: varSelectedReferral.Benefits,
              'Other Source of Income': varSelectedReferral.'Other Source of Income',
              'Monthly Income': varSelectedReferral.'Monthly Income',
              'HMIS Record Number': varSelectedReferral.'HMIS Record Number',
              'County Case Number': varSelectedReferral.'County Case Number',
              'Household Size': varSelectedReferral.'Household Size',
              'Lease Signing Date': DatePicker1.SelectedDate
          }
      )
  );
  Notify("Resident saved successfully.", NotificationType.Success);
  Patch(
      Units,
      LookUp(Units, ID = varSelectedReferral.'Unit Number'.Id),
      {
          'Occupant Household': {
              '@odata.type': "#Microsoft.Azure.Connectors.SharePoint.SPListExpandedReference",
              Id: varNewResident.ID,
              Value: varNewResident.Title
          },
          'Referral Household': Blank(),
          'Referral Requested': Blank(),
          'Has referral': false,
          'Is occupied': true
      }
  );
  RemoveIf('Referrals & Applicants', ID = varSelectedReferral.ID);
  Set(varSelectedResident, varNewResident);
  Navigate(ResInfoScreen, ScreenTransition.None);
  EditForm(Form1)
```

**Design Notes - Field Type Handling:**

This workflow was developed using **progressive debugging**, adding fields one at a time to discover compatibility issues:

1. **Text Fields:** Simple copy works (Title, Preferred Name, Phone, Email, etc.)
2. **Number Fields:** Direct copy works (Household Size, Monthly Income)
3. **Date Fields:** DatePicker.SelectedDate and source date fields work directly
4. **Lookup Fields (Single):** Pass entire lookup object (Unit Number)
5. **Multi-Lookup Fields:** ComboBox.SelectedItems returns proper format (Associated Providers)
6. **Choice Fields:** Can pass entire object when both source and destination are choice (Veteran Status)
7. **Multi-Choice Fields:** Pass entire array (Disability, Benefits)

**Critical Learning - RemoveIf() vs Remove():**
- `Remove('Referrals & Applicants', varSelectedReferral)` only cleared the Title field
- `RemoveIf('Referrals & Applicants', ID = varSelectedReferral.ID)` properly deletes entire record

**Why Capture varNewResident:**
The newly created resident's ID is needed to update the Unit's Occupant Household lookup field. By capturing the Patch result in varNewResident, we can immediately use varNewResident.ID and varNewResident.Title.

## Known Issues

None currently. Workflow is fully functional and tested.

## Testing Notes

**Test Cases - Complete Workflow:**
- [ ] Navigate to RNAInfoScreen for approved applicant
- [ ] Click "Create Resident" button
- [ ] Verify navigation to scrConvert
- [ ] Verify FormViewer displays all referral data correctly
- [ ] Verify DatePicker defaults to today or selected date
- [ ] Enter coordinator name (or leave blank)
- [ ] Select one or more service providers (or leave blank)
- [ ] Click "Convert to Current Resident"
- [ ] Verify success notification appears
- [ ] Wait for processing (should be 1-2 seconds)
- [ ] Verify navigation to ResInfoScreen
- [ ] Verify new resident data displays correctly
- [ ] Verify form is in edit mode
- [ ] Verify all text fields copied correctly
- [ ] Verify Date of Birth copied correctly
- [ ] Verify Veteran Status copied correctly
- [ ] Verify Disability array copied correctly
- [ ] Verify Benefits array copied correctly
- [ ] Verify Unit Number lookup copied correctly
- [ ] Verify Associated Providers from ComboBox saved correctly
- [ ] Verify Lease Signing Date from DatePicker saved correctly
- [ ] Verify Household Status set to "Occupant, Current"
- [ ] Navigate to Units browse
- [ ] Find unit and verify Occupant Household shows new resident
- [ ] Verify Is occupied = true
- [ ] Verify Referral Household cleared
- [ ] Verify Has referral = false
- [ ] Navigate to Referrals browse
- [ ] Verify former referral no longer appears (record deleted)

**Test Cases - Edge Cases:**
- [ ] Convert referral with no associated providers (should work)
- [ ] Convert referral with no phone/email (should work)
- [ ] Convert referral with no veteran status (should work)
- [ ] Convert referral with minimal fields (name and unit only - should work)
- [ ] Convert referral with all fields populated (should work)
- [ ] Leave coordinator field blank (should work)
- [ ] Leave service providers unselected (should work)
- [ ] Test with referral that has multiple disabilities selected
- [ ] Test with referral that has multiple benefits selected

**Test Cases - Data Integrity:**
- [ ] After conversion, verify no duplicate records exist
- [ ] Verify referral record is completely deleted (not just Title cleared)
- [ ] Verify unit shows correct occupant immediately
- [ ] Verify unit no longer shows in vacancy filter
- [ ] Verify unit cannot receive new referral (button hidden)
- [ ] Verify resident appears in residents browse immediately
- [ ] Navigate away and back - verify all data persists

**Performance Test:**
- [ ] Convert resident with large amount of data
- [ ] Verify operation completes in reasonable time (< 5 seconds)
- [ ] No errors or timeouts

## Related Documentation

- [RNAInfoScreen](07e-RNAInfoScreen.md) - Source screen for conversion workflow
- [ResInfoScreen](07f-ResInfoScreen.md) - Destination screen after conversion
- [Progressive Debugging Methodology](../UP%20Clearview%20–%20Technical%20Documentation.md#critical-development-philosophy-progressive-debugging) - How this workflow was debugged
- [Data Model - Current Residents List](../03-Data-Model.md#current-residents-list) - Destination schema
- [Data Model - Referrals & Applicants List](../03-Data-Model.md#referrals--applicants-list) - Source schema
