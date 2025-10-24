---
{"dg-publish":true,"permalink":"/workflows/08d-convert-to-resident/","dgShowToc":true}
---

# Convert to Resident

**Purpose:** Convert an approved applicant to a current resident upon move-in and lease signing
**Status:** ⚠️ Code complete, needs testing
**Trigger:** User clicks "Create Resident" button on RNAInfoScreen
**Data Sources:** Referrals & Applicants (deletes), Current Residents (creates), Units (updates)

## Overview

The Convert to Resident workflow marks the successful conclusion of the referral-to-housing journey. When an applicant has completed intake, been approved for housing, and is ready to move in, staff use this workflow to transition them from applicant status to current resident status.

This is a complex multi-step operation that involves creating a new resident record (copying all demographic and eligibility data from the referral), updating the unit record to reflect the new occupant, and removing the referral record. The workflow ensures data integrity across three SharePoint lists and maintains the relationship between residents and their assigned units.

The screen provides a review panel showing all referral data, allowing staff to verify information before conversion. Staff then enter the lease signing date, optionally assign a housing coordinator, and select associated service providers. The conversion is executed as a single atomic operation to prevent partial data states.

This workflow was completed using progressive debugging techniques, adding fields one at a time to identify and resolve field type mismatches between the Referrals & Applicants and Current Residents lists.

## User Journey

1. **Access Screen:** User views an approved applicant on RNAInfoScreen (status: "Applicant, Approved")
2. **Initiate Conversion:** User clicks "Create Resident" workflow button
3. **Navigate to Screen:** System navigates to scrConvert
4. **Review Data:** User reviews referral data displayed in left FormViewer panel
5. **Enter Lease Date:** User selects lease signing date from DatePicker
6. **Assign Coordinator:** (Optional) User enters housing coordinator name in TextInput
7. **Select Providers:** (Optional) User selects associated service providers from multi-select ComboBox
8. **Convert:** User clicks "Convert to Current Resident" button
9. **Process:** System executes multi-step operation:
   - Creates Current Resident record with all referral data
   - Shows success notification
   - Updates Unit record (sets occupant, clears referral, marks occupied)
   - Deletes Referral record
   - Sets varSelectedResident for navigation
10. **Navigate:** System navigates to ResInfoScreen showing new resident
11. **Edit Mode:** Form opens in edit mode for immediate verification/updates

## Technical Implementation

### Screen(s) Involved
- **RNAInfoScreen** - Starting point with "Create Resident" button (Button3)
- **scrConvert** - Conversion screen with review and data entry
- **ResInfoScreen** - Destination screen for newly created resident

### Data Operations

**Creates:**
- New record in Current Residents list with:
  - All demographic fields from referral (name, DOB, contact info)
  - All eligibility fields from referral (veteran status, disability, income, benefits)
  - Unit assignment from referral
  - Case management IDs from referral (HMIS number, county case number)
  - Household Status set to "Occupant, Current"
  - Lease Signing Date from user input
  - Associated Providers from user selection

**Updates:**
- Units record with:
  - Occupant Household (lookup to newly created resident)
  - Referral Household cleared (set to Blank)
  - Referral Requested date cleared (set to Blank)
  - Has referral set to false
  - Is occupied set to true

**Deletes:**
- Referrals & Applicants record (original referral/applicant removed completely)

### Key Formulas

**Complete Convert Button OnSelect Formula:**
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

**Critical Field Type Handling Notes:**
- **Lookup fields** (Unit Number, Associated Providers): Pass entire object, SharePoint handles proper format
- **Choice fields** (Veteran Status): Can pass object OR .Value depending on destination field type
- **Multi-choice fields** (Disability, Benefits): Pass entire array
- **Text fields**: Simple copy works
- **Date fields**: DatePicker.SelectedDate works directly
- **ComboBox multi-select**: ComboBox.SelectedItems returns correct array format

**Why RemoveIf() instead of Remove():**
```powerapps
// DON'T USE THIS - only clears Title field
Remove('Referrals & Applicants', varSelectedReferral)

// USE THIS - properly deletes entire record
RemoveIf('Referrals & Applicants', ID = varSelectedReferral.ID)
```

