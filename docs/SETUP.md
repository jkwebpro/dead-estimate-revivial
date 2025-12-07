# Setup Guide

Complete setup instructions for the Dead Estimate Revival automation.

## Prerequisites

- n8n instance (self-hosted or cloud)
- Google Workspace account with Sheets and Gmail access
- Twilio account with SMS-enabled phone number
- Basic familiarity with n8n workflow editor

## Step 1: Create Google Sheet

### 1.1 Create New Spreadsheet

1. Go to Google Sheets
2. Create new spreadsheet named "Dead Estimate Revival"
3. Copy the Sheet ID from URL: `https://docs.google.com/spreadsheets/d/[SHEET_ID]/edit`

### 1.2 Create Config Tab

Rename the first tab to "Config" and add these rows:

| key | value |
|-----|-------|
| mode | demo |
| max_attempts | 4 |
| sender_name | Your Name |
| sender_email | you@email.com |
| sender_phone | (555) 123-4567 |
| twilio_phone_number | +15551234567 |

### 1.3 Create Quote_Followups Tab

Create a second tab named "Quote_Followups" with these column headers in Row 1:

```
quote_id | customer_name | customer_email | customer_phone | quote_amount | quote_date | preferred_channel | follow_up_count | last_follow_up | next_follow_up | status | opted_out | notes
```

### 1.4 Add Data Validation (Optional)

For cleaner data entry, add dropdown validation:

**preferred_channel column:**
- Select column range (e.g., G2:G100)
- Data > Data validation
- Criteria: Dropdown (from a range)
- Options: email, sms, both

**status column:**
- Select column range (e.g., K2:K100)
- Criteria: Dropdown
- Options: active, replied, won, lost, opted_out

**opted_out column:**
- Select column range (e.g., L2:L100)
- Criteria: Checkbox

> **Important**: Limit validation ranges (e.g., G2:G100) rather than entire columns. Unlimited ranges create empty rows that cause duplicate sends.

## Step 2: Configure n8n Credentials

### 2.1 Google Sheets OAuth2

1. In n8n, go to Credentials > Add Credential
2. Select "Google Sheets OAuth2 API"
3. Follow OAuth flow to authorize
4. Test connection

### 2.2 Gmail OAuth2

1. Add Credential > "Gmail OAuth2 API"
2. Authorize with your Google account
3. Grant send email permissions
4. Test connection

### 2.3 Twilio API

1. Add Credential > "Twilio API"
2. Enter Account SID (from Twilio Console)
3. Enter Auth Token (from Twilio Console)
4. Test connection

## Step 3: Import Workflows

### 3.1 Main Follow-up Workflow

1. In n8n, click "Add workflow"
2. Click menu (3 dots) > Import from file
3. Select `workflows/main-followup-workflow.json`
4. Save workflow

### 3.2 Inbound SMS Handler

1. Create new workflow
2. Import `workflows/inbound-sms-handler.json`
3. Save workflow

## Step 4: Configure Workflows

### 4.1 Main Follow-up Workflow

Update these nodes with your Google Sheet ID:

**Get Config node:**
1. Open node
2. Under "Document", select your spreadsheet or paste Sheet ID
3. Under "Sheet", select "Config"

**Get Active Quotes node:**
1. Select same spreadsheet
2. Select "Quote_Followups" sheet

**Update Sheet node:**
1. Select same spreadsheet
2. Select "Quote_Followups" sheet

**Send Email node:**
1. Select your Gmail credential

**Send SMS node:**
1. Select your Twilio credential

### 4.2 Inbound SMS Handler

**Find Customer by Phone node:**
1. Select your spreadsheet
2. Select "Quote_Followups" sheet

**Mark Opted Out / Mark as Replied nodes:**
1. Select same spreadsheet
2. Select "Quote_Followups" sheet

**Notify Owner node:**
1. Update "Send To" field with your email address
2. Select Gmail credential

**Send Owner SMS node:**
1. Update "From" with your Twilio number
2. Update "To" with your personal phone number
3. Select Twilio credential

**Twilio Trigger node:**
1. Select your Twilio credential
2. After saving, n8n generates webhook URL
3. Copy this URL for Twilio configuration

### 4.3 Configure Twilio Webhook

1. Go to Twilio Console > Phone Numbers
2. Select your SMS-enabled number
3. Under "Messaging Configuration"
4. Set "A message comes in" webhook to your n8n Twilio Trigger URL
5. Save

## Step 5: Test the Setup

See [TESTING.md](TESTING.md) for complete testing procedures.

### Quick Test

1. Add test row to Quote_Followups:

| Field | Value |
|-------|-------|
| quote_id | TEST001 |
| customer_name | Your Name |
| customer_email | your-email@example.com |
| customer_phone | +1XXXXXXXXXX |
| quote_amount | 1500 |
| quote_date | 2024-01-15 |
| preferred_channel | email |
| follow_up_count | 0 |
| last_follow_up | (leave empty) |
| next_follow_up | 2024-01-01T00:00:00.000Z |
| status | active |
| opted_out | FALSE |

2. Open Main Follow-up workflow
3. Click "Test Workflow"
4. Check email inbox
5. Verify sheet updated (follow_up_count = 1)

## Step 6: Activate Workflows

Once testing is complete:

1. Open each workflow
2. Toggle "Active" switch in top-right
3. Confirm workflows are running

## Troubleshooting

### No emails sending

- Check Gmail credential is valid
- Verify `next_follow_up` is a past timestamp
- Verify `status` = "active" and `opted_out` = FALSE
- Check Filter Valid Quotes output (should show your test row)

### Multiple emails per record

- Check for duplicate rows in sheet
- Ensure data validation ranges are limited (not entire columns)
- Delete empty rows created by validation

### SMS not working

- Verify Twilio phone number is SMS-enabled
- Check phone format is E.164 (+15551234567)
- Verify Twilio credential is valid
- Check Twilio account has sufficient balance

### Inbound SMS not detected

- Verify Twilio webhook URL is correct
- Check Inbound SMS Handler workflow is active
- Test webhook with Twilio Console test feature

### DateTime errors in Filter Due Quotes

- Enable "Loose type validation" in Filter Due Quotes
- Ensure `next_follow_up` contains valid ISO timestamps
- Check for empty cells in `next_follow_up` column

## Production Checklist

Before going live:

- [ ] Change Config `mode` from "demo" to "production"
- [ ] Adjust Schedule Trigger interval (15 min or hourly)
- [ ] Update email templates with production messaging
- [ ] Update SMS templates with production messaging
- [ ] Test with real customer data (small batch first)
- [ ] Monitor first few executions
- [ ] Set up error notifications in n8n
