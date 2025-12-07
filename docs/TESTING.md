# Testing Guide

Complete testing procedures for the Dead Estimate Revival automation.

## Test Environment Setup

### Prerequisites

- Workflows imported and configured
- Google Sheet created with Config and Quote_Followups tabs
- All credentials connected and tested
- Personal email and phone ready to receive test messages

### Config Settings for Testing

Ensure Config tab has:

| key | value |
|-----|-------|
| mode | demo |
| max_attempts | 4 |

Demo mode uses 1-4 minute intervals instead of days.

## Test Data Setup

### Create Test Record

Add one row to Quote_Followups:

| Field | Value |
|-------|-------|
| quote_id | TEST001 |
| customer_name | Test User |
| customer_email | your-real-email@example.com |
| customer_phone | +1XXXXXXXXXX (your real phone) |
| quote_amount | 1500 |
| quote_date | 2024-01-15 |
| preferred_channel | email |
| follow_up_count | 0 |
| last_follow_up | (leave empty) |
| next_follow_up | 2024-01-01T00:00:00.000Z |
| status | active |
| opted_out | FALSE |
| notes | test record |

### Clear Previous Test Data

Before each test run:
1. Delete all test rows except one
2. Reset test row: `follow_up_count` = 0, `status` = active
3. Set `next_follow_up` to past timestamp

## Test Procedures

### Test 1: Main Follow-up Workflow (Node-by-Node)

**Purpose**: Verify each node processes data correctly.

**Steps**:

1. Open Main Follow-up workflow
2. Click "Schedule Trigger" node > Test step
3. Click "Get Config" > Test step
   - **Expected**: 6+ config rows returned
4. Click "Parse Config" > Test step
   - **Expected**: Single item with `config` object containing all settings
5. Click "Single Config Item" > Test step
   - **Expected**: 1 item output
6. Click "Get Active Quotes" > Test step
   - **Expected**: All rows from Quote_Followups (including empty)
7. Click "Filter Valid Quotes" > Test step
   - **Expected**: Only valid rows with quote_id, customer_name, email containing @, status=active, opted_out=FALSE
8. Click "Filter Due Quotes" > Test step
   - **Expected**: Only rows where next_follow_up is past and follow_up_count < max_attempts
9. Click "Route by Channel" > Test step
   - **Expected**: Item routed to correct output (email/sms/both)
10. Continue through template selector and template nodes
11. Click "Send Email" > Test step
    - **Expected**: Email sent, check inbox
12. Click "Calculate Next Follow-up" > Test step
    - **Expected**: new_follow_up_count = 1, new_next_follow_up = 1 min from now
13. Click "Update Sheet" > Test step
    - **Expected**: Sheet row updated

**Pass Criteria**:
- [ ] Email received with correct content
- [ ] Sheet updated: follow_up_count = 1
- [ ] Sheet updated: last_follow_up = current timestamp
- [ ] Sheet updated: next_follow_up = 1 minute from now
- [ ] Sheet updated: status = active

### Test 2: Full Workflow Execution

**Purpose**: Verify end-to-end flow works.

**Steps**:

1. Reset test record (follow_up_count = 0, past next_follow_up)
2. Click "Test Workflow" button
3. Wait for completion
4. Check results

**Pass Criteria**:
- [ ] Workflow completes without errors
- [ ] Email/SMS received
- [ ] Sheet updated correctly

### Test 3: Message Escalation (Email)

**Purpose**: Verify all 4 email templates send in sequence.

**Steps**:

1. Reset test record
2. Run workflow - receive Email Template 1
3. Set next_follow_up to past timestamp
4. Run workflow - receive Email Template 2
5. Repeat for Templates 3 and 4
6. After Template 4, verify status = "lost"

**Pass Criteria**:
- [ ] Template 1: Subject "Quick follow-up on your estimate"
- [ ] Template 2: Subject "Still thinking it over?"
- [ ] Template 3: Subject "Before this slips away"
- [ ] Template 4: Subject "Closing the loop"
- [ ] After 4th: status changes to "lost"

### Test 4: SMS Channel

**Purpose**: Verify SMS sending works.

