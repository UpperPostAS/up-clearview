---
{"dg-publish":true,"permalink":"/up-clearview-technical-documentation/"}
---

**Power Apps Canvas Application Development Guide**

---

## About This Manual

This manual documents the Upper Post Clearview Power Apps canvas application for developers and maintainers. Use it alongside the [[UP Clearview - Introduction\|participant journey guide]] for workflow context, the [[UP Clearview – Data Sources\|data source reference]] for SharePoint schemas, and the [[UP Clearview - Development Roadmap\|development roadmap]] for current priorities.

This document focuses on:
- **How** the application is built (architecture, components, patterns)
- **What** works, what's broken, and what's missing
- **How to test** each feature systematically
- **How to troubleshoot** problems using progressive debugging techniques

---

## Critical Development Philosophy: Progressive Debugging

### The Most Important Lesson Learned

**When facing stubbornly persistent problems in Power Apps, use the smallest piece of functionality you can, progressively adding more to discover exactly where the issue is.**

This approach was critical to solving the Convert to Resident workflow, which silently failed without errors. By stripping the Patch operation down to just Title and Household Status (the only required field), we could verify the basic mechanism worked. Then, adding fields one at a time, we discovered:

1. Simple text fields worked fine
2. Lookup fields (Unit Number, Associated Providers) worked when passed as complete objects
3. The Veteran Status field caused silent failures because it was a **choice field object** in the source but a **text field** in the destination

### How to Apply Progressive Debugging

**For Forms That Won't Save:**
1. Start with a single required field
2. Test - does it save?
3. Add one field at a time
4. When it breaks, you've found your problem field

**For Filters That Don't Work:**
1. Test the filter with no conditions
2. Add one condition at a time
3. Check for delegation warnings
4. Test with small data sets first

**For Navigation That Fails:**
1. Navigate with no variables set
2. Add one Set() at a time
3. Check variable values with a Label control showing the variable

**For Lookups That Return Blank:**
1. Test the LookUp formula in a Label first
2. Verify the ID field exists and has a value
3. Check field names match exactly (case-sensitive!)
4. Use With() to inspect intermediate values

### Example: The Convert Workflow Success Story

**Original (broken - silent failure):**
```powerapps
Patch(
    'Current Residents',
    Defaults('Current Residents'),
    {
        Title: varSelectedReferral.Title,
        'Preferred Name': varSelectedReferral.'Preferred Name',
        // ... 15 more fields ...
        'Veteran Status': varSelectedReferral.'Veteran Status',  // ← This broke everything
        'Unit Number': varSelectedReferral.'Unit Number',
        'Associated Providers': ComboBox1.SelectedItems
    }
)
```

**Minimal test (success):**
```powerapps
Patch(
    'Current Residents',
    Defaults('Current Residents'),
    {
        Title: varSelectedReferral.Title,
        'Household Status': {
            '@odata.type': "#Microsoft.Azure.Connectors.SharePoint.SPListExpandedReference",
            Value: "Occupant, Current"
        }
    }
)
```

**Progressive addition revealed:**
- Adding Unit Number (lookup) → ✅ Works
- Adding Associated Providers (multi-lookup) → ✅ Works
- Adding Date of Birth (date) → ✅ Works
- Adding Veteran Status (choice → text mismatch) → ❌ Silent failure

**Solution:**
```powerapps
'Veteran Status': varSelectedReferral.'Veteran Status'.Value  // Extract .Value for text field
```

**This debugging approach saved hours of frustration and applies to nearly every Power Apps problem.**

---

## Application Architecture

### Technology Stack

**Platform:** Microsoft Power Platform
- **Power Apps Canvas App** (responsive desktop/tablet layout)
- **SharePoint Online** data storage (8 lists)
- **Office 365 Connections** for user authentication
- **Power Automate** (future integration planned)

### Application Structure

```
UP Clearview Canvas App
├── Data Sources (SharePoint Lists)
│   ├── Units
│   ├── Unit Types
│   ├── Current Residents
│   ├── Referrals & Applicants
│   ├── Service Providers
│   ├── Agencies
│   ├── Account Information
│   └── Archived Records
│
├── Screens (15 total)
│   ├── Browse Screens (3)
│   │   ├── UnitBrowseScreen
│   │   ├── RNABrowseScreen
│   │   └── ResBrowseScreen
│   ├── Info/Detail Screens (3)
│   │   ├── UnitInfoScreen
│   │   ├── RNAInfoScreen
│   │   └── ResInfoScreen
│   ├── Workflow Screens (2)
│   │   ├── RNACloseScreen
│   │   └── scrConvert
│   ├── Association Screens (3)
│   │   ├── scrResidentAssociations
│   │   ├── scrResidentAccounts
│   │   └── scrServiceProviders
│   ├── Organization Screen (1)
│   │   └── srcOrganizations
│   ├── Development/Incomplete (2)
│   │   ├── IntakeScreen
│   │   └── Screen1 (unused)
│   └── Editor State (1)
│       └── _EditorState.pa.yaml
│
└── Global Variables
    ├── varSelectedResident
    ├── varSelectedReferral
    ├── varActiveUnit / varSelectedUnit
    ├── varResidentUnit
    ├── varReferralUnit
    ├── varFormMode
    ├── colStatusPalette
    └── [workflow-specific variables]
```

### Design Patterns

**Screen Navigation Pattern:**
```
Browse → OnSelect sets variable → Navigate to Info Screen → Display details
```

**Data Lookup Pattern:**
```
OnVisible: Set related lookup variables
OnSelect: Navigate with context
```

**Color Coding Pattern:**
```
OnVisible: ClearCollect(colStatusPalette, ...)
Gallery Fill: LookUp(colStatusPalette, key = ...)
```

**Form Pattern:**
```
Form.Item = varSelected[Entity]
Form.OnSuccess = Set variables → Patch related records → Navigate
```

---

## Power Apps Components & Control Types

### Understanding Power Apps Component Hierarchy

Power Apps canvas apps are built from **controls** arranged on **screens**. Controls can contain other controls (parent-child relationships).

### Core Control Types Used in UP Clearview

#### 1. **Screen** (`screen`)
- Top-level container for all other controls
- Properties: `Fill` (background color), `OnVisible` (runs when screen loads), `LoadingSpinnerColor`
- **Example:** `RNABrowseScreen`, `UnitInfoScreen`

**Common pattern:**
```powerapps
OnVisible: |-
  =Set(varSelectedReferral, varSelectedReferral);
  Set(varReferralUnit, LookUp(Units, ID = varSelectedReferral.'Unit Number'.Id));
```

#### 2. **Gallery** (`Gallery@2.15.0`)
- Repeating container that displays multiple records
- Properties: `Items` (data source), `OnSelect` (what happens when item clicked), `TemplateFill`, `TemplateSize`
- **Variants:** `BrowseLayout_Vertical_ThreeTextVariant_ver5.0`, `galleryTemplate7_1`
- **Key concept:** Inside a gallery, use `ThisItem` to reference the current record

**Example from RNABrowseScreen:**
```powerapps
Items: |-
  =Sort(
      Filter(
          'Referrals & Applicants',
          !StartsWith(Lower(Coalesce('Household Status'.Value, "")), "former")
      ),
      'Unit Number'.Value,
      SortOrder.Ascending
  )
OnSelect: |-
  =Set(varSelectedReferral, ThisItem);
  Set(varReferralUnit, LookUp(Units, ID = varSelectedReferral.'Unit Number'.Id));
  Navigate(RNAInfoScreen, ScreenTransition.None)
```

**Performance Note:** Avoid nested LookUp() calls inside gallery Items if possible. Pre-load data in collections instead.

#### 3. **Form** (`Form@2.4.4`)
- Data entry and display control bound to a SharePoint list
- Properties: `DataSource` (which list), `Item` (which record), `DefaultMode`, `OnSuccess`, `OnFailure`
- **Variants:** `Classic`, `Modern`
- **Contains:** DataCard controls for each field

**Example from RNACloseScreen:**
```powerapps
DataSource: ='Referrals & Applicants'
Item: =varCloseRec
OnSuccess: |-
  =Set(varClosedReferral, EditForm1.LastSubmit);
  Patch('Referrals & Applicants', varClosedReferral, { 'Household Status': { Value: varClosedStatus } });
  Navigate(RNABrowseScreen, ScreenTransition.None)
```

**Form Modes:**
- `FormMode.New` - Create new record
- `FormMode.Edit` - Edit existing record
- `FormMode.View` - Read-only display

**Critical Functions:**
- `SubmitForm(FormName)` - Saves changes
- `ResetForm(FormName)` - Clears form
- `EditForm(FormName)` - Switches to edit mode
- `NewForm(FormName)` - Switches to new mode
- `ViewForm(FormName)` - Switches to view mode

#### 4. **DataCard** (TypedDataCard@1.0.7)
- Individual field within a form
- Properties: `DataField` (internal field name), `Default` (initial value), `Update` (value to save), `DisplayName`
- **Variants:** `ClassicTextEdit`, `ClassicDateEdit`, `ClassicComboBoxView`, `ClassicComboBoxMultiSelectView`
- **Contains:** DataCardKey (label), DataCardValue (input), ErrorMessage, StarVisible (required indicator)

