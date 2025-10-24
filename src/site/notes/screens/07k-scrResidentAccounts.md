---
{"dg-publish":true,"permalink":"/screens/07k-scr-resident-accounts/","dgShowToc":true}
---

# scrResidentAccounts

**Purpose:** Manage bill pay account information for Housing Support residents.
**Status:** Ready for Deployment (pending password policy decision)
**Data Sources:** Account Information, Current Residents

## Overview

scrResidentAccounts provides a form-based interface for creating and editing bill pay account records for residents enrolled in Housing Support programs. These accounts represent utilities, services, and other bills that case managers pay on behalf of residents as part of their housing support package.

The form includes conditional visibility logic that shows billing-specific fields (frequency, cost) only when the "Is Payable Account" toggle is set to true. This allows the list to also track accounts that residents manage themselves (for reference purposes) without cluttering the interface with irrelevant payment fields.

The form is fully functional and ready for production use, with the only blocker being a policy decision about password field security. The current recommendation is to remove the password field entirely and rely on browser-based password managers.

## Key Components

### Main Controls
- **AccountForm** - Form bound to Account Information list
- **DataCards for:**
  - Linked Resident (lookup, auto-populated with varSelectedResident)
  - Account Provider/Payee (text)
  - Account Type (choice: Electric, Gas, Water, Internet, Phone, etc.)
  - Username (text)
  - Account Number (text)
  - Password (text) - **SECURITY CONCERN**
  - Is Payable Account (yes/no toggle)
  - Billing Frequency (choice: Monthly, Quarterly, Annual) - conditional
  - Monthly Cost (number) - conditional
  - Payment Notes (multi-line text) - conditional

### Data Flow
- **OnVisible:** Sets default Linked Resident to varSelectedResident
- **Form DefaultMode:** New or Edit depending on navigation context
- **Conditional Visibility:** Billing fields visible only if "Is Payable Account" = true
- **Form OnSuccess:** Saves record and navigates back to scrResidentAssociations

## Features

### Account Information Form
**Status:** Complete
**Description:** Comprehensive form for capturing all account details needed for bill payment management.

**Default Values:**
```powerapps
DataCard_LinkedResident.Default: =varSelectedResident
DataCard_IsPayableAccount.Default: =true
```

**Design Notes:**
- Linked Resident defaults to currently selected resident (usually hidden or read-only)
- Is Payable Account defaults to true (most common use case)
- Form can be used in New or Edit mode

### Conditional Billing Fields
**Status:** Complete
**Description:** Payment-related fields show only when the account is marked as payable.

**Visibility Logic:**
```powerapps
Visible: =DataCardValue_IsPayableAccount.Value = true
```

**Conditional Fields:**
- Billing Frequency (Monthly, Quarterly, Annual)
- Monthly Cost (number)
- Payment Notes (additional billing details)

**Design Rationale:**
- Allows tracking of non-payable accounts (for resident reference)
- Reduces form clutter for reference-only accounts
- Clearly distinguishes between Housing Support paid accounts and resident-managed accounts

### Form Save and Navigation
**Status:** Complete
**Description:** OnSuccess handler saves the account and returns to associations screen.

**OnSuccess Logic:**
```powerapps
OnSuccess: |-
  =Notify("Account information saved successfully.", NotificationType.Success);
  Navigate(scrResidentAssociations, ScreenTransition.None);
  Refresh('Account Information')
```

## Known Issues

### Password Field Security Concern
**Severity:** High (Security/Compliance)
**Impact:** Passwords stored in plain text in SharePoint list (not encrypted)
**Symptoms:** Potential security vulnerability and compliance issue
**Regulatory Concern:** May violate data security policies
**Current Workaround:** None
**Recommended Solution:** Remove password field from list and form, document use of browser password managers
**Alternative Solution:** Integrate with Azure Key Vault or third-party credential management system

