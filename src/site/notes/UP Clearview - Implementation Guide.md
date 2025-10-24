---
{"dg-publish":true,"permalink":"/up-clearview-implementation-guide/"}
---

**Complete Guide to Deploying UP Clearview to Any Supportive Housing Site**

---

## Table of Contents

1. [Introduction](#introduction)
2. [Lessons Learned from Upper Post](#lessons-learned-from-upper-post)
3. [Selecting Your Implementation Site](#selecting-your-implementation-site)
4. [Pre-Deployment Planning](#pre-deployment-planning)
5. [SharePoint Schema Design Best Practices](#sharepoint-schema-design-best-practices)
6. [Data Collection and Migration Strategy](#data-collection-and-migration-strategy)
7. [SharePoint List Creation and Data Import](#sharepoint-list-creation-and-data-import)
8. [Site Size Considerations](#site-size-considerations)
9. [Step-by-Step Adaptation Process](#step-by-step-adaptation-process)
10. [Field Customization by Site](#field-customization-by-site)
11. [Testing Checklist for New Sites](#testing-checklist-for-new-sites)
12. [Common Pitfalls and Solutions](#common-pitfalls-and-solutions)
13. [Maintenance and Updates](#maintenance-and-updates)

---

## Introduction

Upper Post Clearview was built for a specific site with specific needs. However, the core functionality—tracking units, referrals, residents, and service connections—is relevant to many supportive housing sites. This guide shows you how to adapt the app for other sites with minimal pain.

**Key Insight:** Starting at a smaller site may actually be easier and allow you to refine the approach before deploying to larger, more complex sites.

---

## Lessons Learned from Upper Post

### What We Learned the Hard Way

#### 1. **Field Naming is Critical**

**The Problem:**
- SharePoint internal field names don't always match display names
- Power Apps sometimes references internal names, sometimes display names
- Example: "Close-out Date" displays nicely but becomes `Close_x002d_out_x0020_Date` internally
- Formulas become harder to read and maintain

**The Lesson:**
**Name fields in SharePoint EXACTLY how you want them to appear in the Power App.**

**Good Examples:**
- `Preferred Name` (no special characters)
- `Date of Birth` (clear, simple)
- `Unit Number` (straightforward)

**Bad Examples:**
- `Close-out Date` (hyphen creates encoding issues)
- `Head of Household (HOH)` (parentheses complicate things)
- `Unit #` (special character)

**Rule:** Use only letters, numbers, and spaces. No hyphens, apostrophes, parentheses, or special characters.

---

#### 2. **Design Your Schema Completely BEFORE Building**

**The Problem:**
- Upper Post schema evolved during development
- Fields were added mid-stream
- Some fields weren't thought through
- Led to rework and confusion

**The Lesson:**
**Have ALL fields figured out and created in SharePoint BEFORE you start building the Power App.**

**Why This Matters:**
- You can't easily rename fields after they're in use
- Adding fields later means updating multiple screens
- Changing field types breaks existing formulas
- Testing becomes repetitive

**Recommendation:**
1. Spend 1-2 weeks planning and documenting schema
2. Get buy-in from all stakeholders
3. Create ALL SharePoint lists with ALL fields
4. Add sample data to test
5. THEN start building Power App

---

#### 3. **Field Type Mismatches Cause Silent Failures**

**The Problem:**
- Upper Post had "Veteran Status" as Choice in one list, Text in another
- Convert workflow failed silently for months
- Only discovered through progressive debugging

**The Lesson:**
**If a field appears in multiple lists, make sure the field TYPE is identical.**

**Common Mismatches to Avoid:**
- Choice in one list, Text in another
- Lookup in one list, Text in another
- Multi-select in one list, Single-select in another

**Verification Step:**
Before building, create a spreadsheet mapping every field that appears in multiple lists and verify types match.

---

#### 4. **Lookup Fields Need Special Care**

**The Problem:**
- Lookup fields store objects with `Id` and `Value` properties
- Can't just copy text—need to pass entire object
- Multi-select lookups are arrays of objects
- Different handling than text fields

**The Lesson:**
**Understand how your data will flow between lists BEFORE creating lookups.**

**Questions to Ask:**
- Will this referral data need to transfer to a resident record?
- If yes, is the field type the same in both lists?
- If it's a lookup, does it point to the same target list?
- If it's a choice, do both lists have the same choice options?

---

#### 5. **Start Small, Add Features Progressively**

**The Problem:**
- Trying to build everything at once is overwhelming
- Hard to debug when multiple features are broken
- Users get confused by incomplete features

**The Lesson:**
**Launch with core functionality, add features based on actual use.**

**Recommended Phased Approach:**
- **Phase 1:** Browse, view, edit (no workflows)
- **Phase 2:** Add one workflow at a time
- **Phase 3:** Add associations and related records
- **Phase 4:** Add reporting and analytics

---

## Selecting Your Implementation Site

### Why Start with a Smaller Site?

**The Case for Starting Small:**

After building UP Clearview for Upper Post (a large, complex site with 50+ units and multiple programs), we've learned that **starting with a smaller site may actually be the better approach** for initial implementation.

**Advantages of Starting Small:**

1. **Faster Implementation**
   - Fewer units to configure
   - Simpler workflows
   - Less data to migrate
   - Shorter timeline to launch

2. **Lower Risk**
   - Fewer users affected if issues arise
   - Easier to fix problems quickly
   - Less complex to roll back if needed
   - Smaller impact on operations

3. **Better Learning Opportunity**
   - Discover issues in manageable environment
   - Refine processes before scaling
   - Build expertise gradually
   - Develop best practices organically

4. **Easier User Adoption**
   - Fewer staff to train
   - More personalized support
   - Closer relationship with users
   - Faster feedback loop

5. **Proof of Concept**
   - Demonstrate value before large investment
   - Build stakeholder confidence
   - Identify actual needs vs. assumed needs
   - Create success stories for larger rollout

**Upper Post Experience:**
Building for a large, complex site first meant:
- Long development cycle (6+ months)
- Many features added that may not be essential
- Complex workflows to accommodate multiple programs
- Extensive testing required
- Higher stakes if something goes wrong

**Recommended Alternative Path:**
1. Implement at a small site first (5-20 units)
2. Launch with minimal features in 4-6 weeks
3. Learn what actually gets used
4. Refine based on real usage
5. Add features users actually request
6. Document lessons learned
7. Roll out refined version to larger sites

---

### Site Selection Criteria

#### Ideal Characteristics for First Implementation

**✅ Good First Site:**
- **Size:** 5-20 units
- **Complexity:** Single program type (e.g., all Housing Support or all Section 8)
- **Turnover:** Moderate (not too stable, not too chaotic)
- **Staff:** 1-3 primary users who are tech-comfortable
- **Current Tracking:** Minimal or very basic (Excel or paper)
- **Stakeholder Buy-In:** Site director enthusiastic about trying new approach
- **Support Access:** Close proximity to IT/development support

**❌ Avoid for First Site:**
- Large site with 50+ units
- Multiple program types with different requirements
- Very high turnover (too chaotic for new system)
- No turnover (won't see benefit of tracking)
- Staff resistant to technology
- Complex existing tracking that works well
- Remote location with limited support access

#### Site Comparison Matrix

Use this to evaluate potential sites:

| Factor | Site A | Site B | Site C |
|--------|--------|--------|--------|
| Number of Units | ___ | ___ | ___ |
| Current Occupancy | ___% | ___% | ___% |
| Annual Turnover | ___ | ___ | ___ |
| Program Types | ___ | ___ | ___ |
| Primary Staff Users | ___ | ___ | ___ |
| Tech Comfort (1-10) | ___ | ___ | ___ |
| Director Buy-In (1-10) | ___ | ___ | ___ |
| Current Tracking Method | ___ | ___ | ___ |
| Pain Points | ___ | ___ | ___ |
| Distance from Support | ___ | ___ | ___ |
| **Total Score** | ___ | ___ | ___ |

**Scoring Guide:**
- Fewer units: Better (5-10 units = 5 points, 11-20 = 3 points, 21+ = 1 point)
- Moderate turnover: Better (1-3 per quarter = 5 points)
- Single program: Better (5 points)
- Tech comfort: Higher better (score 1-10)
- Buy-in: Higher better (score 1-10)
- Simple current tracking: Better (Excel = 5 points, Nothing = 5 points, Complex system = 1 point)

**Recommendation:** Choose site with highest total score

---

### Multi-Site Rollout Strategy

If you're planning to implement across multiple sites, use a phased approach:

#### Phase 1: Pilot Site (Weeks 1-8)
- **Select:** Smallest, simplest site
- **Build:** Minimal feature set
- **Launch:** MVP only
- **Learn:** Document everything that works and doesn't

#### Phase 2: Refinement (Weeks 9-12)
- **Analyze:** What features got used? What didn't?
- **Adjust:** Modify based on pilot feedback
- **Document:** Create updated implementation guide
- **Train:** Develop training materials from real usage

#### Phase 3: Second Site (Weeks 13-18)
- **Select:** Slightly larger or more complex than pilot
- **Deploy:** Refined version from Phase 2
- **Test:** New features with more users
- **Compare:** How does it scale?

#### Phase 4: Expansion (Weeks 19+)
- **Deploy:** To remaining sites progressively
- **Support:** Provide ongoing assistance
- **Iterate:** Continue refinement based on feedback
- **Scale:** Maintain quality as you grow

**Key Principle:** Each implementation should be easier than the last because you're learning and refining the process.

---

### Example Site Profiles

#### Profile 1: Oakwood Apartments (Ideal First Site)
- **Units:** 12 scattered site
- **Program:** LTH Housing Support only
- **Turnover:** 2-3 per year
- **Staff:** 1 housing coordinator, 1 site manager
- **Current Tracking:** Excel spreadsheet on shared drive
- **Pain Points:**
  - Hard to find current occupancy info
  - Intake docs scattered
  - No history of past referrals
- **Why Good:** Small, simple, clear need, enthusiastic staff

#### Profile 2: Riverside Towers (Second Site)
- **Units:** 28 units in one building
- **Programs:** Mix of Section 8 and Housing Support
- **Turnover:** 5-7 per year
- **Staff:** 2 case managers, 1 housing coordinator, 1 site director
- **Current Tracking:** Combination of Excel, paper files, and Yardi
- **Pain Points:**
  - Information in multiple places
  - Hard to track service provider involvement
  - Intake requirements differ by program
- **Why Second:** More complex, tests multi-program features

#### Profile 3: Upper Post (Large Site)
- **Units:** 50+ units
- **Programs:** 4 different program types
- **Turnover:** 15-20 per year
- **Staff:** 5+ staff members
- **Current Tracking:** Complex Excel workbooks, Yardi, multiple systems
- **Why Last:** Most complex, highest stakes, benefits from refined process

---

## Pre-Deployment Planning

### Step 1: Assess Site Needs (1-2 weeks)

Before touching SharePoint or Power Apps, answer these questions:

#### Site Characteristics

**Physical Inventory:**
- [ ] How many units does the site have?
- [ ] Are there multiple buildings?
- [ ] Multiple unit types/programs?
- [ ] Any unique accessibility features?

**Current Population:**
- [ ] How many current residents?
- [ ] Average length of stay?
- [ ] Turnover rate?
- [ ] Waitlist size?

**Staffing:**
- [ ] How many staff will use the app?
- [ ] What are their roles?
- [ ] What are their technical comfort levels?
- [ ] What devices do they use? (Desktop, tablet, phone)

**Current Tracking Methods:**
- [ ] What information is currently tracked?
- [ ] Where is it stored? (Excel, Yardi, paper, etc.)
- [ ] What workflows exist today?
- [ ] What are the pain points?

#### Workflow Identification

**Document Current Workflows:**

For a smaller site, you might have:
1. Unit becomes available
2. Request referral from coordinated entry
3. Receive referral
4. Contact household
5. Conduct showing
6. Intake documentation
7. Lease signing
8. Move-in
9. Ongoing case management
10. Move-out

**Identify Which Workflows to Include in App:**
- Start with top 3-5 most frequent workflows
- Save complex or rare workflows for later phases

---

### Step 2: Categorize Data Fields (3-5 days)

Break down data into categories to organize thinking.

#### Essential Categories

**1. Identification**
- What makes this unique? (Name, unit number, ID numbers)
- Never optional

**2. Contact Information**
- How do we reach them? (Phone, email, address)
- Important but sometimes unavailable

**3. Demographics**
- Who are they? (Age, household size, language)
- Varies by reporting requirements

**4. Program/Eligibility**
- What program are they in? (Unit type, income, disability)
- Critical for compliance

**5. Dates/Milestones**
- When did things happen? (Move-in, referral, lease signing)
- Essential for reporting

**6. Relationships**
- Who else is involved? (Service providers, household members)
- Can be added later

**7. Notes/Documentation**
- Free-form information
- Always useful

#### Field Priority Exercise

For each potential field, ask:
- **Must Have:** App won't function without it (examples: Name, Unit Number)
- **Should Have:** Important for operations (examples: Phone, Move-in Date)
- **Nice to Have:** Useful but not critical (examples: Preferred Name, Notes)
- **Future:** Not needed for MVP (examples: Household members, accounts)

**Recommendation for Smaller Sites:**
- Start with Must Have + Should Have only
- Launch with 10-15 fields per list
- Add Nice to Have based on user feedback
- Save Future for Phase 2+

---

### Step 3: Create Data Model Diagram (1-2 days)

Draw out how your lists relate to each other.

#### Minimal Core Model

```
Unit Types ←─ Units ←─ Referrals & Applicants
                ↓
              Current Residents
                ↓
           Archived Records
```

**Relationships:**
- Unit Types → Units: One-to-many (one type applies to many units)
- Units → Referrals: One-to-one (one referral per unit at a time)
- Units → Residents: One-to-one (one resident per unit at a time)
- Residents → Archives: One-to-one (resident becomes archive)

#### Optional Additions (Phase 2+)

```
Service Providers
       ↓
  Referrals & Residents (many-to-many)

Account Providers
       ↓
Account Information
       ↓
    Residents (one-to-many)

Household Members
       ↓
    Residents (many-to-one)
```

**Recommendation:**
Start with core model only. Add optional pieces after core is stable and users are comfortable.

---

## SharePoint Schema Design Best Practices

### Naming Conventions

#### Do's and Don'ts

**✅ DO:**
- Use clear, descriptive names
- Use title case (Unit Number, not unit number)
- Use full words (not abbreviations unless universally understood)
- Use spaces to separate words
- Keep field names under 32 characters if possible
- Be consistent (if you say "Date of Birth" don't also say "Birth Date")

**❌ DON'T:**
- Use special characters (`#`, `-`, `/`, `()`, `&`)
- Use ALL CAPS (except abbreviations like HMIS, HUD)
- Use abbreviations unless absolutely necessary
- Use ambiguous names (what does "Status" mean without context?)
- Use different names for the same concept across lists

#### Recommended Naming Patterns

**For Dates:**
- `[Action] Date` pattern: Move In Date, Referral Date, Close Out Date
- Avoid: "Date of Move In", "Date Moved In"

**For Yes/No Fields:**
- `Is [State]` pattern: Is Occupied, Is Active, Is Payable
- Or `Has [Thing]` pattern: Has Referral, Has Given Notice
- Avoid: "Occupied?" or "Referral?"

**For Lookup Fields:**
- Use the entity name: Unit Number, Service Provider, Organization
- If multiple lookups to same entity, add descriptor: Primary Provider, Secondary Provider

**For Choice Fields:**
- Use full descriptive name: Household Status, Unit Type, Referral Source
- Avoid: Type, Status, Source (too vague)

---

### Field Types Selection Guide

**When to Use Each Field Type:**

| Field Type | Use For | Power Apps Type | Example |
|------------|---------|-----------------|---------|
| Single line text | Short text, names, IDs | String | Preferred Name, HMIS ID |
| Multiple lines text | Notes, descriptions | String | Move Out Notes, Additional Info |
| Number | Counts, ages, currency | Number | Household Size, Monthly Income |
| Date and Time | Dates only | Date | Move In Date, Date of Birth |
| Choice (single) | One option from list | Object (Id, Value) | Household Status, Unit Type |
| Choice (multi) | Multiple options from list | Array of Objects | Disability, Benefits |
| Yes/No | Boolean flags | Boolean | Is Occupied, Has Referral |
| Person or Group | User assignment | Complex object | Assigned Coordinator |
| Lookup (single) | Reference to item in another list | Object (Id, Value) | Unit Number |
| Lookup (multi) | Multiple references | Array of Objects | Associated Providers |
| Calculated | Auto-calculated from other fields | Varies | Age (from DOB) |

**Critical Decisions:**

**Choice vs. Text:**
- Use **Choice** when: Limited set of options that won't change often
- Use **Text** when: Open-ended or frequently changing values
- **Remember:** If you'll need to copy this field between lists, both must be the same type

**Lookup vs. Text:**
- Use **Lookup** when: Referencing another list where you need to maintain the relationship
- Use **Text** when: Just storing a name/label without needing the relationship
- **Remember:** Lookups are harder to work with but maintain data integrity

**Single vs. Multi-select:**
- Use **Single** when: One and only one option applies
- Use **Multi** when: Multiple options can apply simultaneously
- **Remember:** Can't easily change between these after creation

---

### Required vs. Optional Fields

**Guidelines:**

**Make Required:**
- Unique identifiers (Name, Unit Number)
- Fields needed for the record to make sense
- Fields that drive critical workflows
- Compliance-required fields

**Keep Optional:**
- Contact information (might not be available yet)
- Demographic details (might be unknown)
- Notes and free-form fields
- Anything that might be filled in later

**Recommendation for Smaller Sites:**
- Minimize required fields (5-7 per list maximum)
- Make it easy to create records quickly
- Can always add data later

---

### Choice Field Options

**Keep Choice Lists SHORT:**
- 3-7 options is ideal
- 10 maximum before considering breakdown
- If you need more, use lookup to separate list

**Naming Choice Options:**

**✅ Good:**
- Referral, New
- Applicant, Pending Approval
- Occupant, Current
- Former Referral

**❌ Bad:**
- New (too vague)
- Approved (what's approved?)
- Current (current what?)

**Pattern for Status Fields:**
- `[Entity Type], [Status]`
- Groups related statuses together
- Makes filtering easier

**Order Matters:**
- Put most common options first
- Group related options together
- Consider workflow order

---

## Data Collection and Migration Strategy

### Understanding Your Existing Data Sources

Before creating SharePoint lists and importing data, you need to understand where your existing data lives and how to collect it efficiently.

#### Common Data Sources at CommonBond Sites

**1. PSH Master Unit List**
- **Location:** Supportive Housing SharePoint site → Documents
- **What It Contains:**
  - Complete inventory of all PSH units across all sites
  - Unit numbers, building addresses
  - Program types (LTH, CoC, Section 8, etc.)
  - Unit characteristics (bedrooms, accessibility features)
  - May include current occupancy status
- **How to Access:** Navigate to Supportive Housing SharePoint, find Documents library, look for "PSH Master Unit List" or similar
- **Export Method:** Download as Excel file

**2. Site-Specific Information Lists**
- **Location:** Individual site SharePoint sites or shared drives
- **What They May Contain:**
  - Current resident lists
  - Historical move-in dates
  - Contact information
  - Program-specific data
  - Service provider connections
- **Note:** Not all sites maintain these consistently
- **Export Method:** Download as Excel or copy from existing list

**3. Yardi Property Management System**
- **What It Contains:**
  - Official lease information
  - Move-in and move-out dates
  - Rent and payment history
  - Contact information
  - Household composition
  - Some demographic data
- **How to Access:** Yardi reports (may need to request from property management team)
- **Limitations:** Not all PSH data is in Yardi; may only have basic lease info
- **Export Method:** Standard Yardi reports exported to Excel

**4. Ashley's Move-In/Out Records**
- **Who:** Ashley (or your organization's equivalent housing coordinator)
- **What They Track:**
  - Historical move-ins and move-outs across sites
  - Dates and reasons for move-outs
  - May include referral sources
  - May track turnover patterns
- **Format:** Likely Excel or other tracking spreadsheet
- **How to Access:** Request directly from housing coordinator
- **Value:** May have historical context missing from other sources

**5. Case Management Records**
- **Location:** Various (case management systems, ETO, paper files, Excel)
- **What They Contain:**
  - Detailed resident information
  - Service provider connections
  - Demographics and eligibility information
  - Household members
  - Income and benefits
- **Access:** May need to coordinate with multiple case managers
- **Challenges:** Often scattered, not centralized

**6. Paper Files and Archives**
- **Location:** Site offices, central file storage
- **What They Contain:**
  - Intake paperwork
  - Lease documents
  - Historical referrals
  - Correspondence
- **When to Use:** When digital records are incomplete or missing
- **Limitation:** Time-intensive to extract data

---

### Data Collection Workflow

#### Phase 1: Inventory What You Need (Week 1)

**Step 1: Create Data Requirements List**

Based on your field dictionary, list everything you need to populate:

**For Units:**
- [ ] Unit numbers/identifiers
- [ ] Building/location information
- [ ] Unit characteristics (bedrooms, accessibility)
- [ ] Program types
- [ ] Current occupancy status
- [ ] Current resident (if occupied)

**For Current Residents:**
- [ ] Names
- [ ] Contact information (phone, email)
- [ ] Move-in dates
- [ ] Lease signing dates (if tracked)
- [ ] Demographics (if needed for your site)
- [ ] Service provider connections (if tracking)
- [ ] Unit assignments

**For Active Referrals:**
- [ ] Names
- [ ] Contact information
- [ ] Referral dates
- [ ] Status (where in process)
- [ ] Assigned units (if applicable)

---

**Step 2: Map Sources to Requirements**

Create a matrix showing where each piece of data lives:

| Data Element | Primary Source | Secondary Source | Backup Source |
|--------------|----------------|------------------|---------------|
| Unit numbers | PSH Master List | Site records | Physical walk-through |
| Unit types | PSH Master List | Site director | Program docs |
| Current residents | Yardi | Site list | Case management |
| Move-in dates | Yardi | Ashley's records | Lease files |
| Contact info | Case management | Yardi | Intake files |
| Service providers | Case managers | ETO | Referral docs |
| Referral status | Site coordinator | Recent emails | Coordinated entry |

This matrix helps you:
- Know where to look first
- Have backup if primary source is incomplete
- Identify gaps early

---

**Step 3: Assess Data Quality**

Before collecting everything, sample each source to understand quality:

**Questions to Answer:**
- [ ] Is the data current? (Last updated when?)
- [ ] Is it complete? (What percentage has blanks?)
- [ ] Is it accurate? (Spot-check against known facts)
- [ ] Is it consistent? (Same person listed different ways?)
- [ ] What cleanup will be needed?

**Common Data Quality Issues:**
- Names formatted differently (Last, First vs. First Last)
- Phone numbers formatted inconsistently
- Dates in different formats
- Yes/No represented as Y/N, True/False, 1/0
- Missing data in critical fields
- Duplicate entries

**Document Issues Found:**
Create a "Data Cleanup Required" list so you know what to fix before import.

---

#### Phase 2: Collect and Consolidate (Week 2)

**Step 1: Gather All Sources**

**For Each Data Source:**
1. Download/export to Excel format
2. Save with descriptive name: `[Source]_[Date]_[Content].xlsx`
   - Example: `PSH_Master_Unit_List_2025-10-24.xlsx`
   - Example: `Yardi_Current_Residents_2025-10-24.xlsx`
3. Store in organized folder structure:
   ```
   Data_Collection/
   ├── 1_Source_Files/
   │   ├── PSH_Master_Unit_List_2025-10-24.xlsx
   │   ├── Yardi_Current_Residents_2025-10-24.xlsx
   │   ├── Ashley_MoveIn_Records_2025-10-24.xlsx
   │   └── [other sources]
   ├── 2_Consolidated/
   │   ├── Units_Consolidated.xlsx
   │   ├── Residents_Consolidated.xlsx
   │   └── Referrals_Consolidated.xlsx
   └── 3_Import_Ready/
       ├── Units_ImportReady.xlsx
       ├── Residents_ImportReady.xlsx
       └── Referrals_ImportReady.xlsx
   ```

4. Mark each source with collection date and who provided it

---

**Step 2: Create Consolidation Templates**

**For Each SharePoint List (Units, Residents, Referrals):**

Create Excel file with:
- Column headers matching your SharePoint field names EXACTLY
- Column order matching your preference
- Formula columns for data transformation if needed
- Notes column for data quality issues

**Example: Units_Consolidated.xlsx**

| Unit | Building | Bedrooms | Unit Type | Is Occupied | Occupant Household | Notes |
|------|----------|----------|-----------|-------------|--------------------|-------|
| 101  | Main     | 1        | LTH Housing Support | TRUE | Smith, John | From PSH list |
| 102  | Main     | 1        | LTH Housing Support | FALSE | | Vacant per Yardi |

**Important Consolidation Rules:**

1. **Field Names:** Column headers must match SharePoint field names exactly
2. **Choice Fields:** Use exact text of choice options (case-sensitive)
3. **Yes/No Fields:** Use TRUE/FALSE (Excel boolean)
4. **Date Fields:** Use consistent date format (Excel recognizes dates automatically)
5. **Lookup Fields:** For now, just store the name/identifier; we'll handle lookups separately
6. **Multi-Select Fields:** Use semicolon separation: "Option1;Option2;Option3"

---

**Step 3: Merge Data from Multiple Sources**

**Process for Each Entity:**

**Example: Consolidating Resident Data**

1. Start with primary source (e.g., Yardi resident list)
2. Add columns from secondary sources:
   - VLOOKUP move-in dates from Ashley's records
   - VLOOKUP service providers from case management
   - VLOOKUP contact info from most recent source
3. Identify and resolve conflicts:
   - Different move-in dates? Check lease files
   - Different phone numbers? Use most recent
   - Missing data? Note for manual research
4. Create "Data Confidence" column:
   - "Verified": Confirmed from primary source
   - "Likely": From reliable secondary source
   - "Uncertain": Conflicting or old data
   - "Missing": No data found

**Excel Tips for Consolidation:**

```excel
# VLOOKUP to pull data from another sheet
=VLOOKUP(A2, AshleyData!A:D, 3, FALSE)

# IF to handle missing values
=IF(ISBLANK(B2), C2, B2)  // Use column C if column B is blank

# TEXT to standardize phone formats
=TEXT(A2, "(000) 000-0000")

# TRIM to remove extra spaces
=TRIM(A2)

# PROPER to standardize name capitalization
=PROPER(A2)
```

---

**Step 4: Data Cleanup**

**Common Cleanup Tasks:**

**1. Standardize Names**
- Decide on format: "Last, First" or "First Last"
- Apply consistently across all records
- Use Excel PROPER function to fix capitalization
- Remove extra spaces with TRIM function

**2. Standardize Phone Numbers**
- Choose format: (123) 456-7890 or 123-456-7890
- Use Excel TEXT function or Find & Replace
- Remove invalid numbers (000-000-0000, 123-456-7890)

**3. Standardize Dates**
- Ensure Excel recognizes as dates (not text)
- Choose display format (MM/DD/YYYY recommended)
- Handle partial dates (if you only have month/year, use 1st of month)

**4. Standardize Yes/No Fields**
- Convert all to TRUE/FALSE
- Find & Replace: Y → TRUE, N → FALSE, Yes → TRUE, etc.

**5. Standardize Choice Fields**
- Verify values match your SharePoint choice options exactly
- Find & Replace for common variations
- Example: "Housing Support" → "LTH Housing Support" (if that's your choice option)

**6. Handle Blanks**
- Decide which blanks are acceptable (optional fields)
- Identify which blanks need research
- Mark blanks that will be filled later
- Delete entirely blank rows

---

#### Phase 3: Prepare for Import (Week 3)

**Step 1: Create Import-Ready Files**

For each list, create final import file:

**File Requirements:**
- [ ] Column headers match SharePoint field names exactly
- [ ] All data cleaned and standardized
- [ ] No formula cells (paste values only)
- [ ] No extra columns (only fields that exist in SharePoint)
- [ ] No extra rows (no headers, summaries, or blanks)
- [ ] Choice field values match SharePoint options exactly
- [ ] Yes/No fields are TRUE/FALSE
- [ ] Dates are in Excel date format

**Special Handling for Lookup Fields:**

You CANNOT directly import lookup relationships via copy-paste. Strategy:

**Option 1: Import in Stages (Recommended)**
1. Import Units first (no lookups to other lists yet)
2. Import Residents and Referrals (without Unit Number lookup)
3. Manually connect lookups after import using SharePoint grid mode
4. Update in batches using SharePoint's lookup dropdown

**Option 2: Use Power Automate Flow (Advanced)**
- Create flow that reads Excel file
- Flow creates SharePoint items
- Flow handles lookup matching programmatically
- More complex but handles lookups automatically

**Option 3: Import with Text, Then Convert (Hybrid)**
1. Create temporary text field in SharePoint (e.g., "Unit Number Text")
2. Import with unit numbers as text
3. Use Power Automate or manual process to convert text to lookups
4. Delete temporary text field

**For Initial Implementation, Recommend Option 1:**
- Import entities without lookups
- Connect relationships manually in SharePoint
- Takes more time but ensures accuracy
- Easier to verify and fix issues

---

**Step 2: Create Import Verification Checklist**

Before importing, verify each file:

**Units File Check:**
- [ ] Unit column populated for every row
- [ ] Unit Types match SharePoint choice options
- [ ] TRUE/FALSE used for Yes/No fields
- [ ] No duplicate unit numbers
- [ ] All required fields populated
- [ ] Row count matches expected unit count

**Residents File Check:**
- [ ] Names populated for every row
- [ ] Household Status values match SharePoint choices
- [ ] Dates in date format
- [ ] No duplicate residents
- [ ] All required fields populated
- [ ] Can identify which unit each resident should link to (even if not imported yet)

**Referrals File Check:**
- [ ] Names populated for every row
- [ ] Household Status values match SharePoint choices
- [ ] Referral dates present
- [ ] Status values valid
- [ ] All required fields populated

---

**Step 3: Create Lookup Connection Plan**

Since lookup fields can't be copy-pasted directly, create a plan:

**Document to Create: Lookup_Connection_Plan.xlsx**

**Tab 1: Residents to Units**
| Resident Name | Should Link to Unit | Notes |
|---------------|---------------------|-------|
| Smith, John | 101 | Current resident |
| Jones, Mary | 205 | Current resident |

**Tab 2: Referrals to Units**
| Referral Name | Should Link to Unit | Notes |
|---------------|---------------------|-------|
| Williams, Bob | 103 | Applicant stage |

**Tab 3: Units to Residents** (reverse lookup)
| Unit | Should Link to Resident | Notes |
|------|------------------------|-------|
| 101 | Smith, John | Occupied |
| 103 | Williams, Bob | Has referral |

**Use This Document:**
- During manual lookup connection phase
- To verify all relationships established
- As checklist (mark off as completed)

---

### Data Migration Best Practices

#### Principle 1: Build Schema First, Then Import All Data at Once

**Why This Approach?**

SharePoint doesn't have a good way to import into an existing list (except grid mode copy-paste, which is useful but clunky). Therefore:

**✅ Recommended Approach:**
1. Design complete schema
2. Create all SharePoint lists with all fields
3. Test with 2-3 sample records
4. Verify everything works
5. Delete sample records
6. Import all data at once using grid mode copy-paste
7. Verify import successful
8. Connect lookup relationships
9. Begin using

**❌ Avoid:**
- Creating list with partial schema
- Importing some data
- Adding fields later
- Importing more data
- Result: Inconsistent data, hard to track what's imported

---

#### Principle 2: Verify Before You Import

**Create Test Import Process:**

1. **Test with 5 Records First**
   - Select 5 representative records from your import file
   - Copy to new "Test Import" sheet
   - Import these 5 into SharePoint
   - Verify all fields imported correctly
   - Verify formatting looks good
   - Verify choice fields work
   - Verify dates display correctly

2. **Fix Any Issues**
   - If something doesn't import right, fix in source Excel file
   - Delete test records from SharePoint
   - Test import again with fixed data
   - Repeat until 5 test records import perfectly

3. **Then Import All**
   - Once test successful, import full dataset
   - Import entity by entity (all units, then all residents, then all referrals)
   - Verify count (expected vs. actual)

---

#### Principle 3: Keep Source Files as Backup

**After Import:**
- [ ] Do NOT delete source Excel files
- [ ] Keep "Import Ready" files
- [ ] Add note: "Successfully imported on [date]"
- [ ] Store in organized archive
- [ ] You may need to reference or re-import if issues found

**Why:**
- If import partially fails, you can try again
- If you need to rebuild list, you have source data
- Serves as documentation of what was imported
- May need to verify data later

---

## SharePoint List Creation and Data Import

### SharePoint Grid Mode: Your Best Friend for Data Import

**What is Grid Mode?**

SharePoint lists have a "Grid view" that looks like Excel:
- Allows copy-paste from Excel directly into SharePoint
- Faster than creating records one-by-one
- Preserves formatting for most field types
- Works well for bulk data entry

**How to Access Grid Mode:**

1. Navigate to your SharePoint list
2. Click "Edit in grid view" button (looks like spreadsheet icon)
3. SharePoint displays as editable grid
4. You can paste multiple rows at once

---

### Step-by-Step Import Process

#### Step 1: Create SharePoint Lists with Complete Schema

**Before Any Import:**
- [ ] All lists created (Units, Referrals, Residents, etc.)
- [ ] All fields created with exact names
- [ ] All field types configured
- [ ] Choice options configured
- [ ] Required fields set
- [ ] Default values set (if any)

**Verify Schema:**
- [ ] Open each list
- [ ] Click "Add column" dropdown to see all fields
- [ ] Verify all expected fields present
- [ ] Verify field types correct (hover shows type)

---

#### Step 2: Import Units (Simplest - No Dependencies)

**Units are imported first because other entities reference them.**

**Process:**

1. Open Units list in SharePoint
2. Click "Edit in grid view"
3. Open your Units_ImportReady.xlsx file
4. Select all data rows (not headers)
5. Copy (Ctrl+C)
6. Click in first empty row of SharePoint grid
7. Paste (Ctrl+V)
8. SharePoint will paste all rows
9. Verify data looks correct
10. Click "Exit grid view"
11. Review records in normal view

**Common Grid Mode Import Issues:**

**Issue: Choice field shows as blank**
- Cause: Value doesn't exactly match a choice option
- Fix: Edit the cell, select from dropdown

**Issue: Date field shows as text**
- Cause: Excel date not recognized by SharePoint
- Fix: Re-format in Excel as date before import, or manually enter in SharePoint

**Issue: Yes/No field shows wrong value**
- Cause: Needs TRUE/FALSE not Yes/No
- Fix: Use Find & Replace in Excel before import

**Issue: Multi-line text field truncated**
- Cause: Grid mode limitation for very long text
- Fix: Edit record individually after import to add full text

---

**Post-Import Verification:**

- [ ] Count records: Does it match Excel row count?
- [ ] Spot-check 5-10 records: All fields imported?
- [ ] Check choice fields: Displaying correctly?
- [ ] Check Yes/No fields: Correct values?
- [ ] Check dates: Formatted correctly?
- [ ] Check for error indicators (red icons)

**If Issues Found:**
- Small issues: Fix manually in SharePoint
- Many issues: Delete all imported records, fix Excel file, re-import

---

#### Step 3: Import Residents and Referrals (Without Lookups Initially)

**Strategy:**
Import everything EXCEPT lookup fields that reference other lists.

**Process for Residents:**

1. In your Excel file, hide or delete lookup columns temporarily
   - Example: Hide "Unit Number" column (you'll connect this later)
   - Keep all other columns
2. Open Current Residents list in SharePoint
3. Edit in grid view
4. Copy and paste data from Excel
5. Exit grid view
6. Verify records imported

**Repeat for Referrals & Applicants**

**Post-Import Verification:**
- [ ] Correct count imported
- [ ] Names imported correctly
- [ ] Contact info imported correctly
- [ ] Dates imported correctly
- [ ] Status fields showing correct choices
- [ ] All non-lookup fields populated

---

#### Step 4: Connect Lookup Relationships

**Now connect residents to units, referrals to units, etc.**

**Method 1: One-by-One Manual (Small Datasets)**

1. Open Residents list
2. For each resident:
   - Click to edit record
   - In "Unit Number" field, type unit number and select from dropdown
   - Save
3. Repeat for all residents
4. Repeat for all referrals

**Method 2: Grid Mode Lookup Editing (Faster)**

1. Open Residents list
2. Edit in grid view
3. Have your "Lookup Connection Plan" document open
4. Click in Unit Number cell for first resident
5. Type unit number, select from dropdown
6. Press Tab to move to next resident
7. Repeat for all residents
8. Exit grid view

**Advantages of Grid Mode:**
- Stay in one view
- Quick Tab through records
- Can copy-paste if same unit for multiple residents (rare)

**Tips for Lookup Connection:**

- **Type-ahead search works:** Type "101" and SharePoint filters to units matching
- **Must select from dropdown:** Can't just type and leave, must select
- **Can clear lookup:** Delete value to set back to blank
- **Multi-select lookups:** Click field, checkboxes appear, select multiple

---

#### Step 5: Verify All Relationships Connected

**Verification Checklist:**

**Residents to Units:**
- [ ] Open Residents list
- [ ] Filter to show only records where Unit Number is blank
- [ ] Should be empty (unless you have unassigned residents)
- [ ] If records appear, connect them to units

**Units to Residents:**
- [ ] Open Units list
- [ ] For occupied units, verify Occupant Household field populated
- [ ] If blank but should have resident, edit and connect

**Test Bidirectional Lookups:**
- [ ] Open a unit record
- [ ] Verify resident name shows in Occupant Household field
- [ ] Click resident name (should navigate to resident record)
- [ ] Verify unit shows in resident's Unit Number field
- [ ] Click unit number (should navigate back to unit record)

**If Relationships Don't Match:**
- Check both directions: unit → resident AND resident → unit
- Both should point to each other
- If mismatch, fix manually

---

### Tips for Setting Up Bidirectional Lookup Relationships

**What Are Bidirectional Lookups?**

When two lists reference each other:
- Units list has "Occupant Household" (lookup to Residents)
- Residents list has "Unit Number" (lookup to Units)
- Ideally, these stay in sync: if Unit 101 points to John Smith, then John Smith should point to Unit 101

---

#### Creating Bidirectional Lookups in SharePoint

**There is NO automatic way to keep bidirectional lookups in sync.** You must maintain both sides manually or with automation.

**When Creating Lookup Fields:**

**Step 1: Create First Lookup**
1. In Units list, add column "Occupant Household"
2. Type: Lookup
3. Get information from: Current Residents
4. In this column: Title (or whichever field is resident name)
5. Allow blank values: Yes
6. Do NOT select "Enforce relationship behavior" for cascade deletes (too risky)

**Step 2: Create Second Lookup**
1. In Residents list, add column "Unit Number"
2. Type: Lookup
3. Get information from: Units
4. In this column: Title (unit number)
5. Allow blank values: Yes

**Step 3: Maintain Manually or with Automation**

SharePoint will NOT automatically keep these in sync. You have options:

---

#### Option 1: Manual Maintenance (Simplest)

**Process:**
- When you connect a resident to a unit in Power App workflow, manually update both:
  1. Set resident's Unit Number field
  2. Set unit's Occupant Household field
- This is what UP Clearview Convert workflow does:
  ```powerapps
  // Create resident with unit lookup
  Patch(Residents, ..., {'Unit Number': varUnit});

  // Update unit with resident lookup
  Patch(Units, varUnit, {'Occupant Household': {...}});
  ```

**Advantages:**
- No additional automation needed
- You control exactly when updates happen
- Clear logic in workflow code

**Disadvantages:**
- Must remember to update both sides
- Can get out of sync if only one side updated
- No automatic fix if they diverge

---

#### Option 2: Power Automate Flow (Automatic Sync)

**Create Flow: When Resident Updated, Update Unit**

**Trigger:** When an item is created or modified (Residents list)

**Condition:** If "Unit Number" is not empty

**Actions:**
1. Get the unit item (by ID from Unit Number lookup)
2. Update unit item:
   - Set Occupant Household to current resident
   - Set Is Occupied to Yes

**Create Mirror Flow: When Unit Updated, Update Resident**

**Trigger:** When an item is created or modified (Units list)

**Condition:** If "Occupant Household" changed

**Actions:**
1. If Occupant Household is not empty:
   - Get resident item
   - Update resident's Unit Number to current unit
2. If Occupant Household is empty:
   - Find any resident pointing to this unit
   - Clear their Unit Number

**Advantages:**
- Automatic synchronization
- Reduces chance of mismatch
- One source of truth (update either side, other follows)

**Disadvantages:**
- Requires Power Automate license/permissions
- Can trigger loops if not configured carefully (each update triggers the other)
- Must handle edge cases (what if multiple residents point to same unit?)
- More complex to troubleshoot

**Loop Prevention:**
- Add condition: Only run if "Modified By" is not "Power Automate"
- Or: Use separate field to track sync status
- Or: Add delay and check if update already happened

---

#### Option 3: Scheduled Cleanup Flow (Periodic Sync)

Instead of real-time sync, run cleanup flow periodically:

**Flow: Daily Bidirectional Lookup Verification**

**Trigger:** Recurrence (daily at 2 AM)

**Actions:**
1. Get all residents where Unit Number is not blank
2. For each resident:
   - Get associated unit
   - If unit's Occupant Household doesn't match this resident:
     - Update unit's Occupant Household
3. Get all units where Occupant Household is not blank
4. For each unit:
   - Get associated resident
   - If resident's Unit Number doesn't match this unit:
     - Update resident's Unit Number

**Advantages:**
- Fixes any mismatches that occur
- Doesn't risk update loops
- Less complex than real-time sync
- Good "safety net"

**Disadvantages:**
- Not real-time (up to 24-hour delay)
- May override intentional changes

---

#### Recommended Approach for New Implementation

**For Initial Launch:**

**Use Manual Maintenance** (Option 1)
- Keep bidirectional update logic in Power App workflows
- Simpler to implement and debug
- One less thing to troubleshoot during launch
- You control exactly what happens

**After Stable (3-6 Months):**

**Add Scheduled Cleanup Flow** (Option 3)
- Runs nightly to catch any mismatches
- Acts as safety net
- Logs discrepancies for review
- Minimal risk of causing new issues

**Only If Needed:**

**Implement Real-Time Sync** (Option 2)
- Only if you find frequent manual errors
- Only if you have resources to build and maintain
- Test extensively before deploying
- Include loop prevention logic

---

### Common Data Import Troubleshooting

#### Problem: "Value does not match one of the choices"

**Cause:** Choice field value in Excel doesn't exactly match SharePoint choice option

**Solutions:**
1. Open SharePoint list settings → Click field name → See exact choice options
2. In Excel, use Find & Replace to match exactly (case-sensitive)
3. Re-import, or manually select from dropdown in SharePoint

---

#### Problem: Dates Import as Text or Wrong Format

**Cause:** Excel date format not recognized by SharePoint

**Solutions:**
1. In Excel, select date column → Format Cells → Date → Choose standard format
2. Verify Excel recognizes as date (should be right-aligned in cell)
3. Copy-paste just the date column in grid mode
4. Or manually edit dates in SharePoint

---

#### Problem: Large Paste Fails or Times Out

**Cause:** Too many rows to paste at once

**Solutions:**
1. Break import into batches (50-100 rows at a time)
2. Paste first 100 rows, exit grid view, verify
3. Re-enter grid view, paste next 100 rows
4. Repeat until all imported

---

#### Problem: Multi-Line Text Truncated

**Cause:** Grid mode doesn't handle very long text well

**Solutions:**
1. Import record with short placeholder text
2. After import, edit record individually
3. Paste full text into multi-line field in edit form
4. Save

---

#### Problem: Lookup Field Won't Accept Value

**Cause:** Referenced item doesn't exist yet, or ID mismatch

**Solutions:**
1. Verify the item you're trying to link to exists in target list
2. Type the display value (e.g., unit number) and select from dropdown
3. Can't type ID directly; must select from lookup list
4. If item truly doesn't exist, create it first

---

#### Problem: Import Duplicates Records

**Cause:** Pasted multiple times, or pasted over existing records

**Solutions:**
1. Delete duplicates:
   - Sort list by Created date
   - Select duplicate records (checkbox)
   - Delete selected items
2. For large duplicates, use Power Automate flow to find and delete duplicates

---

### Post-Import Data Quality Check

After importing all data, run quality checks:

**Completeness Check:**
- [ ] All units imported (count matches expected)
- [ ] All residents imported (count matches expected)
- [ ] All referrals imported (count matches expected)
- [ ] Required fields populated for all records
- [ ] No unexpected blank fields

**Relationship Check:**
- [ ] All residents connected to units
- [ ] Units reflect resident occupancy status
- [ ] Referrals connected to units where applicable
- [ ] Bidirectional lookups match

**Data Integrity Check:**
- [ ] No duplicate unit numbers
- [ ] No duplicate residents (check for same name)
- [ ] Dates are reasonable (no future dates, no dates from 1900)
- [ ] Phone numbers formatted consistently
- [ ] Choice fields showing correct values

**Spot Check:**
- [ ] Pick 10 random records
- [ ] Compare to source data
- [ ] Verify all fields transferred correctly
- [ ] Verify formatting looks good

**User Acceptance:**
- [ ] Show imported data to site staff
- [ ] Have them verify accuracy
- [ ] Fix any errors they identify
- [ ] Get sign-off before going live

---

## Site Size Considerations

### Small Site (5-20 units)

**Advantages:**
- Easier to implement
- Fewer users to train
- Simpler workflows
- Less data to migrate
- Faster to test

**Recommended Approach:**
- Start with absolute minimum lists: Units, Referrals, Residents
- Skip Service Providers initially (just use text field)
- Skip Household Members initially (note in household size)
- Focus on core browse/view/edit functionality
- Add one workflow at a time

**Timeline:**
- Planning: 1 week
- SharePoint setup: 2-3 days
- App building: 2-3 weeks
- Testing: 1 week
- **Total: 4-6 weeks**

**Recommended MVP Features:**
1. Browse units, referrals, residents
2. View details
3. Edit records
4. Receive referral
5. Convert to resident (simplified)
6. Exit resident

**Skip for MVP:**
- Phone screening workflow
- Intake workflow
- Account management
- Service provider associations
- Complex reporting

---

### Medium Site (20-50 units)

**Considerations:**
- More complex workflows
- Multiple staff members
- May have multiple programs
- More data history to consider

**Recommended Approach:**
- Include Service Providers list from start
- Include basic workflow automation
- Consider phased rollout by unit type
- Plan for more extensive testing

**Timeline:**
- Planning: 2 weeks
- SharePoint setup: 1 week
- App building: 4-6 weeks
- Testing: 2 weeks
- **Total: 9-11 weeks**

**Recommended MVP Features:**
- Everything from Small Site
- Plus: Service provider associations
- Plus: Phone screening workflow
- Plus: Basic intake tracking

---

### Large Site (50+ units) - Like Upper Post

**Considerations:**
- Complex multi-program structure
- Many staff users
- Extensive workflows
- Integration with other systems
- Higher stakes if something goes wrong

**Recommended Approach:**
- Full planning process (2-4 weeks)
- Pilot with subset of units first
- Phased rollout by program type
- Extensive UAT with multiple roles
- Plan for ongoing support

**Timeline:**
- Planning: 3-4 weeks
- SharePoint setup: 1-2 weeks
- App building: 8-12 weeks
- Testing: 3-4 weeks
- Pilot: 2-4 weeks
- Full rollout: 2-4 weeks
- **Total: 18-26 weeks**

**Recommendation:**
Consider starting with a smaller site to work out the kinks, then bring refined version to large site.

---

## Step-by-Step Adaptation Process

### Phase 0: Planning (Before Any Technical Work)

#### Week 1: Discovery

**Day 1-2: Stakeholder Interviews**
- [ ] Interview site director
- [ ] Interview case managers
- [ ] Interview housing coordinators
- [ ] Interview anyone who tracks data currently
- [ ] Document current pain points
- [ ] Document wish list features

**Day 3-4: Process Observation**
- [ ] Shadow staff during referral intake
- [ ] Observe move-in process
- [ ] Watch how they track information now
- [ ] Note what information they look up frequently
- [ ] Document actual workflows (not just described workflows)

**Day 5: Priority Setting**
- [ ] Compile findings
- [ ] Categorize features (must/should/nice/future)
- [ ] Get agreement on MVP scope
- [ ] Set timeline expectations

#### Week 2: Schema Design

**Day 1-2: Field Inventory**
- [ ] List every piece of data currently tracked
- [ ] Categorize by entity (Unit, Referral, Resident)
- [ ] Identify which fields are must-have
- [ ] Map relationships between entities

**Day 3: Field Design**
- [ ] Name all fields (review naming best practices)
- [ ] Choose field types
- [ ] Decide required vs. optional
- [ ] Document choice options
- [ ] Create field dictionary document

**Day 4: Schema Review**
- [ ] Review with stakeholders
- [ ] Verify all needs captured
- [ ] Verify field names are clear
- [ ] Get sign-off on schema
- [ ] Document that changes after this point cost significant time

**Day 5: SharePoint Planning**
- [ ] Identify SharePoint site to use
- [ ] Verify permissions
- [ ] Plan list structure
- [ ] Prepare for list creation

---

### Phase 1: SharePoint Setup (Week 3)

#### Create Lists (Do in This Order)

**Step 1: Unit Types List** (Day 1, 1-2 hours)

This list has no dependencies.

**Fields to Create:**
```
- Title → "Unit Type" (rename default field)
- Program (Choice): LTH Housing Support, Section 8 PBV, Ramsey CoC, etc.
- Referral Source (Choice): HUD-VASH, Coordinated Entry, etc.
- Income Limit (Currency)
- Income Basis (Choice): AMI 30%, AMI 50%, etc.
- Minimum Occupancy (Number)
- Maximum Occupancy (Number)
- Homelessness Required (Yes/No)
- Disability Required (Yes/No)
- Notes (Multiple lines)
```

**For Smaller Sites:**
Consider if you even need this list, or if all units are same type. If all same, skip this list and hardcode values.

- [ ] Create list
- [ ] Add all fields with exact names
- [ ] Set required fields (only Title)
- [ ] Create 1-3 unit type records with sample data
- [ ] Verify all fields display correctly

---

**Step 2: Units List** (Day 1, 2-3 hours)

Depends on Unit Types (if used).

**Fields to Create:**
```
- Title → "Unit" (unit number, like "101")
- Building (Single line text)
- Floor (Single line text)
- Bedrooms (Number)
- Bathrooms (Number)
- ADA Accessible (Yes/No)
- Unit Type (Lookup to Unit Types list) - OR Choice if no Unit Types list
- Is Occupied (Yes/No)
- Has Referral (Yes/No)
- Has Given Notice (Yes/No)
- Is Offline (Yes/No)
- Date Available (Date)
- Date Referral Requested (Date)
- Occupant Household (Lookup to Current Residents) - create later
- Referral Household (Lookup to Referrals & Applicants) - create later
- Notes (Multiple lines)
```

**For Smaller Sites:**
Skip: Building, Floor, Bathrooms, Has Given Notice, Is Offline, Date Referral Requested

- [ ] Create list
- [ ] Add all fields EXCEPT the two lookup fields to other lists
- [ ] Create 5-10 unit records with realistic data
- [ ] Verify everything displays correctly

---

**Step 3: Referrals & Applicants List** (Day 2, 3-4 hours)

Depends on Units.

**Core Fields:**
```
- Title → "Head of Household"
- Preferred Name (Single line text)
- Preferred Language (Single line text)
- Phone (Single line text)
- Email (Single line text)
- Date of Birth (Date)
- Household Size (Number)
- Household Status (Choice):
  * Referral, New
  * Referral, Contacted
  * Referral, Phone Screening Complete
  * Applicant, Intake Scheduled
  * Applicant, Pending Approval
  * Applicant, Approved
  * Former Referral
  * Former Applicant
- Unit Number (Lookup to Units)
- Date Referred (Date)
- Intake Completed (Date)
- Close Out Date (Date) - NO HYPHEN!
- Close Out Reason (Choice):
  * Declined
  * No Contact
  * Housed Elsewhere
  * Ineligible
  * Other
- Close Out Notes (Multiple lines)
- Notes (Multiple lines)
```

**Additional Fields for Larger Sites:**
```
- Veteran Status (Choice or Text)
- Disability (Choice multi-select): Physical, Developmental, Mental Health, Substance Use
- Benefits (Choice multi-select): SSI, SSDI, GA, SNAP, etc.
- Monthly Income (Currency)
- Other Source of Income (Single line text)
- HMIS Record Number (Single line text)
- County Case Number (Single line text)
- Associated Providers (Lookup multi-select to Service Providers) - if using
```

**For Smaller Sites:**
Start with just Core Fields. Can add Additional Fields later if needed.

- [ ] Create list
- [ ] Add core fields with exact names
- [ ] Add additional fields if appropriate for site size
- [ ] Create 2-5 sample referral records
- [ ] Verify all fields work correctly

---

**Step 4: Current Residents List** (Day 3, 3-4 hours)

Depends on Units.

**Important:** Field types and names should MATCH Referrals list for any field that will be copied during conversion!

**Core Fields:**
```
- Title → "Head of Household"
- Preferred Name (Single line text) ← SAME as Referrals
- Preferred Language (Single line text) ← SAME
- Phone (Single line text) ← SAME
- Email (Single line text) ← SAME
- Date of Birth (Date) ← SAME
- Household Size (Number) ← SAME
- Household Status (Choice):
  * Occupant, Current
  * Occupant, Given Notice
  * Former Occupant
- Unit Number (Lookup to Units) ← SAME
- Move In Date (Date)
- Lease Signing Date (Date)
- Expected Move Out (Date)
- Notes (Multiple lines)
```

**Additional Fields:**
```
- Veteran Status (SAME type as Referrals!)
- Disability (SAME type as Referrals!)
- Benefits (SAME type as Referrals!)
- Monthly Income (Currency) ← SAME
- Other Source of Income (Single line text) ← SAME
- HMIS Record Number (Single line text) ← SAME
- County Case Number (Single line text) ← SAME
- Associated Providers (Lookup multi-select) ← SAME if used
```

**CRITICAL VERIFICATION:**
- [ ] Open both lists side-by-side in SharePoint
- [ ] For every field that appears in both, verify:
  - [ ] Field name is EXACTLY the same
  - [ ] Field TYPE is EXACTLY the same
  - [ ] If Choice, options are the same (or superset)
  - [ ] If Lookup, points to same list
- [ ] Document any intentional differences

- [ ] Create list
- [ ] Add all fields, paying careful attention to type matching
- [ ] Create 2-5 sample resident records
- [ ] Verify all fields work

---

**Step 5: Add Cross-List Lookups** (Day 3, 1 hour)

Now go back and add the lookup fields that reference other lists.

**Back to Units List:**
- [ ] Add "Occupant Household" (Lookup to Current Residents, allow blank)
- [ ] Add "Referral Household" (Lookup to Referrals & Applicants, allow blank)

**Test the Relationships:**
- [ ] Select a unit
- [ ] In Occupant Household, select a resident → verify link works
- [ ] In Referral Household, select a referral → verify link works
- [ ] Verify you can clear the lookups (set back to blank)

---

**Step 6: Optional Lists** (Day 4-5, if needed)

**Service Providers** (if including)
```
- Title → "Service Provider" (name)
- Organization (Single line text) - or Lookup if creating Agencies list
- Role (Single line text)
- Program (Single line text)
- Phone (Single line text)
- Email (Single line text)
- Is Active (Yes/No)
- Notes (Multiple lines)
```

**Agencies** (if including)
```
- Title → "Agency Name"
- Phone (Single line text)
- Email (Single line text)
- Website (Hyperlink)
- Services Provided (Multiple lines)
- Notes (Multiple lines)
```

**Archived Records** (if including)
```
- Title → "Record Name" (auto-generate: "Name - Archive Date")
- Record Type (Choice): Referral, Resident
- Linked Referral (Lookup to Referrals, allow blank)
- Linked Resident (Lookup to Residents, allow blank)
- Unit (Lookup to Units, allow blank)
- Archive Date (Date)
- Archive Reason (Choice): Moved Out, Declined, Housed Elsewhere, etc.
- Outcome Notes (Multiple lines)
```

**For Smaller Sites:**
Skip these lists for MVP. Add in Phase 2 if needed.

---

**Step 7: Load Sample Data** (Day 5, 2-3 hours)

Create realistic sample data to test with:

**Unit Types:** 1-3 types
**Units:** 5-10 units representing your site
**Referrals:** 3-5 active referrals at various stages
**Residents:** 3-5 current residents
**Service Providers:** 3-5 providers (if using)

**Make Data Realistic:**
- Use actual unit numbers
- Use realistic names (can be fake but realistic)
- Use variety of statuses
- Link records appropriately (referrals to units, residents to units)

**Purpose of Sample Data:**
- Test Power App with real-looking data
- Train staff with familiar-looking records
- Verify all field types work as expected
- Test formulas and filters

- [ ] Create sample data
- [ ] Verify all relationships work
- [ ] Verify all choice fields display correctly
- [ ] Document which records are test data (prefix with "TEST - ")

---

### Phase 2: Power App Building (Weeks 4-8 for Small Site)

#### Progressive Build Approach

**Important:** Build in this order, testing each piece before moving on.

---

**Milestone 1: Browse Screens (Week 4)**

**Day 1-2: UnitBrowseScreen**
- [ ] Create new canvas app from blank
- [ ] Add Units as data source
- [ ] Create gallery showing all units
- [ ] Display: Unit, Unit Type, Occupancy Status
- [ ] Add color coding by unit type (ColorValue function)
- [ ] Add toggle for "Show Vacancies Only"
- [ ] Test gallery loads and filter works

**Day 3: RNABrowseScreen**
- [ ] Create new screen
- [ ] Add gallery for Referrals & Applicants
- [ ] Display: Name, Unit, Status
- [ ] Filter out "Former" records
- [ ] Add color coding
- [ ] Sort by Unit Number
- [ ] Test display

**Day 4: ResBrowseScreen**
- [ ] Create new screen
- [ ] Add gallery for Current Residents
- [ ] Display: Name, Unit, Unit Type
- [ ] Filter out "Former" records
- [ ] Add color coding
- [ ] Test display

**Day 5: Navigation Between Browse Screens**
- [ ] Add top navigation bar
- [ ] Add buttons for Units, Referrals, Residents
- [ ] Test navigation works smoothly
- [ ] **Milestone 1 Complete: Can browse all data**

---

**Milestone 2: Detail Screens with View Mode (Week 5)**

**Day 1: UnitInfoScreen**
- [ ] Create new screen
- [ ] Add FormViewer or Labels to display unit details
- [ ] Show all unit fields
- [ ] Display linked unit type details
- [ ] Add "Back" button to UnitBrowseScreen
- [ ] Test navigation and display

**Connect to Browse:**
- [ ] UnitBrowseScreen gallery OnSelect: Set varSelectedUnit, Navigate
- [ ] Test: Click unit in browse, see details on info screen

**Day 2-3: RNAInfoScreen**
- [ ] Create new screen
- [ ] Add FormViewer for referral details
- [ ] Display all referral fields
- [ ] Show linked unit information
- [ ] Add "Back" button
- [ ] Connect gallery OnSelect
- [ ] Test navigation and display

**Day 4-5: ResInfoScreen**
- [ ] Create new screen
- [ ] Add FormViewer for resident details
- [ ] Display all resident fields
- [ ] Show linked unit information
- [ ] Add "Back" button
- [ ] Connect gallery OnSelect
- [ ] Test navigation and display

**Milestone 2 Complete: Can browse and view details**

---

**Milestone 3: Edit Mode (Week 6)**

**Day 1-2: Enable Editing on UnitInfoScreen**
- [ ] Replace FormViewer with Form control
- [ ] Set DataSource = Units
- [ ] Set Item = varSelectedUnit
- [ ] Add Edit/View mode toggle
- [ ] Add Save button (SubmitForm)
- [ ] Add Cancel button (ResetForm, ViewForm)
- [ ] Test editing and saving

**Day 3: Enable Editing on RNAInfoScreen**
- [ ] Convert to editable form
- [ ] Add edit/save/cancel controls
- [ ] Test editing referrals

**Day 4: Enable Editing on ResInfoScreen**
- [ ] Convert to editable form
- [ ] Add edit/save/cancel controls
- [ ] Test editing residents

**Day 5: Testing and Refinement**
- [ ] Test all edit operations
- [ ] Verify saves persist to SharePoint
- [ ] Test cancel functionality
- [ ] Add OnSuccess handlers with Notify
- [ ] **Milestone 3 Complete: Can browse, view, and edit**

---

**Milestone 4: First Workflow - Receive Referral (Week 7)**

**Day 1-2: Build Receive Referral Form**
- [ ] On UnitInfoScreen, add "Receive Referral" button
- [ ] Button visible only if unit vacant and no referral
- [ ] Create modal container with form
- [ ] Form creates new Referrals & Applicants record
- [ ] Pre-populate: Unit Number, Date Referred
- [ ] User enters: Head of Household name
- [ ] OnSuccess: Update unit's Referral Household field

**Formula (simplified):**
```powerapps
OnSuccess: |-
  =Patch(
      Units,
      varSelectedUnit,
      {
          'Referral Household': {
              '@odata.type': "#Microsoft.Azure.Connectors.SharePoint.SPListExpandedReference",
              Id: ReferralForm.LastSubmit.ID,
              Value: ReferralForm.LastSubmit.Title
          },
          'Has Referral': true
      }
  );
  Set(varShowReceiveForm, false);
  Refresh(Units);
  Notify("Referral created successfully.", NotificationType.Success)
```

**Day 3: Test Receive Referral Workflow**
- [ ] Navigate to vacant unit
- [ ] Click Receive Referral
- [ ] Enter name
- [ ] Save
- [ ] Verify referral created
- [ ] Verify unit updated
- [ ] Verify referral appears in RNABrowseScreen

**Day 4-5: Refinement**
- [ ] Test edge cases
- [ ] Improve form layout
- [ ] Add validation
- [ ] **Milestone 4 Complete: First workflow functioning**

---

**Milestone 5: Convert to Resident Workflow (Week 8)**

This is the most complex workflow. Use progressive debugging approach.

**Day 1: Create scrConvert Screen**
- [ ] Create new screen
- [ ] Add FormViewer showing referral data (for review)
- [ ] Add input fields: Lease Date, Coordinator (optional)
- [ ] Add "Convert to Resident" button
- [ ] Add navigation from RNAInfoScreen

**Day 2-3: Build Convert Logic Progressively**

**Start Minimal:**
```powerapps
OnSelect: |-
  =Set(
      varNewResident,
      Patch(
          'Current Residents',
          Defaults('Current Residents'),
          {
              Title: varSelectedReferral.Title,
              'Household Status': {Value: "Occupant, Current"}
          }
      )
  )
```

**Test:** Does it create resident with name?

**Add Simple Fields One at a Time:**
```powerapps
{
    Title: varSelectedReferral.Title,
    'Household Status': {Value: "Occupant, Current"},
    'Preferred Name': varSelectedReferral.'Preferred Name',  // Add
    Phone: varSelectedReferral.Phone  // Test again
}
```

**Add Lookup Fields:**
```powerapps
{
    // ... previous fields ...
    'Unit Number': varSelectedReferral.'Unit Number'  // Entire object
}
```

**Complete Formula:**
```powerapps
OnSelect: |-
  =// Create resident
  Set(
      varNewResident,
      Patch(
          'Current Residents',
          Defaults('Current Residents'),
          {
              Title: varSelectedReferral.Title,
              'Household Status': {Value: "Occupant, Current"},
              'Preferred Name': varSelectedReferral.'Preferred Name',
              Phone: varSelectedReferral.Phone,
              Email: varSelectedReferral.Email,
              'Date of Birth': varSelectedReferral.'Date of Birth',
              'Household Size': varSelectedReferral.'Household Size',
              'Unit Number': varSelectedReferral.'Unit Number',
              'Lease Signing Date': DatePicker1.SelectedDate
              // Add other fields that match between lists
          }
      )
  );

  // Update unit
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
          'Has Referral': false,
          'Is Occupied': true
      }
  );

  // Delete referral
  RemoveIf('Referrals & Applicants', ID = varSelectedReferral.ID);

  // Navigate to new resident
  Set(varSelectedResident, varNewResident);
  Notify("Resident created successfully.", NotificationType.Success);
  Navigate(ResInfoScreen, ScreenTransition.None)
```

**Day 4: Test Convert Workflow**
- [ ] Create test referral
- [ ] Convert to resident
- [ ] Verify resident created with all data
- [ ] Verify unit updated
- [ ] Verify referral deleted
- [ ] Test multiple times with different data

**Day 5: Refinement and Documentation**
- [ ] Test edge cases (missing data, etc.)
- [ ] Document any field-specific handling needed
- [ ] **Milestone 5 Complete: Core workflows functioning**

---

**Week 8+: Additional Features**

Based on site needs and priorities:
- [ ] Close Referral workflow
- [ ] Exit Resident workflow
- [ ] Service Provider associations (if using)
- [ ] Additional screens as needed

**For Smaller Sites:**
Stop here for MVP. Launch with core functionality. Add features based on actual usage and feedback.

---

### Phase 3: Testing (1-2 weeks)

Use the Testing Guide to systematically test:

**Week 9: Unit Testing**
- [ ] Test all browse screens
- [ ] Test all info screens
- [ ] Test all edit operations
- [ ] Document bugs found

**Week 10: Workflow and Integration Testing**
- [ ] Test Receive Referral end-to-end
- [ ] Test Convert to Resident end-to-end
- [ ] Test complete lifecycle: Vacant → Referral → Resident
- [ ] Test edge cases
- [ ] Fix critical bugs

**Week 10 (second half): UAT**
- [ ] Staff test with realistic scenarios
- [ ] Collect feedback
- [ ] Make priority fixes
- [ ] Get sign-off

---

### Phase 4: Deployment and Training

**Week 11: Soft Launch**
- [ ] Launch to small group of staff
- [ ] Monitor usage closely
- [ ] Fix issues quickly
- [ ] Gather feedback

**Week 12: Full Launch**
- [ ] Train all staff
- [ ] Migrate existing data (if needed)
- [ ] Go live
- [ ] Provide support

---

## Field Customization by Site

### Understanding What's Universal vs. Site-Specific

#### Universal Fields (Include at Every Site)

**Units:**
- Unit (number/identifier)
- Is Occupied
- Has Referral

**Referrals & Residents:**
- Name (Title)
- Phone
- Unit Number
- Household Status
- Date Referred / Move In Date

**These fields are core to the app functioning. Don't skip them.**

---

#### Common But Not Universal

**May need at larger sites:**
- Veteran Status
- Disability
- Benefits
- HMIS Record Number
- Service Provider associations

**May not need at smaller sites:**
- These can be tracked elsewhere
- Consider adding later if needed
- Don't overcomplicate MVP

---

#### Site-Specific Fields

**Examples of fields specific to certain sites/programs:**
- Professional Statement of Need (LTH only)
- CoC Self Certification Months (CoC only)
- Housing Support Vendor ID (Housing Support only)
- VA Case Number (HUD-VASH only)

**Approach:**
1. Identify which programs your site has
2. Research requirements for those programs
3. Add only fields you actually need for compliance
4. Group program-specific fields together in form layout

---

#### Field Customization Matrix

Use this matrix to decide what to include:

| Field | All Sites | Small Sites | Medium/Large | Program-Specific |
|-------|-----------|-------------|--------------|------------------|
| Name, Unit | ✅ | ✅ | ✅ | - |
| Phone, Email | ✅ | ✅ | ✅ | - |
| Household Size | ✅ | ⚠️ | ✅ | - |
| Date of Birth | ⚠️ | ❌ | ✅ | - |
| Preferred Name | ⚠️ | ❌ | ✅ | - |
| Preferred Language | ⚠️ | ❌ | ✅ | - |
| Veteran Status | ⚠️ | ❌ | ⚠️ | HUD-VASH |
| Disability | ⚠️ | ❌ | ⚠️ | Some programs |
| Benefits | ⚠️ | ❌ | ✅ | Case management |
| Income | ⚠️ | ❌ | ✅ | Compliance |
| HMIS Number | ⚠️ | ❌ | ✅ | Reporting |
| County Case Number | ⚠️ | ❌ | ✅ | County programs |
| Service Providers | ❌ | ❌ | ✅ | Coordination |
| Household Members | ❌ | ❌ | ⚠️ | Family units |

**Legend:**
- ✅ Recommended
- ⚠️ Consider (depends on needs)
- ❌ Skip for MVP

---

### Customization Scenarios

#### Scenario 1: Small Site, Singles Only (10 units)

**Include:**
- Units: Unit, Unit Type, Is Occupied, Has Referral, Notes
- Referrals: Name, Phone, Unit Number, Status, Date Referred, Notes
- Residents: Name, Phone, Unit Number, Status, Move In Date, Notes

**Skip:**
- Household Size (always 1)
- Household Members
- Disability, Benefits, Income (not tracked at this site)
- Service Providers (just note in text field if needed)
- All program-specific fields

**Result:**
- 5-7 fields per list
- Very quick to set up
- Easy for staff to use
- Can add fields later if needs evolve

---

#### Scenario 2: Medium Site, Mixed Programs (30 units)

**Include:**
- All Universal fields
- Household Size (have families)
- Preferred Name, Language (diverse population)
- Veteran Status (have HUD-VASH units)
- Basic Benefits tracking
- Service Providers list (many external partners)

**Skip:**
- Complex disability tracking (note in text instead)
- Income tracking (not needed for their programs)
- Household Members detailed tracking (just note count and ages in text)
- Account management (no bill pay)

**Result:**
- 12-15 fields per list
- Balances functionality with simplicity
- Room to grow

---

#### Scenario 3: Large Site, Multiple Programs - Like Upper Post

**Include:**
- Everything from Medium Site
- Plus: Detailed disability tracking (multi-select)
- Plus: Benefits tracking
- Plus: Income fields
- Plus: HMIS and County Case Numbers
- Plus: Household Members list
- Plus: Account management (for Housing Support)
- Plus: All program-specific fields

**Result:**
- 20-25 fields per list
- Comprehensive tracking
- Supports complex workflows
- Requires more training

---

## Testing Checklist for New Sites

### Pre-Launch Testing Checklist

**SharePoint Verification:**
- [ ] All lists created
- [ ] All fields created with correct names (no special characters)
- [ ] All field types correct
- [ ] Required fields set appropriately
- [ ] Choice options configured
- [ ] Lookup relationships working
- [ ] Sample data loaded
- [ ] Permissions set correctly

**Power App Basic Functionality:**
- [ ] App opens without errors
- [ ] All screens accessible
- [ ] Navigation works (forward and back)
- [ ] Variables set correctly on screen load
- [ ] Galleries display data
- [ ] Forms display data
- [ ] Color coding works

**Data Operations:**
- [ ] Can view records (all entity types)
- [ ] Can create records (all entity types)
- [ ] Can edit records (all entity types)
- [ ] Saves persist to SharePoint
- [ ] Cancel/reset works
- [ ] Validation works on required fields

**Workflows (Test Each):**
- [ ] Receive Referral: Creates referral, updates unit
- [ ] Convert to Resident: Creates resident, updates unit, deletes referral
- [ ] (Any other workflows): Test end-to-end

**Integration Testing:**
- [ ] Complete lifecycle: Vacant → Referral → Resident → Exit
- [ ] Referral → Close (if implemented)
- [ ] Data consistency across related records

**Edge Cases:**
- [ ] Blank optional fields
- [ ] Maximum field lengths
- [ ] Special characters in names
- [ ] Date boundaries

**User Acceptance:**
- [ ] Staff can complete tasks without help
- [ ] Staff understand the interface
- [ ] Staff find it easier than current method
- [ ] Staff sign off

---

## Common Pitfalls and Solutions

### Pitfall 1: "Just One More Field..."

**The Problem:**
- Stakeholders keep requesting additional fields
- Scope creeps during development
- Launch date keeps slipping
- App becomes overwhelming

**The Solution:**
- Set firm MVP scope at start
- Document "Phase 2" features separately
- Use "must/should/nice/future" framework
- Launch with minimum, add based on usage
- Say: "That's a great idea for Phase 2 once we're comfortable with the basics."

---

### Pitfall 2: Over-Engineering for Edge Cases

**The Problem:**
- Trying to handle every possible scenario
- Adding complex logic for rare situations
- App becomes fragile and hard to maintain

**The Solution:**
- Focus on the 80% use case
- Document workarounds for rare situations
- Let some things be manual
- Remember: Perfect is the enemy of done
- Staff can always add a note for unusual cases

---

### Pitfall 3: Assuming Staff Know SharePoint/Power Apps

**The Problem:**
- Building with tech-savvy workflow in mind
- Not enough training
- Staff revert to old methods

**The Solution:**
- Design for least technical user
- Make workflows obvious
- Add instructions and tooltips
- Provide hands-on training
- Offer ongoing support
- Don't assume anything is "obvious"

---

### Pitfall 4: Not Testing with Real Data

**The Problem:**
- Testing with clean, perfect sample data
- Real data has blanks, errors, inconsistencies
- App breaks with real use

**The Solution:**
- Create messy test data
- Test with blank fields
- Test with long text
- Test with special characters
- Test with actual staff doing actual tasks
- Fix edge cases before launch

---

### Pitfall 5: Building Everything Before Getting Feedback

**The Problem:**
- Build complete app in isolation
- Show staff when "done"
- Find out they wanted it differently
- Major rework required

**The Solution:**
- Show work early and often
- Get feedback at each milestone
- Build progressively, demo each piece
- Adjust based on feedback before moving on
- "Fail fast" - find issues early when they're cheap to fix

---

### Pitfall 6: Forgetting About Permissions

**The Problem:**
- App works fine for you (developer/admin)
- Users get errors due to permissions
- Can't create records, can't see data

**The Solution:**
- Set SharePoint permissions early
- Test with non-admin account
- Verify users have: Read, Create, Edit access to all lists
- Test Power Apps sharing settings
- Document required permissions

---

### Pitfall 7: No Support Plan

**The Problem:**
- Launch app and walk away
- Users have questions, find bugs
- No one to help, adoption fails

**The Solution:**
- Plan for ongoing support
- Designate "app champion" at site
- Schedule check-ins (1 week, 1 month, 3 months)
- Have bug reporting process
- Budget time for fixes and enhancements
- Treat launch as beginning, not end

---

## Maintenance and Updates

### Ongoing Maintenance Plan

**Weekly (First Month):**
- [ ] Check in with users
- [ ] Collect feedback
- [ ] Fix critical bugs immediately
- [ ] Document non-critical issues for batch fix

**Monthly (First 6 Months):**
- [ ] Review usage analytics
- [ ] Identify unused features
- [ ] Identify missing features users need
- [ ] Batch fix non-critical bugs
- [ ] Consider Phase 2 additions

**Quarterly (Ongoing):**
- [ ] Review data quality
- [ ] Update choice options if needed
- [ ] Review and update training materials
- [ ] Plan next phase of features

---

### Making Changes After Launch

**Adding a New Field:**
1. Add to SharePoint list first
2. Test with sample data
3. Add to Power App forms/displays
4. Update training materials
5. Notify users of new field

**Changing a Field (AVOID IF POSSIBLE):**
- Renaming: Can break Power App formulas
- Changing type: May lose data
- **Better:** Add new field, migrate data, retire old field

**Adding a Workflow:**
1. Design on paper first
2. Build in development environment
3. Test thoroughly
4. Deploy to production during low-usage time
5. Notify and train users

---

### When to Consider Rebuilding

**Signs It's Time to Rebuild:**
- Field naming is a mess and causing constant issues
- App has become so complex it's unmaintainable
- Users are frustrated and avoiding the app
- Too many workarounds and patches
- Clear vision for better structure

**Rebuilding Approach:**
1. Keep current app running
2. Build new app following best practices (this guide!)
3. Test thoroughly
4. Migrate data
5. Switch over
6. Retire old app

**Use Lessons Learned:**
- Name fields correctly from the start
- Complete schema before building
- Build progressively
- Launch with MVP only

---

## Summary: Key Principles for Success

### 1. **Name Fields Right From the Start**
- No special characters
- Exactly how you want them to display
- Consistent across lists
- This saves HOURS of pain

### 2. **Complete Schema Before Building**
- All fields designed and named
- All field types chosen
- All choice options determined
- Field dictionary documented
- Stakeholder sign-off obtained

### 3. **Match Field Types Between Lists**
- If data will transfer, types must match exactly
- Create verification matrix
- Test data flow before building workflows

### 4. **Start Small, Build Progressively**
- Launch with minimum viable features
- Add complexity based on actual usage
- Test each piece before moving on
- Use progressive debugging when issues arise

### 5. **Smaller Sites Are Easier**
- Consider piloting at smaller site first
- Fewer units = simpler workflows
- Fewer users = easier training
- Less data = faster testing
- Refine before bringing to larger site

### 6. **Plan for Support**
- App needs ongoing care
- Budget time for fixes and enhancements
- Designate app champion
- Check in regularly with users

### 7. **Documentation Matters**
- Document decisions and why
- Document workarounds
- Update documentation as you go
- Makes future maintenance much easier

---

## Appendix A: Field Dictionary Template

Use this template to plan your fields before creating them in SharePoint.

```
==============================================
FIELD DICTIONARY: [Your Site Name] Clearview
==============================================

LIST: Units
-----------

Field: Unit
- Display Name: Unit
- Internal Name: Title
- Type: Single line text
- Required: Yes
- Description: Unit number or identifier (e.g., "101")
- Examples: 101, 201A, Garden Unit 5

Field: Unit Type
- Display Name: Unit Type
- Internal Name: UnitType
- Type: Choice
- Required: Yes
- Options:
  * LTH Housing Support
  * Section 8 PBV
  * Ramsey CoC
  * (add your types)
- Description: Program type for this unit

Field: Is Occupied
- Display Name: Is Occupied
- Internal Name: IsOccupied
- Type: Yes/No
- Required: Yes
- Default: No
- Description: Whether unit currently has resident

[Continue for all fields...]

LIST: Referrals & Applicants
-----------------------------

Field: Head of Household
- Display Name: Head of Household
- Internal Name: Title
- Type: Single line text
- Required: Yes
- Description: Name of primary household member
- Examples: Smith, John; Nguyen, Mai

[Continue for all fields...]

[Repeat for each list]
```

---

## Appendix B: Adaptation Checklist

Use this checklist when adapting UP Clearview for a new site:

```
SITE ADAPTATION CHECKLIST
==========================

Site: _______________________
Prepared By: ________________
Date: _______________________

☐ PLANNING COMPLETE
  ☐ Stakeholder interviews conducted
  ☐ Current workflows documented
  ☐ Pain points identified
  ☐ MVP scope defined and agreed
  ☐ Field inventory completed
  ☐ Field dictionary created
  ☐ Schema review meeting held
  ☐ Sign-off obtained

☐ SHAREPOINT SETUP
  ☐ Site identified and permissions verified
  ☐ Unit Types list created (if needed)
  ☐ Units list created
  ☐ Referrals list created
  ☐ Residents list created
  ☐ Field types verified between lists
  ☐ Cross-list lookups added
  ☐ Optional lists created (as needed)
  ☐ Sample data loaded
  ☐ Relationships tested

☐ POWER APP - MILESTONE 1
  ☐ New app created
  ☐ Data sources connected
  ☐ UnitBrowseScreen built
  ☐ RNABrowseScreen built
  ☐ ResBrowseScreen built
  ☐ Navigation between browse screens works
  ☐ Milestone 1 tested and approved

☐ POWER APP - MILESTONE 2
  ☐ UnitInfoScreen built
  ☐ RNAInfoScreen built
  ☐ ResInfoScreen built
  ☐ Browse → Detail navigation works
  ☐ Variables set correctly
  ☐ Milestone 2 tested and approved

☐ POWER APP - MILESTONE 3
  ☐ Edit mode enabled on all info screens
  ☐ Save/Cancel functionality works
  ☐ Changes persist to SharePoint
  ☐ OnSuccess handlers added
  ☐ Notifications display
  ☐ Milestone 3 tested and approved

☐ POWER APP - MILESTONE 4
  ☐ Receive Referral workflow built
  ☐ Button visibility logic works
  ☐ Form creates referral
  ☐ Unit updates correctly
  ☐ End-to-end workflow tested
  ☐ Milestone 4 tested and approved

☐ POWER APP - MILESTONE 5
  ☐ Convert to Resident workflow built
  ☐ Progressive build approach used
  ☐ All fields copying correctly
  ☐ Unit updates correctly
  ☐ Referral deletes correctly
  ☐ End-to-end workflow tested
  ☐ Milestone 5 tested and approved

☐ ADDITIONAL FEATURES (as needed)
  ☐ [List additional features for your site]

☐ TESTING COMPLETE
  ☐ All browse screens tested
  ☐ All info screens tested
  ☐ All edit operations tested
  ☐ All workflows tested end-to-end
  ☐ Integration testing complete
  ☐ Edge cases tested
  ☐ UAT conducted with staff
  ☐ Critical bugs fixed
  ☐ Sign-off obtained

☐ DEPLOYMENT READY
  ☐ Training materials prepared
  ☐ Staff training scheduled
  ☐ Support plan in place
  ☐ App champion designated
  ☐ Go-live date set
  ☐ Rollback plan documented

☐ POST-LAUNCH
  ☐ Week 1 check-in completed
  ☐ Week 2 check-in completed
  ☐ Month 1 review completed
  ☐ Feedback collected and documented
  ☐ Phase 2 planning begun
```

---

**Document Version:** 1.0
**Date:** October 24, 2025
**Last Updated:** After Convert to Resident workflow completion
**Next Review:** After first site adaptation

---

**Remember:** Every site is different. Use this guide as a framework, but adapt to your specific needs. When in doubt, start simpler rather than more complex. You can always add features later, but it's hard to remove complexity once users are relying on it.

**Good luck with your deployment!**