**Example:**
```powerapps
DataField: ="Close_x002d_out_x0020_Date"
Default: =ThisItem.'Close-out Date'
Update: =DataCardValue67.SelectedDate
DisplayName: =DataSourceInfo([@'Referrals & Applicants'], DataSourceInfo.DisplayName, 'Close-out Date')
```

**DataCard Children:**
- `DataCardKey` - Label showing field name
- `DataCardValue` - Input control (TextInput, DatePicker, ComboBox, etc.)
- `ErrorMessage` - Validation error display
- `StarVisible` - Asterisk for required fields

#### 5. **FormViewer** (`FormViewer@2.3.4`)
- Display-only version of a form (no editing)
- Properties: `DataSource`, `Item`, `NumberOfColumns`, `SnapToColumns`
- **Use case:** Showing referral data for review before conversion (scrConvert)

#### 6. **Button** (`Classic/Button@2.2.0`)
- Clickable action trigger
- Properties: `Text`, `OnSelect` (formula to run when clicked), `Fill`, `Color`, `DisplayMode`
- **DisplayMode values:** `DisplayMode.Edit` (clickable), `DisplayMode.Disabled`, `DisplayMode.View`

**Example - Convert to Resident button:**
```powerapps
OnSelect: |-
  =Set(varNewResident, Patch('Current Residents', Defaults('Current Residents'), {...}));
  RemoveIf('Referrals & Applicants', ID = varSelectedReferral.ID);
  Navigate(ResInfoScreen, ScreenTransition.None);
```

#### 7. **Label** (`Label@2.5.1`)
- Display text or data
- Properties: `Text` (content), `Color`, `Font`, `Size`, `Align`, `AutoHeight`
- **Common use:** Displaying field values, headings, instructions

#### 8. **TextInput** (`Classic/TextInput@2.3.2`)
- Single-line text entry
- Properties: `Default` (initial value), `HintText` (placeholder), `OnChange`

#### 9. **ComboBox** (`Classic/ComboBox@2.4.0`)
- Dropdown selection from a list
- Properties: `Items` (list of options), `DefaultSelectedItems`, `SelectMultiple` (true for multi-select)
- **DisplayFields:** Which field(s) to show (e.g., `["Value"]` for lookup fields)
- **SearchFields:** Which fields are searchable

**Example - Associated Providers (multi-select):**
```powerapps
Items: =Choices('Referrals & Applicants'.'Associated Providers')
DisplayFields: =["Value"]
SelectMultiple: =true
```

**Important:** `ComboBox.SelectedItems` returns a **table** (array) of selected items, each with `Id` and `Value` properties.

#### 10. **DatePicker** (`Classic/DatePicker@2.6.0`)
- Calendar-based date selection
- Properties: `SelectedDate`, `DefaultDate`, `StartYear`, `EndYear`

#### 11. **Icon** (`Classic/Icon@2.5.0`)
- Clickable icon graphic
- Properties: `Icon` (which icon), `Color`, `OnSelect`
- **Common use:** Back arrows, navigation, action triggers

#### 12. **Rectangle** (`Rectangle@2.3.0`)
- Colored rectangle for visual design
- Properties: `Fill`, `BorderColor`, `Height`, `Width`
- **Common use:** Headers, dividers, selection indicators

#### 13. **GroupContainer** (`GroupContainer@1.3.0`)
- Container for grouping related controls
- **Variant:** `AutoLayout` (automatically arranges children)
- Properties: `LayoutDirection` (Vertical/Horizontal), `LayoutJustifyContent`, `LayoutAlignItems`
- **Common use:** Modal dialogs, organized sections

### Control Naming Conventions

**Current (inconsistent):**
- Forms: `EditForm1`, `Form1`, `Form2`
- Galleries: `RNAGallery`, `UnitBrowseGallery`, `Gallery1`
- Buttons: `Button1`, `Button3`, `IconBackarrow1`
- Containers: `Container4`, `Container6`

**Recommended (for refactoring):**
- Forms: `frmResidentEdit`, `frmReferralClose`, `frmReceiveReferral`
- Galleries: `galReferrals`, `galUnits`, `galResidents`
- Buttons: `btnConvertResident`, `btnCloseReferral`, `btnBack`
- Containers: `grpEligibility`, `grpHeader`

---

## Variable Management

### Global Variables (Set with `Set()`)

Variables persist across screens and hold single values.

#### Screen Context Variables

| Variable | Type | Set On | Purpose | Example Value |
|----------|------|--------|---------|---------------|
| `varSelectedResident` | Record | ResBrowseGallery.OnSelect | Currently selected resident | `{ID: 5, Title: "John Doe", 'Unit Number': {...}}` |
| `varSelectedReferral` | Record | RNAGallery.OnSelect | Currently selected referral | `{ID: 12, Title: "Jane Smith", ...}` |
| `varActiveUnit` | Record | UnitBrowseGallery.OnSelect | Currently selected unit (consider renaming to varSelectedUnit) | `{ID: 8, Unit: "130", ...}` |

#### Lookup Cache Variables

These improve performance by avoiding repeated lookups.

| Variable | Type | Set On | Purpose |
|----------|------|--------|---------|
| `varResidentUnit` | Record | ResInfoScreen.OnVisible | Unit record for selected resident |
| `varReferralUnit` | Record | RNAInfoScreen.OnVisible | Unit record for selected referral |
| `varIntakeUnit` | Record | IntakeScreen.OnVisible | Unit record for intake process |
| `varUnitType` | Record | IntakeScreen.OnVisible, UnitInfoScreen.OnVisible | Unit type details for eligibility |

**Pattern:**
```powerapps
OnVisible: |-
  =Set(varSelectedReferral, varSelectedReferral);  // Ensure variable exists
  Set(varReferralUnit, LookUp(Units, ID = varSelectedReferral.'Unit Number'.Id));
  Set(varUnitType, LookUp('Unit Types', 'Unit Type' = varReferralUnit.'Unit Type'.Value));
```

#### Form Mode Variables

| Variable | Type | Set On | Purpose | Values |
|----------|------|--------|---------|--------|
| `varFormMode` | Text | Button clicks | Controls whether forms display in edit/view mode | `FormMode.Edit`, `FormMode.View`, `FormMode.New` |

#### Workflow Variables

| Variable | Type | Set On | Purpose |
|----------|------|--------|---------|
| `varNewResident` | Record | scrConvert Button3.OnSelect | Newly created resident record (used to get ID for unit update) |
| `varClosedReferral` | Record | RNACloseScreen.OnSuccess | Referral being closed out |
| `varClosedStatus` | Text | RNACloseScreen.OnSuccess | Computed status ("Former Referral" or "Former Applicant") |
| `varCloseRec` | Record | RNAInfoScreen workflow button | Referral record passed to close screen |

### Collections (Set with `ClearCollect()` or `Collect()`)

Collections are in-memory tables that persist across screens.

| Collection | Type | Set On | Purpose | Example |
|------------|------|--------|---------|---------|
| `colStatusPalette` | Table | Screen.OnVisible | Color coding for unit types | `[{key: "section 8 pbv", color: ColorValue("#DDEBF7")}, ...]` |

**Pattern:**
```powerapps
OnVisible: |-
  =ClearCollect(
      colStatusPalette,
      { key: "section 8 pbv",        color: ColorValue("#DDEBF7") },
      { key: "lth housing support",  color: ColorValue("#C6EFCE") },
      { key: "coc rental assistance",color: ColorValue("#FFF2CC") },
      { key: "hud-vash pbv",         color: ColorValue("#E5DFEC") }
  );
```

**Usage in Gallery:**
```powerapps
TemplateFill: =LookUp(colStatusPalette, key = Lower(ThisItem.'Unit Type'.Value), color)
```

### Context Variables (Set with `UpdateContext()` or `Navigate()` parameters)

Context variables are local to a single screen.

**Example:**
```powerapps
Navigate(RNAInfoScreen, ScreenTransition.None, { rec: ThisItem })
```

Then on RNAInfoScreen:
```powerapps
OnVisible: =Set(varSelectedReferral, rec)  // Convert context to global
```

**Note:** UP Clearview primarily uses global variables rather than context variables.

### Variable Debugging Tips

**Display variable contents:**
```powerapps
Label.Text: =varSelectedReferral.Title
Label.Text: =JSON(varSelectedReferral, JSONFormat.IndentFour)  // See entire record structure
```

**Check if variable is blank:**
```powerapps
Label.Text: =If(IsBlank(varSelectedReferral), "No referral selected", varSelectedReferral.Title)
```

**Inspect lookup field structure:**
```powerapps
Label.Text: ="ID: " & varSelectedReferral.'Unit Number'.Id & ", Value: " & varSelectedReferral.'Unit Number'.Value
```

---

## Data Flow & Lookup Field Handling

### SharePoint Field Types in Power Apps