**Policy Decision Needed:**
1. **Option A (Recommended):** Remove password field entirely
   - Document browser password manager usage in user guide
   - Educate staff on password manager best practices
   - Effort: 1 hour (remove field, update documentation)

2. **Option B:** Implement secure credential storage
   - Integrate with Azure Key Vault
   - Requires Azure subscription and configuration
   - Effort: 20+ hours (research, implementation, testing)

3. **Option C:** Accept risk with documentation
   - Document that passwords are stored unencrypted
   - Require admin approval and risk acceptance
   - Limit list permissions to minimize exposure
   - Effort: 2 hours (documentation, permission review)

**Deployment Blocker:** Cannot deploy to production with plain text password storage without explicit policy decision and approval.

## Testing Notes

**Test Cases - Form Functionality:**
- [ ] Navigate to scrResidentAssociations
- [ ] Click "Add Account" button (or equivalent navigation)
- [ ] Verify AccountForm loads in New mode
- [ ] Verify Linked Resident pre-filled with current resident
- [ ] Enter account provider name
- [ ] Select account type from dropdown
- [ ] Enter username and account number
- [ ] Toggle "Is Payable Account" to true
- [ ] Verify billing fields appear (frequency, cost, notes)
- [ ] Enter billing frequency, monthly cost, and notes
- [ ] Click Save
- [ ] Verify success notification
- [ ] Verify navigation to scrResidentAssociations
- [ ] Verify new account appears in accounts gallery

**Test Cases - Conditional Visibility:**
- [ ] Create new account
- [ ] Set "Is Payable Account" to false
- [ ] Verify billing fields disappear
- [ ] Set "Is Payable Account" to true
- [ ] Verify billing fields reappear
- [ ] Save with Is Payable = false (billing fields blank)
- [ ] Verify record saves successfully
- [ ] Edit existing account
- [ ] Toggle Is Payable between true and false
- [ ] Verify conditional fields show/hide correctly

**Test Cases - Edit Mode:**
- [ ] From scrResidentAssociations, click existing account
- [ ] Verify AccountForm loads in Edit mode
- [ ] Verify all fields populated with existing data
- [ ] Modify account provider name
- [ ] Change account type
- [ ] Update monthly cost
- [ ] Click Save
- [ ] Verify changes persist
- [ ] Navigate back and verify updated information displays

**Test Cases - Validation:**
- [ ] Attempt to save without required fields
- [ ] Verify validation errors appear
- [ ] Verify form prevents save until required fields filled
- [ ] Enter invalid data (letters in cost field)
- [ ] Verify validation error
- [ ] Test field character limits (very long provider name)

**Test Cases - Edge Cases:**
- [ ] Create account with minimal data (provider and type only)
- [ ] Create account with all fields populated
- [ ] Create multiple accounts for same resident
- [ ] Edit account immediately after creation
- [ ] Cancel form without saving (if cancel button exists)

## Development Roadmap

**Immediate (Pre-Deployment):**
- [ ] Policy decision on password field (1 hour meeting)
- [ ] Implement password solution based on decision (1-20 hours depending on option)
- [ ] Final testing with resolved password issue
- [ ] Deploy to production

**Post-Deployment Enhancements:**
- [ ] Add "Add New Account" button directly on scrResidentAssociations
- [ ] Implement inline editing of accounts from associations screen
- [ ] Add bulk payment view (group accounts by payment date)
- [ ] Add payment tracking (mark as paid, payment history)
- [ ] Add payment reminders/notifications

## Related Documentation

- [scrResidentAssociations](07j-scrResidentAssociations.md) - Parent screen showing all accounts
- [ResInfoScreen](07f-ResInfoScreen.md) - Resident details screen
- [Data Model - Account Information List](../03-Data-Model.md#account-information-list) - Schema for accounts
- [Security & Compliance](../09-Known-Issues.md#password-storage-security) - Security concerns and recommendations