**Steps**:

1. Change test record: preferred_channel = "sms"
2. Reset: follow_up_count = 0, past next_follow_up
3. Run workflow
4. Check phone for SMS

**Pass Criteria**:
- [ ] SMS received
- [ ] Message contains customer name and quote amount
- [ ] First SMS includes "Reply STOP to opt out"

### Test 5: Both Channels

**Purpose**: Verify dual-channel sending.

**Steps**:

1. Change test record: preferred_channel = "both"
2. Reset record
3. Run workflow
4. Check email AND phone

**Pass Criteria**:
- [ ] Email received
- [ ] SMS received
- [ ] Sheet updated once (not twice)

### Test 6: Multiple Quotes

**Purpose**: Verify batch processing.

**Steps**:

1. Add second test record (TEST002) with different email
2. Set both records: follow_up_count = 0, past next_follow_up
3. Run workflow
4. Check both received messages

**Pass Criteria**:
- [ ] Both quotes processed
- [ ] Each received appropriate message
- [ ] Both sheet rows updated

### Test 7: Inbound SMS - Reply

**Purpose**: Verify reply detection and owner notification.

**Steps**:

1. Ensure Inbound SMS Handler is active
2. Send SMS reply to Twilio number (not containing "STOP")
3. Check sheet status
4. Check owner email notification
5. Check owner SMS notification

**Pass Criteria**:
- [ ] Sheet status changed to "replied"
- [ ] Owner email received with reply content
- [ ] Owner SMS received with reply summary

### Test 8: Inbound SMS - Opt-Out

**Purpose**: Verify STOP keyword handling.

**Steps**:

1. Reset test record (status = active, opted_out = FALSE)
2. Send SMS containing "STOP" to Twilio number
3. Check sheet
4. Check for opt-out confirmation SMS
5. Check owner SMS notification

**Pass Criteria**:
- [ ] Sheet: opted_out = TRUE
- [ ] Customer received opt-out confirmation
- [ ] Owner SMS received opt-out notification
- [ ] Subsequent workflow runs skip this record

### Test 9: Opt-Out via Sheet

**Purpose**: Verify manual opt-out.

**Steps**:

1. Reset test record
2. Manually set opted_out = TRUE in sheet
3. Set past next_follow_up
4. Run workflow

**Pass Criteria**:
- [ ] No message sent
- [ ] Record filtered out by Filter Valid Quotes

### Test 10: Invalid Data Handling

**Purpose**: Verify bad data doesn't cause errors.

**Steps**:

1. Add row with missing quote_id
2. Add row with invalid email (no @)
3. Add row with status = "won"
4. Run workflow

**Pass Criteria**:
- [ ] Workflow completes without errors
- [ ] Invalid rows filtered out
- [ ] Valid rows still processed

## Test Checklist Summary

### Core Functionality
- [ ] Test 1: Node-by-node execution
- [ ] Test 2: Full workflow execution
- [ ] Test 3: Message escalation
- [ ] Test 4: SMS channel
- [ ] Test 5: Both channels

### Advanced Features
- [ ] Test 6: Multiple quotes
- [ ] Test 7: Inbound reply handling
- [ ] Test 8: STOP opt-out
- [ ] Test 9: Manual opt-out
- [ ] Test 10: Invalid data handling

## Production Verification

Before activating in production:

1. Run all tests with production Config settings
2. Test with 1-2 real (non-critical) quotes
3. Monitor first 24 hours of automated execution
4. Verify no duplicate sends
5. Confirm timing intervals are correct

## Common Issues

### Email not sending
- Verify Gmail credential
- Check Send Email node configuration
- Verify customer_email is valid

### Duplicate messages
- Check for duplicate rows in sheet
- Verify data validation ranges are limited
- Check Single Config Item node exists

### Wrong template sent
- Verify follow_up_count in sheet
- Check Email/SMS Template Selector routing
- Ensure type validation is set to "loose"

### Sheet not updating
- Verify Update Sheet has correct Sheet ID
- Check matching column is quote_id
- Verify quote_id reference: `$('Filter Valid Quotes').item.json.quote_id`