SharePoint stores data with specific field types. Understanding how these appear in Power Apps is critical.

#### Text Fields
**SharePoint:** Single line of text
**Power Apps Type:** String
**Access:** `ThisItem.Phone` or `varSelectedReferral.'Preferred Name'`
**Patch Example:**
```powerapps
{ Phone: varSelectedReferral.Phone }
```

#### Number Fields
**SharePoint:** Number (with or without decimals)
**Power Apps Type:** Number
**Access:** `ThisItem.'Household Size'`
**Patch Example:**
```powerapps
{ 'Household Size': varSelectedReferral.'Household Size' }
```

#### Date Fields
**SharePoint:** Date and Time or Date Only
**Power Apps Type:** Date
**Access:** `ThisItem.'Date of Birth'`
**Patch Example:**
```powerapps
{ 'Date of Birth': varSelectedReferral.'Date of Birth' }
{ 'Lease Signing Date': DatePicker1.SelectedDate }
```

#### Choice Fields (Single-Select)
**SharePoint:** Choice column
**Power Apps Type:** **Object** with `Id` and `Value` properties
**Access:**
- Display: `ThisItem.'Household Status'.Value`
- ID: `ThisItem.'Household Status'.Id`

**Patch Example (if destination is also Choice):**
```powerapps
{
    'Household Status': {
        '@odata.type': "#Microsoft.Azure.Connectors.SharePoint.SPListExpandedReference",
        Value: "Occupant, Current"
    }
}
```

**Patch Example (if destination is Text - type mismatch!):**
```powerapps
{ 'Veteran Status': varSelectedReferral.'Veteran Status'.Value }  // Extract .Value!
```

**Critical Lesson:** This was the root cause of the Convert workflow's silent failure. Referrals & Applicants has Veteran Status as a **Choice** field (object), but Current Residents has it as a **Text** field (string). Passing the object directly caused silent failure. Extracting `.Value` fixed it.

#### Multi-Choice Fields
**SharePoint:** Choice column with "Allow multiple selections"
**Power Apps Type:** **Table** (array) of objects, each with `Id` and `Value`
**Access:**
- Display all: `Concat(ThisItem.Disability, Value, ", ")`
- Check if contains value: `"Physical Disability" in ThisItem.Disability.Value`

**Patch Example:**
```powerapps
{ Disability: varSelectedReferral.Disability }  // Pass entire array
```

#### Lookup Fields (Single)
**SharePoint:** Lookup to another list
**Power Apps Type:** **Object** with `Id` and `Value` properties
**Access:**
- Display: `ThisItem.'Unit Number'.Value`
- Get ID for further lookups: `ThisItem.'Unit Number'.Id`

**Patch Example:**
```powerapps
{ 'Unit Number': varSelectedReferral.'Unit Number' }  // Pass entire lookup object
```

**Alternative (explicit construction):**
```powerapps
{
    'Unit Number': {
        '@odata.type': "#Microsoft.Azure.Connectors.SharePoint.SPListExpandedReference",
        Id: varSelectedReferral.'Unit Number'.Id,
        Value: varSelectedReferral.'Unit Number'.Value
    }
}
```

**For Unit updates with newly created resident:**
```powerapps
{
    'Occupant Household': {
        '@odata.type': "#Microsoft.Azure.Connectors.SharePoint.SPListExpandedReference",
        Id: varNewResident.ID,
        Value: varNewResident.Title
    }
}
```

#### Lookup Fields (Multi-Select)
**SharePoint:** Lookup with "Allow multiple values"
**Power Apps Type:** **Table** (array) of objects, each with `Id` and `Value`
**Access:**
- Display all: `Concat(ThisItem.'Associated Providers', Value, ", ")`
- Count: `CountRows(ThisItem.'Associated Providers')`

**Patch Example:**
```powerapps
{ 'Associated Providers': ComboBox1.SelectedItems }  // ComboBox returns proper array
```

**Getting choices for ComboBox:**
```powerapps
Items: =Choices('Referrals & Applicants'.'Associated Providers')
```

#### Yes/No (Boolean) Fields
**SharePoint:** Yes/No checkbox
**Power Apps Type:** Boolean
**Access:** `ThisItem.'Is occupied'`
**Patch Example:**
```powerapps
{ 'Is occupied': true }
{ 'Has referral': false }
```

### Clearing Lookup Fields

To clear a lookup field (remove the relationship):

```powerapps
{ 'Unit Number': Blank() }
{ 'Referral Household': Blank() }
{ 'Referral Requested': Blank() }  // Date fields use Blank() too
```

### LookUp() Function Patterns

**Basic pattern:**
```powerapps
LookUp(DataSource, Condition, ResultColumn)
```

**Find unit by ID:**
```powerapps
LookUp(Units, ID = varSelectedReferral.'Unit Number'.Id)
```

**Find unit type by value:**
```powerapps
LookUp('Unit Types', 'Unit Type' = varReferralUnit.'Unit Type'.Value)
```

**Find resident by HMIS number:**
```powerapps
LookUp('Current Residents', 'HMIS Record Number' = varSelectedReferral.'HMIS Record Number')
```

**With default value if not found:**
```powerapps
Coalesce(LookUp(Units, ID = ThisItem.'Unit Number'.Id), {Unit: "N/A"})
```

### Patch() Function Patterns

**Create new record:**
```powerapps
Patch(DataSource, Defaults(DataSource), { field1: value1, field2: value2 })
```

**Update existing record:**
```powerapps
Patch(DataSource, LookUpRecord, { field1: newValue1 })
```

**Create and capture result:**
```powerapps
Set(varNewRecord, Patch(DataSource, Defaults(DataSource), {...}))
// varNewRecord.ID now available for related updates
```

**Update multiple related records:**
```powerapps
Set(varNewResident, Patch('Current Residents', Defaults('Current Residents'), {...}));
Patch(Units, LookUp(Units, ID = varReferralUnit.ID), {
    'Occupant Household': {
        '@odata.type': "#Microsoft.Azure.Connectors.SharePoint.SPListExpandedReference",
        Id: varNewResident.ID,
        Value: varNewResident.Title
    }
});
```

### Remove() vs RemoveIf()

**Remove() - removes by record object:**
```powerapps
Remove('Referrals & Applicants', varSelectedReferral)
```
**Issue:** May only clear the Title field in SharePoint instead of deleting the entire record.

**RemoveIf() - removes by condition (RECOMMENDED):**
```powerapps
RemoveIf('Referrals & Applicants', ID = varSelectedReferral.ID)
```
**Result:** Reliably deletes the entire SharePoint list item.

---

## Screen-by-Screen Functionality Review

### 1. UnitBrowseScreen ✅ COMPLETE

**Purpose:** Display all housing units with occupancy status and vacancy filtering.

**Layout:**
- Header: Blue rectangle with "Units" title
- Toggle: "Show Vacancies" checkbox
- Gallery: 3-column grid of unit cards

**Key Components:**
- `UnitBrowseGallery` - Gallery displaying units
- `Toggle1` - Show vacancies filter

**Data Flow:**
```
OnVisible: Load colStatusPalette
Gallery.Items: Filter units (optionally by vacancy)
Gallery.OnSelect: Set varActiveUnit → Navigate to UnitInfoScreen
```

**Formula Details:**
```powerapps
Items: =If(
    Toggle1.Value,  // Show vacancies only
    Filter(
        Units,
        Or(
            'Is occupied' = false,
            'Has given notice' = true
        )
    ),
    Units  // Show all
)
TemplateFill: =LookUp(colStatusPalette, key = Lower(ThisItem.'Unit Type'.Value), color)
```

**Variables Set:**
- `varActiveUnit` - Selected unit record

**Status:** ✅ Complete and functional

**Potential Issues:**
- Gallery uses inline LookUp for unit type colors (acceptable for small datasets <100)
- Variable name `varActiveUnit` inconsistent with naming pattern (should be `varSelectedUnit`)

---

### 2. UnitInfoScreen ✅ COMPLETE

**Purpose:** Display detailed unit information and handle "Receive Referral" workflow.

**Layout:**
- Left panel: Static unit information (FormViewer)
  - Unit details (bedrooms, type, etc.)
  - Administrative info (HMIS ID, subsidy admin)
  - Eligibility requirements (income, homelessness, disability)
- Right panel: Current status (Form)
  - Current occupant or referral information
  - "Receive Referral" button (conditional)
  - Edit mode for status fields

**Key Components:**
- `FormViewer1` - Display static unit details
- `Form1` - Editable unit status
- `Container3` - Modal for "Receive Referral" form
- `Button2_2` - "Receive Referral" button

**Data Flow:**
```
OnVisible: Set varUnitType (lookup unit type for eligibility display)
Button "Receive Referral".OnSelect: Set varShowReceiveForm = true
Form in Container3.OnSuccess: Patch Units, Refresh data, Hide modal
```

**Variables Set:**
- `varUnitType` - Unit type record for eligibility display
- `varShowReceiveForm` - Boolean to show/hide modal