## Testing Procedure

### Prerequisites:
- Approved applicant record with status "Applicant, Approved"
- Referral assigned to a valid unit
- Referral has complete demographic information
- User has permissions to create residents and update units

### Basic Testing:

1. **Setup:**
   - Create test referral with all fields populated
   - Update status to "Applicant, Approved"
   - Assign to test unit
   - Navigate to RNAInfoScreen for test referral

2. **Execute:**
   - Verify "Create Resident" button visible
   - Click "Create Resident" button
   - Verify navigation to scrConvert
   - Verify FormViewer shows all referral data
   - Select lease signing date (e.g., today)
   - Enter coordinator name (optional)
   - Select 1-2 service providers (optional)
   - Click "Convert to Current Resident"

3. **Verify:**
   - Success notification appears ("Resident saved successfully")
   - Navigation to ResInfoScreen occurs
   - Form opens in edit mode
   - All demographic data copied correctly (name, DOB, phone, email)
   - All eligibility data copied correctly (veteran status, disability, benefits, income)
   - Unit assignment maintained
   - HMIS and County Case numbers transferred
   - Lease signing date appears on resident record
   - Coordinator name saved (if entered)
   - Service providers associated (if selected)
   - Household Status shows "Occupant, Current"

### Related Records Verification:

4. **Verify Unit Updated:**
   - Navigate to UnitInfoScreen for the unit
   - Verify "Occupant Household" shows new resident name
   - Verify "Referral Household" is blank/empty
   - Verify "Has referral" shows No/False
   - Verify "Is occupied" shows Yes/True

5. **Verify Referral Deleted:**
   - Navigate to RNABrowseScreen
   - Verify former referral does NOT appear in list
   - Search for referral in SharePoint list directly
   - Confirm record completely deleted (not just Title cleared)

6. **Verify Resident Created:**
   - Navigate to ResBrowseScreen
   - Verify new resident appears in list
   - Verify resident shows correct unit assignment
   - Click resident to view details
   - Verify all fields populated correctly

### Edge Case Testing:

7. **Test Minimal Data:**
   - Create referral with only required fields (name, unit)
   - No phone, email, or optional demographics
   - Execute conversion
   - Verify successful creation with blank optional fields

8. **Test Maximum Data:**
   - Create referral with ALL fields populated
   - Multiple disabilities checked
   - Multiple benefits checked
   - Multiple service providers
   - Execute conversion
   - Verify all data transferred correctly

9. **Test No Associated Providers:**
   - Execute conversion without selecting any service providers
   - Verify conversion completes successfully
   - Verify Associated Providers field on resident is empty/blank

10. **Test Coordinator Field:**
    - Test with coordinator name entered
    - Test with coordinator field left blank
    - Verify both scenarios work correctly

## Known Issues

**Status: Recently completed - needs production testing**

The workflow code is complete and functional, but requires comprehensive production testing with real data before marking as fully Complete.

**Potential Future Enhancements:**
- Add validation to ensure lease signing date is not in future
- Add confirmation dialog before conversion ("Are you sure?")
- Pre-populate coordinator field with current user name
- Pre-populate service providers from referral's existing associated providers
- Add ability to undo conversion (within certain timeframe)
- Log conversion action to audit trail

## Related Documentation

- [scrConvert Screen Documentation](../screens/scrConvert.md)
- [RNAInfoScreen Documentation](../screens/RNAInfoScreen.md)
- [ResInfoScreen Documentation](../screens/ResInfoScreen.md)
- [Intake Workflow](./08c-Intake.md) - Previous step
- [Exit Resident Workflow](./08e-Exit-Resident.md) - End of residency process
- [Data Model - Referrals & Applicants](../03-Data-Model.md)
- [Data Model - Current Residents](../03-Data-Model.md)
- [Data Model - Units](../03-Data-Model.md)
- [Progressive Debugging Guide](../UP%20Clearview%20–%20Technical%20Documentation.md#critical-development-philosophy-progressive-debugging) - Development approach used