**Receive Referral Logic:**
```powerapps
OnSuccess: |-
  =Patch(
      Units,
      varActiveUnit,
      {
          'Referral Household': {
              '@odata.type': "#Microsoft.Azure.Connectors.SharePoint.SPListExpandedReference",
              Id: ReferralForm.LastSubmit.ID,
              Value: ReferralForm.LastSubmit.Title
          },
          'Has referral': true
      }
  );
  Set(varShowReceiveForm, false);
  Refresh(Units);
  Refresh('Referrals & Applicants');
```

**Status:** ✅ Complete and functional

**Known Issues:**
- No OnSuccess handler on main Form1 (no user feedback after status edits)
- Consider adding Notify() for successful saves

---

### 3. RNABrowseScreen ✅ COMPLETE

**Purpose:** Browse all active referrals and applicants (excludes "former" records).

**Layout:**
- Header: Blue rectangle
- Gallery: 3-column grid of referral cards

**Key Components:**
- `RNAGallery` - Gallery displaying referrals

**Data Flow:**
```
OnVisible: Load colStatusPalette
Gallery.Items: Sort and filter referrals (exclude "former")
Gallery.OnSelect: Set varSelectedReferral, varReferralUnit → Navigate to RNAInfoScreen
```

**Formula Details:**
```powerapps
Items: |-
  =Sort(
      Filter(
          'Referrals & Applicants',
          !StartsWith(Lower(Coalesce('Household Status'.Value, "")), "former")
      ),
      'Unit Number'.Value,
      SortOrder.Ascending
  )
OnSelect: |-
  =Set(varSelectedReferral, ThisItem);
  Set(varReferralUnit, LookUp(Units, ID = varSelectedReferral.'Unit Number'.Id));
  Set(varFormMode, FormMode.View);
  Navigate(RNAInfoScreen, ScreenTransition.None)
```

**Variables Set:**
- `varSelectedReferral` - Selected referral record
- `varReferralUnit` - Unit record for selected referral
- `varFormMode` - Set to view mode

**Status:** ✅ Complete and functional

**Performance Notes:**
- Filter with `!StartsWith()` is delegable to SharePoint
- Sort by lookup field `.Value` works correctly

---

### 4. RNAInfoScreen ✅ MOSTLY COMPLETE

**Purpose:** Display and edit detailed referral/applicant information with workflow buttons.

**Layout:**
- Header: Unit and referral info
- Left: Workflow buttons (Phone Screening, Intake, Create Resident, Close Referral)
- Right: Form with referral details

**Key Components:**
- `Form1` - Main referral edit form
- Workflow buttons: `Button1` (Phone Screening), `Button2` (Intake), `Button3` (Create Resident), `Button4` (Close Referral)

**Data Flow:**
```
OnVisible: Ensure varSelectedReferral and varReferralUnit are set
Workflow buttons: Navigate to respective workflow screens
Form edits: User can edit referral details
```

**Workflow Button Logic:**

**Phone Screening (Button1):** ✅ Complete
```powerapps
OnSelect: =Navigate(PhoneScreeningScreen, ScreenTransition.None)  // Not in this export
```

**Intake (Button2):** ⚠️ In Progress
```powerapps
OnSelect: =Navigate(IntakeScreen, ScreenTransition.None)
```

**Create Resident (Button3):** ✅ JUST COMPLETED
```powerapps
OnSelect: =Navigate(scrConvert, ScreenTransition.None)
```

**Close Referral (Button4):** ✅ Complete
```powerapps
OnSelect: |-
  =Set(varCloseRec, varSelectedReferral);
  Navigate(RNACloseScreen, ScreenTransition.None)
```

**Status:** ✅ Mostly complete

**Known Issues:**
- No OnSuccess handler on Form1 (no user feedback, no data refresh)
- Phone Screening screen not in this export (may be in different version)

**Needs:**
- Add `OnSuccess` to Form1 for user notifications and data refresh

---

### 5. RNACloseScreen ✅ COMPLETE

**Purpose:** Close out a referral that declined, withdrew, or was otherwise not housed.

**Layout:**
- Dark background
- Form with three fields:
  - Close-out Date (DatePicker)
  - Close-out Reason (ComboBox)
  - Close-out Notes (TextBox)

**Key Components:**
- `EditForm1` - Form for Referrals & Applicants

**Data Flow:**
```
Item: varCloseRec (set from RNAInfoScreen)
OnSuccess:
  1. Capture last submitted record
  2. Calculate new status ("Former Referral" or "Former Applicant")
  3. Patch status change and clear unit
  4. If unit exists, clear referral household from unit
  5. Refresh data and navigate back
```

**Formula Analysis:**
```powerapps
OnSuccess: |-
  =Set(varClosedReferral, EditForm1.LastSubmit);

  // Calculate new status
  Set(
      varClosedStatus,
      With(
          { currentStatus: Lower(Coalesce(varClosedReferral.'Household Status'.Value, "")) },
          If(
              StartsWith(currentStatus, "referral"),
              "Former Referral",
              If(
                  StartsWith(currentStatus, "applicant"),
                  "Former Applicant",
                  varClosedReferral.'Household Status'.Value
              )
          )
      )
  );

  // Update referral status and clear unit
  Patch(
      'Referrals & Applicants',
      varClosedReferral,
      {
          'Household Status': { Value: varClosedStatus },
          'Unit Number': Blank()
      }
  );

  // Update unit (clear referral household)
  If(
      !IsBlank(varClosedReferral.'Unit Number'),
      Patch(
          Units,
          LookUp(Units, ID = varClosedReferral.'Unit Number'.Id),
          {
              'Referral Household': Blank(),
              'Has referral': false
          }
      )
  );

  Refresh('Referrals & Applicants');
  Navigate(RNABrowseScreen, ScreenTransition.None)
```

**Variables Used:**
- `varCloseRec` - Referral to close (input)
- `varClosedReferral` - Captured after form submission
- `varClosedStatus` - Computed status value

**Status:** ✅ Complete and functional

**Design Pattern Notes:**
- Excellent example of multi-step workflow
- Uses `With()` for cleaner intermediate calculation
- Handles nullable unit reference gracefully with `If(!IsBlank(...))`
- Updates both referral and unit records for data consistency

---

### 6. scrConvert ✅ JUST COMPLETED

**Purpose:** Convert a referral/applicant to a current resident upon move-in.

**Layout:**
- Header: Unit and referral info
- Left: FormViewer showing referral data for review
- Right: Input fields
  - Date of Leasing (DatePicker)
  - Assign to Coordinator (TextInput)
  - Associate Service Providers (ComboBox multi-select)
- Bottom: "Convert to Current Resident" button

**Key Components:**
- `FormViewer1` - Display referral data
- `DatePicker1` - Lease signing date
- `TextInput1` - Coordinator name
- `ComboBox1` - Service providers selection
- `Button3` - Convert button

**Data Flow:**
```
OnVisible: (none currently)
Button3.OnSelect:
  1. Create Current Resident record from referral data
  2. Show success notification
  3. Update Unit (set occupant, clear referral, mark occupied)
  4. Remove referral record
  5. Set varSelectedResident
  6. Navigate to ResInfoScreen
  7. Set form to edit mode
```

**Complete Formula (WORKING):**
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

**Critical Learning - Field Type Handling:**

This workflow was developed using **progressive debugging** starting with minimal fields:

**Step 1 - Minimal test (Title + required Household Status):** ✅ Works
**Step 2 - Add Unit Number (lookup):** ✅ Works
**Step 3 - Add Associated Providers (multi-lookup from ComboBox):** ✅ Works
**Step 4 - Add text fields (name, phone, email, etc.):** ✅ Works
**Step 5 - Add Date of Birth (date):** ✅ Works
**Step 6 - Add Veteran Status (choice in source):** ✅ Works (takes entire object)
**Step 7 - Add Disability & Benefits (multi-choice):** ✅ Works (takes array)

**Key Discoveries:**
1. **Lookup fields** - Pass entire object, SharePoint handles it
2. **Multi-lookup fields** - ComboBox.SelectedItems returns correct format
3. **Choice fields** - Can pass object OR .Value depending on destination type
4. **Multi-choice fields** - Pass entire array
5. **Text fields** - Simple copy works
6. **Date fields** - DatePicker.SelectedDate works directly

**Why RemoveIf() instead of Remove():**
- `Remove()` only cleared the Title field
- `RemoveIf()` properly deletes the entire record by ID

**Variables Used:**
- `varSelectedReferral` - Source referral data
- `varNewResident` - Newly created resident (captured to get ID)

**Status:** ✅ COMPLETE - Just finished and tested successfully

**Future Enhancements:**
- Add validation (ensure required fields are filled)
- Add confirmation dialog before conversion
- Pre-populate coordinator with current user
- Pre-populate service providers from referral's associated providers

---

### 7. IntakeScreen ⚠️ IN PROGRESS

**Purpose:** Guide staff through intake documentation requirements based on unit type.

**Layout:**
- Header: Unit and referral info
- Left: Container displaying eligibility requirements
  - Household Composition (min/max occupancy)
  - Income Restriction (guidelines and limits)
  - Other Eligibility (homelessness, disability docs)
- Right: (EMPTY - Form needed)

**Key Components:**
- `Container6` - Eligibility requirements display
- `Form2` - Empty form placeholder

**Data Flow:**
```
OnVisible: |-
  Set(varIntakeReferral, varSelectedReferral);
  Set(varIntakeUnit, LookUp(Units, ID = varIntakeReferral.'Unit Number'.Id));
  Set(varUnitType, LookUp('Unit Types', 'Unit Type' = varIntakeUnit.'Unit Type'.Value));
```

**What Works:**
- ✅ Loads unit type and displays eligibility requirements
- ✅ Shows min/max occupancy, income limits, homelessness definitions, disability docs

**What's Missing:**
- ❌ No intake form (Form2 is placeholder, not configured)
- ❌ No workflow logic
- ❌ No save functionality
- ❌ No navigation after completion

**Status:** ⚠️ 30% complete - Requirements display works, form not built

**What Needs to Be Built:**
1. Form with intake fields:
   - Documentation checklist (checkboxes for required docs)
   - Intake completion date
   - Staff notes
   - Verification status
2. OnSuccess handler to:
   - Update referral record
   - Set "Intake Completed" flag
   - Navigate back to RNAInfoScreen
3. Validation logic based on unit type requirements

**Estimated Effort:** 6-8 hours

---

### 8. ResBrowseScreen ✅ COMPLETE

**Purpose:** Browse all current residents.

**Layout:**
- Header: Blue rectangle
- Gallery: Grid of resident cards

**Key Components:**
- `ResidentGallery` - Gallery displaying residents

**Data Flow:**
```
OnVisible: Load colStatusPalette
Gallery.Items: Filter residents (exclude "former"), sort by unit
Gallery.OnSelect: Set varSelectedResident, varResidentUnit → Navigate
```

**Formula Details:**
```powerapps
Items: |-
  =SortByColumns(
      Filter(
          'Current Residents',
          !StartsWith(Lower(Coalesce('Household Status'.Value, "")), "former")
      ),
      "UnitNumber", Ascending
  )
OnSelect: |-
  =Set(varSelectedResident, ThisItem);
  Set(varResidentUnit, LookUp(Units, ID = varSelectedResident.'Unit Number'.Id));
  Navigate(ResInfoScreen, ScreenTransition.None)
```

**Variables Set:**
- `varSelectedResident` - Selected resident record
- `varResidentUnit` - Unit record for resident

**Status:** ✅ Complete and functional

**Performance Note:**
- Uses `SortByColumns()` instead of `Sort()` for better delegation
- Sorts by internal field name "UnitNumber" for SharePoint delegation

---

### 9. ResInfoScreen ✅ MOSTLY COMPLETE

**Purpose:** Display and edit resident details with access to associations and workflows.

**Layout:**
- Header: Resident name and unit
- Left: Resident details form
- Right: Workflow buttons (Associations, Exit Resident)

**Key Components:**
- `Form1` - Resident edit form
- `Button_Associations` - Navigate to associations screen
- `Button_Exit` - Exit resident workflow

**Data Flow:**
```
OnVisible: Ensure varSelectedResident and varResidentUnit set
Form.Item: varSelectedResident
Button_Associations: Navigate to scrResidentAssociations
Button_Exit: Archive resident, update unit, navigate away
```

**Exit Resident Logic (partial):**
```powerapps
OnSelect: |-
  =// Archive resident
  Patch(
      'Archived Records',
      Defaults('Archived Records'),
      {
          'Linked Resident': varSelectedResident,
          'Archive Date': Today(),
          'Archive Reason': "Moved Out"
      }
  );

  // Update resident status
  Patch(
      'Current Residents',
      varSelectedResident,
      { 'Household Status': { Value: "Former Occupant" } }
  );

  // Navigate away
  Navigate(ResBrowseScreen, ScreenTransition.None)
```

**Status:** ✅ Mostly complete

**Known Issues:**
- ❌ Exit resident doesn't update Unit record (doesn't clear Occupant Household, doesn't set Is occupied = false)
- ❌ No OnSuccess handler on Form1
- ⚠️ No "Hide empty fields in view mode" feature

**What Needs to Be Added to Exit Workflow:**
```powerapps
// Add this after archiving resident:
Patch(
    Units,
    varResidentUnit,
    {
        'Occupant Household': Blank(),
        'Is occupied': false,
        'Date Available': Today()
    }
);
```

**Estimated Effort to Fix:** 2-3 hours

---

### 10. scrResidentAssociations ⚠️ PARTIALLY COMPLETE

**Purpose:** Three-column view showing resident's bill pay accounts, service providers, and household members.

**Layout:**
- Three equal columns:
  - Left: Bill Pay Accounts
  - Center: Service Providers
  - Right: Household Members (not started)

**Key Components:**
- `Gallery_Accounts` - Displays resident's accounts
- `Gallery_Providers` - Displays associated providers
- `Gallery_HouseholdMembers` - Placeholder (not built)

**Status:** ⚠️ 65% complete

**What Works:**
- ✅ Bill pay accounts display
- ✅ Service providers display
- ✅ Layout structure

**What's Missing:**
- ❌ Household members section (completely missing)
- ❌ Add/edit/delete functionality for inline management

**Household Members Needs:**
1. Create `Household Members` SharePoint list (or use existing)
2. Fields: Name, Relationship, Date of Birth, Notes
3. Lookup to Current Residents
4. Gallery + Form in scrResidentAssociations
5. Add/Edit/Delete buttons

**Estimated Effort:** 4-6 hours

---

### 11. scrResidentAccounts ✅ READY FOR DEPLOYMENT (pending password solution)

**Purpose:** Manage bill pay account information for Housing Support residents.

**Layout:**
- Form for creating/editing accounts
- Conditional fields (billing info shows only for payable accounts)

**Key Components:**
- `AccountForm` - Form for Account Information list

**Data Flow:**
```
OnVisible: Set default resident to varSelectedResident
Form fields: Account provider, username, account number, billing frequency, cost
Conditional visibility: Billing fields visible only if "Is Payable Account" = true
OnSuccess: Navigate back to associations
```

**Status:** ✅ 100% functional - Ready for deployment

**Deployment Blocker:**
- ⚠️ Password security not resolved
- **Recommendation:** Remove password field, use browser password managers

**Once Password Field Removed:**
- Can deploy immediately
- All other functionality works perfectly

---

### 12. scrServiceProviders ✅ COMPLETE

**Purpose:** Browse and manage service providers directory.

**Layout:**
- Gallery of providers
- Form for viewing/editing provider details

**Key Components:**
- `ProvidersGallery` - List of providers
- `ProviderForm` - Provider details form

**Data Flow:**
```
Gallery.Items: Service Providers list
Gallery.OnSelect: Select provider for viewing/editing
Form: Display/edit provider details (name, organization, role, contact)
```

**Status:** ✅ Complete and functional

**Known Issues:**
- ⚠️ Cannot create new providers through app (permission issue)
- Must create in SharePoint directly

**Permission Issue:**
- Check SharePoint list permissions
- Verify Power Apps connection has "Add items" permission
- May require Azure AD app registration changes

---

### 13. srcOrganizations ✅ COMPLETE

**Purpose:** Browse and manage agencies/organizations.

**Layout:**
- Gallery of organizations
- Form for viewing/editing organization details

**Key Components:**
- `OrganizationsGallery` - List of agencies
- `OrganizationForm` - Organization details form

**Data Flow:**
```
Gallery.Items: Agencies list
Gallery.OnSelect: Select organization for viewing/editing
Form: Display/edit organization details (name, services, contacts)
```

**Status:** ✅ Complete and functional

**Same Permission Issue as scrServiceProviders:**
- Cannot add new organizations through app
- Must use SharePoint directly

---

### 14. Screen1 ❓ UNKNOWN/UNUSED

**Purpose:** Unknown - likely default screen from app creation

**Status:** Can likely be deleted

---

### 15. _EditorState.pa.yaml 🔧 SYSTEM FILE

**Purpose:** Stores Power Apps editor state (not user-facing)

**Do Not Edit**

---

## Workflow Implementation Status

### Summary Table

| Workflow | Screen | Status | Completeness | Blockers |
|----------|--------|--------|--------------|----------|
| Browse Units | UnitBrowseScreen | ✅ Complete | 100% | None |
| View Unit Details | UnitInfoScreen | ✅ Complete | 100% | None |
| Receive Referral | UnitInfoScreen | ✅ Complete | 100% | None |
| Browse Referrals | RNABrowseScreen | ✅ Complete | 100% | None |
| View Referral Details | RNAInfoScreen | ✅ Complete | 95% | No OnSuccess handler |
| Phone Screening | (Not in export) | ❓ Unknown | ? | Screen missing from export |
| Intake | IntakeScreen | ⚠️ In Progress | 30% | Form not built |
| Convert to Resident | scrConvert | ✅ Complete | 100% | None - JUST FINISHED! |
| Close Referral | RNACloseScreen | ✅ Complete | 100% | None |
| Browse Residents | ResBrowseScreen | ✅ Complete | 100% | None |
| View Resident Details | ResInfoScreen | ✅ Complete | 95% | No OnSuccess handler |
| Exit Resident | ResInfoScreen | ⚠️ Incomplete | 70% | Doesn't update unit |
| Resident Associations | scrResidentAssociations | ⚠️ Partial | 65% | Household members missing |
| Resident Accounts | scrResidentAccounts | ✅ Ready | 100% | Password security policy needed |
| Service Providers | scrServiceProviders | ✅ Complete | 100% | Creation permission issue |
| Organizations | srcOrganizations | ✅ Complete | 100% | Creation permission issue |

### Workflow Deep Dives

#### ✅ Receive Referral Workflow (COMPLETE)

**User Journey:**
1. User navigates to UnitInfoScreen for a vacant unit
2. Clicks "Receive Referral" button (visible only if unit is vacant and has no existing referral)
3. Modal form appears
4. User enters referral name (head of household)
5. Form auto-populates:
   - Unit Number (from current unit)
   - Unit Type (from current unit)
   - Date Referred (today)
   - Household Status (default: "Referral, New")
6. User clicks Save
7. System creates referral record
8. System updates unit record (sets Referral Household, Has referral = true)
9. Modal closes, data refreshes
10. User can navigate to new referral

**Formula Breakdown:**
```powerapps
// Button visibility (show only when applicable)
Visible: =And(
    varActiveUnit.'Is occupied' = false,
    IsBlank(varActiveUnit.'Referral Household')
)

// Form defaults (pre-populate fields)
Default: =If(
    DataCard.DisplayName = "Unit Number",
    varActiveUnit,
    If(
        DataCard.DisplayName = "Date Referred",
        Today(),
        // ... other defaults
    )
)

// OnSuccess (after form saves)
OnSuccess: |-
  =Patch(
      Units,
      varActiveUnit,
      {
          'Referral Household': {
              '@odata.type': "#Microsoft.Azure.Connectors.SharePoint.SPListExpandedReference",
              Id: ReferralForm.LastSubmit.ID,
              Value: ReferralForm.LastSubmit.Title
          },
          'Has referral': true
      }
  );
  Set(varShowReceiveForm, false);
  Refresh(Units);
  Refresh('Referrals & Applicants');
```

**Testing Checklist:**
- [ ] Button shows only for vacant units
- [ ] Button hides if unit already has referral
- [ ] Button hides if unit is occupied
- [ ] Form opens when clicked
- [ ] Unit number pre-fills correctly
- [ ] Date referred defaults to today
- [ ] Save creates referral record
- [ ] Save updates unit's referral household
- [ ] Save sets Has referral = true
- [ ] Modal closes after save
- [ ] Data refreshes in browse view
- [ ] Can navigate to new referral immediately

#### ✅ Close Referral Workflow (COMPLETE)

**User Journey:**
1. User on RNAInfoScreen viewing a referral
2. Clicks "Close Referral" workflow button
3. Navigates to RNACloseScreen
4. Form shows with three fields:
   - Close-out Date (defaults to today)
   - Close-out Reason (dropdown)
   - Close-out Notes (text)
5. User fills out form and saves
6. System:
   - Adds "Former" prefix to household status
   - Clears unit number from referral
   - Clears referral household from unit
   - Sets unit's Has referral = false
7. User returns to RNABrowseScreen
8. Closed referral no longer appears (filtered out by "former" status)

**Formula Breakdown:**
```powerapps
// Calculate new status (add "Former" prefix)
Set(
    varClosedStatus,
    With(
        { currentStatus: Lower(Coalesce(varClosedReferral.'Household Status'.Value, "")) },
        If(
            StartsWith(currentStatus, "referral"),
            "Former Referral",
            If(
                StartsWith(currentStatus, "applicant"),
                "Former Applicant",
                varClosedReferral.'Household Status'.Value
            )
        )
    )
);

// Update referral
Patch(
    'Referrals & Applicants',
    varClosedReferral,
    {
        'Household Status': { Value: varClosedStatus },
        'Unit Number': Blank()
    }
);

// Update unit (if exists)
If(
    !IsBlank(varClosedReferral.'Unit Number'),
    Patch(
        Units,
        LookUp(Units, ID = varClosedReferral.'Unit Number'.Id),
        {
            'Referral Household': Blank(),
            'Has referral': false
        }
    )
);
```

**Testing Checklist:**
- [ ] Close button appears on RNAInfoScreen
- [ ] Clicking navigates to RNACloseScreen
- [ ] Form loads with referral data
- [ ] Close-out date defaults to today
- [ ] Reason dropdown shows all options
- [ ] Notes field accepts text
- [ ] Save updates referral status correctly ("Referral, New" → "Former Referral")
- [ ] Save updates applicant status correctly ("Applicant, Approved" → "Former Applicant")
- [ ] Save clears unit number from referral
- [ ] Save clears referral household from unit
- [ ] Save sets unit's Has referral = false
- [ ] Closed referral disappears from browse view
- [ ] Unit now shows as available for new referral

#### ✅ Convert to Resident Workflow (JUST COMPLETED!)

**User Journey:**
1. User on RNAInfoScreen viewing an approved applicant
2. Clicks "Create Resident" workflow button
3. Navigates to scrConvert
4. Reviews referral data in FormViewer
5. Selects:
   - Lease signing date (DatePicker)
   - Assigned coordinator (TextInput - optional)
   - Associated service providers (ComboBox multi-select)
6. Clicks "Convert to Current Resident"
7. System:
   - Creates Current Resident record with all referral data
   - Updates unit (sets Occupant Household, clears Referral Household, marks occupied)
   - Deletes referral record
   - Navigates to new resident's ResInfoScreen in edit mode
8. User can immediately edit/verify resident details

**Formula (Complete and Working):**
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

**Testing Checklist:**
- [ ] Button appears on RNAInfoScreen
- [ ] Clicking navigates to scrConvert
- [ ] FormViewer shows all referral data correctly
- [ ] DatePicker defaults to today or selected date
- [ ] TextInput accepts coordinator name
- [ ] ComboBox shows available service providers
- [ ] ComboBox allows multiple selections
- [ ] Convert button clickable
- [ ] Success notification appears
- [ ] New resident record created with correct data
- [ ] All text fields copied correctly
- [ ] Date of birth copied correctly
- [ ] Veteran status copied correctly (choice → text)
- [ ] Disability copied correctly (multi-choice → multi-choice)
- [ ] Benefits copied correctly (multi-choice → multi-choice)
- [ ] Unit Number copied correctly (lookup → lookup)
- [ ] Associated Providers from ComboBox saved correctly
- [ ] Lease signing date from DatePicker saved correctly
- [ ] Household Status set to "Occupant, Current"
- [ ] Unit's Occupant Household set to new resident
- [ ] Unit's Referral Household cleared
- [ ] Unit's Referral Requested cleared
- [ ] Unit's Has referral set to false
- [ ] Unit's Is occupied set to true
- [ ] Referral record fully deleted (not just Title cleared)
- [ ] Navigation to ResInfoScreen successful
- [ ] varSelectedResident set correctly
- [ ] Form in edit mode on arrival
- [ ] Former referral no longer in browse view
- [ ] New resident appears in residents browse view
- [ ] Unit shows as occupied with resident name

**Known Edge Cases to Test:**
- [ ] Referral with no associated providers
- [ ] Referral with missing optional fields
- [ ] Referral with no veteran status set
- [ ] Referral with no disability set
- [ ] Referral with no benefits set
- [ ] Coordinator field left blank
- [ ] Service providers field left empty

#### ⚠️ Intake Workflow (IN PROGRESS - 30% complete)

**What Exists:**
- Screen loads
- Displays unit type eligibility requirements
- Shows min/max occupancy
- Shows income restrictions
- Shows homelessness requirements
- Shows disability documentation options

**What's Missing:**
- Form for data entry
- Fields for documentation checklist
- Save functionality
- Status update logic
- Navigation after completion

**What Needs to Be Built:**

**Form Fields Needed:**
```
- Intake Completion Date (Date, required)
- Intake Completed By (Person/Text, required)
- Documentation Checklist (Multi-choice or individual Yes/No)
  - State ID or Driver's License (Yes/No)
  - Social Security Card (Yes/No)
  - Birth Certificate (Yes/No)
  - Income Verification (Yes/No)
  - Disability Documentation (Yes/No, if required)
  - Homelessness Verification (Yes/No, if required)
  - Professional Statement of Need (Yes/No, if required)
  - [Other docs based on unit type requirements]
- Verification Notes (Multi-line text)
- Eligibility Confirmed (Yes/No, required)
```

**OnSuccess Logic Needed:**
```powerapps
OnSuccess: |-
  =Patch(
      'Referrals & Applicants',
      varIntakeReferral,
      {
          'Intake Completed': IntakeForm.LastSubmit.'Intake Completion Date',
          'Household Status': { Value: "Applicant, Pending Approval" }
      }
  );
  Notify("Intake completed successfully.", NotificationType.Success);
  Refresh('Referrals & Applicants');
  Navigate(RNAInfoScreen, ScreenTransition.None)
```

**Estimated Effort:** 6-8 hours
- 2 hours: Design form fields and layout
- 2 hours: Build form and connect to data
- 1 hour: Add validation logic based on unit type
- 1 hour: Implement OnSuccess workflow
- 1-2 hours: Testing and refinement

**Testing Checklist (Future):**
- [ ] Screen loads with correct unit type requirements
- [ ] Form displays all required documentation fields
- [ ] Required fields enforce validation
- [ ] Optional fields based on unit type show/hide correctly
- [ ] Save updates referral record
- [ ] Save sets Intake Completed date
- [ ] Save advances Household Status
- [ ] Navigation returns to referral detail screen
- [ ] Intake button disabled after completion

#### ⚠️ Exit Resident Workflow (INCOMPLETE - 70% complete)

**What Works:**
- Archives resident record
- Updates resident status to "Former Occupant"
- Navigates away

**What's Missing:**
- ❌ Doesn't update Unit record
- ❌ Doesn't clear Occupant Household
- ❌ Doesn't set Is occupied = false
- ❌ Doesn't set Date Available

**Fix Required:**
```powerapps
// Add after resident archiving, before navigation:
Patch(
    Units,
    varResidentUnit,
    {
        'Occupant Household': Blank(),
        'Is occupied': false,
        'Date Available': Today()
    }
);
Refresh(Units);
```

**Testing Checklist (After Fix):**
- [ ] Exit button appears on ResInfoScreen
- [ ] Clicking archives resident
- [ ] Resident status updated to "Former Occupant"
- [ ] Archive record created in Archived Records
- [ ] Archive date set to today
- [ ] Unit's Occupant Household cleared
- [ ] Unit's Is occupied set to false
- [ ] Unit's Date Available set to today
- [ ] Former resident disappears from browse view
- [ ] Unit shows as vacant in Units view
- [ ] Unit available for new referral

---

## Testing Strategy

### Progressive Debugging Approach

**Always start with the smallest testable piece and build up.**

#### Testing a New Patch Formula

**Step 1: Minimal required fields**
```powerapps
// Test with ONLY required fields
Patch(DataSource, Defaults(DataSource), {
    RequiredField1: "Test Value"
})
```
✅ Success? → Proceed to Step 2
❌ Failure? → Fix required field issue first

**Step 2: Add simple fields one at a time**
```powerapps
Patch(DataSource, Defaults(DataSource), {
    RequiredField1: "Test",
    TextField1: varSource.TextField1
})
```
✅ Success? → Add next field
❌ Failure? → Problem is with TextField1

**Step 3: Add complex fields individually**
```powerapps
// Test lookup field separately
Patch(DataSource, Defaults(DataSource), {
    RequiredField1: "Test",
    LookupField: varSource.LookupField
})
```

**Step 4: Add multi-value fields**
```powerapps
// Test multi-choice or multi-lookup
Patch(DataSource, Defaults(DataSource), {
    RequiredField1: "Test",
    MultiChoiceField: varSource.MultiChoiceField
})
```

**Step 5: Complete formula**
```powerapps
// All fields together
Patch(DataSource, Defaults(DataSource), { ... })
```

#### Testing Forms

**Test Form Submission:**
1. Create minimal form with one required field
2. Test submit
3. Add one field at a time
4. Test after each addition
5. When save fails, you've found the problematic field

**Display Variable Contents:**
```powerapps
// Add a Label control to see what's in a variable
Label.Text: =JSON(varSelectedReferral, JSONFormat.IndentFour)
```

**Check Lookup Structure:**
```powerapps
// Verify lookup has Id and Value
Label.Text: ="ID: " & varSelectedReferral.'Unit Number'.Id &
            " | Value: " & varSelectedReferral.'Unit Number'.Value
```

#### Testing Navigation

**Step 1: Navigate with no variables**
```powerapps
Navigate(TargetScreen, ScreenTransition.None)
```

**Step 2: Add one Set() at a time**
```powerapps
Set(var1, value1);
Navigate(TargetScreen, ScreenTransition.None)
```

**Step 3: Add all Sets**
```powerapps
Set(var1, value1);
Set(var2, value2);
Set(var3, value3);
Navigate(TargetScreen, ScreenTransition.None)
```

### Manual Testing Checklists

#### Complete Workflow Tests

**Test: Receive Referral → Intake → Convert → Exit**

**Pre-conditions:**
- Vacant unit exists (Unit 101)
- Unit has no current referral
- User has permissions to create records

**Steps:**
1. Navigate to Units view
2. Toggle "Show Vacancies"
3. Verify Unit 101 appears
4. Click Unit 101
5. Verify "Receive Referral" button visible
6. Click "Receive Referral"
7. Verify modal form opens
8. Enter name: "Test Resident"
9. Verify Unit Number pre-filled: "101"
10. Verify Date Referred = Today
11. Click Save
12. Verify modal closes
13. Verify unit shows referral household
14. Navigate to Referrals & Applicants
15. Verify "Test Resident" appears
16. Click "Test Resident"
17. Verify referral details screen loads
18. Click "Phone Screening" (if available)
19. Complete phone screening
20. Return to referral details
21. Click "Intake"
22. Complete intake form (WHEN BUILT)
23. Save intake
24. Return to referral details
25. Verify status updated to "Applicant"
26. Click "Create Resident"
27. Verify scrConvert screen loads
28. Review referral data in left panel
29. Select lease signing date
30. Enter coordinator name (optional)
31. Select service providers (optional)
32. Click "Convert to Current Resident"
33. Verify success notification
34. Verify navigation to ResInfoScreen
35. Verify resident details displayed
36. Verify form in edit mode
37. Navigate to Residents browse
38. Verify "Test Resident" appears
39. Navigate to Units browse
40. Verify Unit 101 shows as occupied
41. Verify Unit 101 shows "Test Resident"
42. Navigate to Referrals browse
43. Verify "Test Resident" no longer appears
44. Navigate back to resident details
45. Click "Exit Resident"
46. Confirm exit
47. Verify navigation away
48. Navigate to Residents browse
49. Verify "Test Resident" no longer appears
50. Navigate to Units browse
51. Verify Unit 101 shows as vacant
52. Verify "Date Available" = Today

**Expected Results:**
- All steps complete without errors
- Data consistent across all views
- Unit status reflects current state
- Referral → Resident → Former Resident lifecycle works

**Actual Results:**
- [To be filled during testing]

**Pass/Fail:**
- [ ] PASS
- [ ] FAIL - Details: ___________

#### Individual Screen Tests

**Test Template:**
```
Test: [Screen Name] - [Specific Feature]

Pre-conditions:
- [Required setup]

Steps:
1. [Action 1]
2. [Action 2]
...

Expected Results:
- [Expected outcome 1]
- [Expected outcome 2]

Actual Results:
- [What actually happened]

Pass/Fail:
- [ ] PASS
- [ ] FAIL - Details: ___________
```

#### Edge Case Testing

**Test: Convert referral with no optional fields**
- Referral has only name and unit
- No phone, no email, no providers
- Should still convert successfully

**Test: Convert referral with all fields**
- Every field populated
- Multiple service providers
- Multiple disabilities
- Should handle all data correctly

**Test: Close referral with no unit**
- Referral created without unit assignment
- Close referral workflow
- Should not error when trying to update unit

**Test: Concurrent users**
- Two users viewing same referral
- Both try to convert
- Second conversion should fail gracefully

**Test: Missing lookup targets**
- Referral references deleted unit
- Should handle gracefully
- Display error message or "N/A"

### Performance Testing

**Large Dataset Test:**
- Create 100+ units
- Create 100+ referrals
- Create 100+ residents
- Test browse gallery performance
- Check for delegation warnings

**Delegation Warning Check:**
```
File → Settings → Advanced settings → Enable formula bar
Check for yellow warning triangles in formulas
Review delegation warnings and fix if possible
```

**Improve Delegation:**
- Use `SortByColumns()` instead of `Sort()` with internal field names
- Use `Filter()` with simple conditions (`=`, `<>`, `StartsWith`)
- Avoid complex `Or()` conditions when possible
- Pre-load collections in `OnVisible` for complex joins

### Regression Testing

After any change, test:
1. Navigation between all screens
2. Variable persistence (navigate away and back)
3. Form save operations
4. Filter and sort in galleries
5. Lookup field displays
6. Related record updates

---

## Known Issues & Bugs

### Critical Issues (Blocking Deployment)

#### 🔴 IntakeScreen Not Functional
**Severity:** HIGH
**Impact:** Core workflow unavailable
**Symptoms:** Screen displays requirements but has no form
**Cause:** Form2 placeholder not configured
**Workaround:** Manual intake outside app
**Fix Required:** Build intake form (6-8 hours)
**Status:** In progress

#### 🔴 Exit Resident Doesn't Update Unit
**Severity:** HIGH
**Impact:** Data integrity - units remain marked as occupied
**Symptoms:** Former residents removed, but unit still shows occupied
**Cause:** Missing Patch() for Units in exit workflow
**Workaround:** Manually update unit in SharePoint
**Fix Required:** Add unit update logic (2-3 hours)
**Status:** Known, not fixed

#### 🔴 Password Storage Security
**Severity:** HIGH (Security)
**Impact:** Compliance risk
**Symptoms:** Account passwords stored in plain text
**Cause:** No secure storage mechanism
**Workaround:** Remove password field, use browser password managers
**Fix Required:** Policy decision + field removal (1-2 hours)
**Status:** Awaiting decision

### Medium Issues (Should Fix Before Launch)

#### 🟡 No OnSuccess Handlers on Forms
**Severity:** MEDIUM
**Impact:** No user feedback, no data refresh
**Affected Screens:** RNAInfoScreen, ResInfoScreen, UnitInfoScreen
**Symptoms:** Silent saves, stale data after edit
**Cause:** Forms missing OnSuccess property
**Workaround:** Manually refresh browser
**Fix Required:** Add OnSuccess handlers with Notify() and Refresh()
**Estimated Effort:** 1-2 hours
**Status:** Known, not fixed

#### 🟡 Cannot Create Providers/Orgs Through App
**Severity:** MEDIUM
**Impact:** Poor user experience
**Symptoms:** Add new provider button doesn't work
**Cause:** SharePoint permissions or Azure AD configuration
**Workaround:** Create in SharePoint directly
**Fix Required:** Troubleshoot permissions (2-4 hours)
**Status:** Under investigation

#### 🟡 Household Members Section Missing
**Severity:** MEDIUM
**Impact:** Cannot track family composition
**Symptoms:** scrResidentAssociations shows placeholder
**Cause:** Not yet built
**Workaround:** Track in notes or external spreadsheet
**Fix Required:** Build household members feature (4-6 hours)
**Status:** Planned for Friday

### Low Issues (Nice to Have)

#### 🟢 Variable Naming Inconsistency
**Severity:** LOW
**Impact:** Code maintainability
**Symptoms:** Mix of varActive*, varSelected*
**Cause:** Evolved naming conventions
**Workaround:** None needed
**Fix Required:** Refactor variable names (2-3 hours)
**Status:** Ongoing cleanup

#### 🟢 Control Naming Inconsistency
**Severity:** LOW
**Impact:** Code maintainability
**Symptoms:** Button1, Form1, Gallery1 not descriptive
**Cause:** Default names not changed
**Workaround:** None needed
**Fix Required:** Systematic renaming (4-6 hours)
**Status:** Planned cleanup

#### 🟢 Empty Fields Always Visible
**Severity:** LOW
**Impact:** Visual clutter
**Symptoms:** All form fields show even if blank
**Cause:** No conditional visibility
**Workaround:** None needed
**Fix Required:** Add `Visible` property to data cards (1-2 hours)
**Status:** Enhancement request

### Resolved Issues (Documented for Reference)

#### ✅ Convert Workflow Silent Failure
**Status:** RESOLVED
**Symptoms:** Convert button clicked, nothing happened, no error
**Cause:** Veteran Status field type mismatch (choice → text)
**Resolution:** Extract .Value from choice field
**Fixed By:** Progressive debugging approach
**Date Fixed:** [Today]
**Lessons Learned:** Always start with minimal fields and add progressively

#### ✅ Remove() Only Clearing Title
**Status:** RESOLVED
**Symptoms:** Referral record not deleted, just Title field cleared
**Cause:** SharePoint Remove() behavior
**Resolution:** Use RemoveIf() with ID condition instead
**Fixed By:** Testing different remove approaches
**Date Fixed:** [Today]

#### ✅ Unit Type Data Not Saving in Referral Form
**Status:** RESOLVED
**Symptoms:** Unit Type showed in app but didn't save to list
**Cause:** Form binding issue
**Resolution:** Corrected data binding and field mapping
**Date Fixed:** [Earlier in project]

#### ✅ Delegation Warnings on Sort
**Status:** RESOLVED
**Symptoms:** Yellow warning triangles on gallery sort
**Cause:** Using Sort() on lookup field display values
**Resolution:** Changed to SortByColumns() with internal field names
**Date Fixed:** [Earlier in project]

---

## Development Roadmap

### Immediate Priorities (Pre-Deployment)

**Week 1: Critical Workflows**
- [ ] Complete IntakeScreen form (6-8 hours)
  - Design form fields
  - Add validation
  - Implement OnSuccess workflow
  - Test with real data
- [ ] Fix Exit Resident workflow (2-3 hours)
  - Add unit update logic
  - Test unit status changes
  - Verify vacancy displays correctly
- [ ] Resolve password management (1-2 hours)
  - Remove password field from schema
  - Document browser password manager usage
  - Update user guide

**Week 2: Polish & Stability**
- [ ] Add OnSuccess handlers to all forms (1-2 hours)
  - RNAInfoScreen Form1
  - ResInfoScreen Form1
  - UnitInfoScreen Form1
  - Add Notify() messages
  - Add Refresh() calls
- [ ] Build Household Members section (4-6 hours)
  - Create or verify Household Members list
  - Add gallery to scrResidentAssociations
  - Add form for member details
  - Implement add/edit/remove
- [ ] Troubleshoot provider creation permissions (2-4 hours)
  - Check SharePoint permissions
  - Test with different user roles
  - Document resolution

**Week 3: Testing & Refinement**
- [ ] Develop comprehensive test suite (4-8 hours)
  - Write test cases for each workflow
  - Create edge case scenarios
  - Document expected vs actual results
- [ ] Execute full regression testing
  - Test all workflows end-to-end
  - Test with multiple concurrent users
  - Test with large datasets
- [ ] Fix bugs discovered in testing
- [ ] User acceptance testing with staff

**Week 4: Deployment Preparation**
- [ ] Clean up variable naming (2-3 hours)
  - Rename varActiveUnit → varSelectedUnit
  - Ensure consistency across app
- [ ] Clean up control naming (4-6 hours)
  - Rename forms (Form1 → frmResident, etc.)
  - Rename buttons (Button1 → btnConvert, etc.)
  - Rename galleries descriptively
- [ ] Add "hide empty fields" feature (1-2 hours)
- [ ] Final documentation review
- [ ] Deployment plan finalization

### Post-Deployment Enhancements

**Phase 1: User Experience**
- [ ] Side navigation menu
  - Direct access to Providers, Organizations
  - Persistent across screens
- [ ] Add provider from association screen
  - Modal form for quick creation
  - Auto-select after creation
- [ ] Batch payment view
  - Group accounts by provider
  - Monthly payment workflow

**Phase 2: Automation**
- [ ] Power Automate flows
  - Email notifications for intake completion
  - Reminders for lease renewals
  - Automated status transitions
- [ ] ETO integration (if feasible)
  - Auto-create participants in ETO
  - Sync demographic data
- [ ] HMIS integration exploration

**Phase 3: Advanced Features**
- [ ] Dashboard view
  - Units available/occupied
  - Referrals in pipeline
  - Upcoming lease renewals
- [ ] Reporting suite
  - Occupancy reports
  - Referral pipeline metrics
  - Service provider engagement
- [ ] Mobile optimization
  - Responsive layout adjustments
  - Touch-friendly controls

### Future Considerations

**Performance Optimization:**
- Pre-load joined data in collections
- Reduce real-time lookups
- Optimize delegation where possible

**Security Enhancements:**
- Role-based access control
- Audit logging
- Secure credential storage integration

**Integration Opportunities:**
- Yardi sync (if API available)
- Microsoft Teams notifications
- Calendar integration for renewals

---

## Conclusion

Upper Post Clearview is a well-structured Power Apps canvas application that successfully consolidates supportive housing operations into a single, user-friendly interface. With 75% functionality complete, the core browse/view/edit workflows are solid and ready for use.

**Key Strengths:**
- Consistent navigation patterns
- Effective use of lookup fields for data relationships
- Color-coded visual design for quick identification
- Working workflows for referral intake and close-out
- **Just-completed Convert to Resident workflow** using progressive debugging

**Key Remaining Work:**
- Complete IntakeScreen form
- Fix Exit Resident unit updates
- Build Household Members section
- Add user feedback (OnSuccess handlers)
- Resolve permission issues for provider creation

**Development Philosophy:**
**When stuck, start with the smallest working piece and build up.** This approach solved the stubborn Convert workflow issue and should guide all future troubleshooting.

**Estimated Time to Deployment:** 23-34 hours of focused development + testing

**Next Steps:**
1. Prioritize IntakeScreen completion
2. Fix Exit Resident workflow
3. Execute comprehensive testing
4. Deploy to production

---

**Document Version:** 1.0
**Date:** October 24, 2025
**Author:** Technical documentation based on code analysis
**Last Updated:** After successful completion of Convert to Resident workflow
