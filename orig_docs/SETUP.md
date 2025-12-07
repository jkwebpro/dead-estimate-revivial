# Dead Estimate Revival - Setup Documentation

## Overview

This automation revives cold/ghosted estimates through scheduled SMS and email follow-ups. It consists of two workflows:

1. **Main Follow-up Workflow** - Scheduled engine that sends follow-ups
2. **Inbound SMS Handler** - Processes customer replies and opt-outs

---

## Prerequisites

### Accounts Required

| Service | Purpose | Setup Link |
|---------|---------|------------|
| n8n | Workflow automation | [n8n.io](https://n8n.io) |
| Google Workspace | Sheets + Gmail | [workspace.google.com](https://workspace.google.com) |
| Twilio | SMS sending/receiving | [twilio.com](https://twilio.com) |

### Twilio Setup

1. Create a Twilio account
2. Purchase a phone number with SMS capability
3. Note your:
   - Account SID
   - Auth Token
   - Phone Number (E.164 format: +1XXXXXXXXXX)

---

## Step 1: Create Google Sheet

### 1.1 Create New Spreadsheet

Create a new Google Sheet named: `Dead Estimate Revival`

### 1.2 Create Config Sheet

Add a sheet tab named `Config` with these values:

| Row | A (Key) | B (Value) |
|-----|---------|-----------|
| 1 | sheet_id | [Your Google Sheet ID] |
| 2 | company_name | [Your Company Name] |
| 3 | sender_name | [Your Name] |
| 4 | sender_email | [your@email.com] |
| 5 | sender_phone | [(555) 123-4567] |
| 6 | twilio_phone_number | [+1XXXXXXXXXX] |
| 7 | max_attempts | 4 |
| 8 | mode | demo |

**Note:** Change `mode` to `production` for real-world timing.

### 1.3 Create Quote_Followups Sheet

Add a sheet tab named `Quote_Followups` with these column headers (Row 1):

| A | B | C | D | E | F | G | H | I | J | K | L | M |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| quote_id | customer_name | customer_email | customer_phone | quote_amount | quote_date | preferred_channel | follow_up_count | last_follow_up | next_follow_up | status | opted_out | notes |

### 1.4 Add Data Validation

**Column G (preferred_channel):**
- Select column G (excluding header)
- Data > Data validation
- Criteria: Dropdown
- Options: `email`, `sms`, `both`

**Column K (status):**
- Select column K (excluding header)
- Data > Data validation
- Criteria: Dropdown
- Options: `active`, `replied`, `won`, `lost`, `opted_out`

**Column L (opted_out):**
- Select column L (excluding header)
- Insert > Checkbox

### 1.5 Get Sheet ID

Your Sheet ID is in the URL:
```
https://docs.google.com/spreadsheets/d/[THIS_IS_YOUR_SHEET_ID]/edit
```

---

## Step 2: Configure n8n Credentials

### 2.1 Google Sheets Credential

1. In n8n, go to **Credentials**
2. Add new **Google Sheets OAuth2 API** credential
3. Follow OAuth flow to authorize your Google account
4. Name it: `Google Sheets Account`

### 2.2 Gmail Credential

1. Add new **Gmail OAuth2 API** credential
2. Authorize the same Google account
3. Name it: `Gmail Account`

### 2.3 Twilio Credential

1. Add new **Twilio API** credential
2. Enter:
   - Account SID
   - Auth Token
3. Name it: `Twilio Account`

---

## Step 3: Import Workflows

### 3.1 Main Follow-up Workflow

1. In n8n, go to **Workflows**
2. Click **Import from File**
3. Select `main-followup-workflow.json`
4. Open the imported workflow

### 3.2 Configure Main Workflow Nodes

**Get Config Node:**
- Update `documentId` to your Sheet ID

**Get Active Quotes Node:**
- Update `documentId` to your Sheet ID

**Send Email Node:**
- Select your Gmail credential

**Send SMS Node:**
- Select your Twilio credential

**Update Sheet Node:**
- Select your Google Sheets credential

### 3.3 Inbound SMS Handler Workflow

1. Import `inbound-sms-handler.json`
2. Open the workflow

**Find Customer by Phone Node:**
- Update `documentId` to your Sheet ID

**Mark Opted Out Node:**
- Update `documentId` to your Sheet ID

**Send Opt-Out Confirmation Node:**
- Update `from` to your Twilio phone number
- Select your Twilio credential

**Mark as Replied Node:**
- Update `documentId` to your Sheet ID

**Notify Owner Node:**
- Update `to` with owner's email address
- Select your Gmail credential

---

## Step 4: Configure Twilio Webhook

### 4.1 Get Webhook URL

1. In n8n, open the Inbound SMS Handler workflow
2. Click the **Twilio Webhook** node
3. Copy the **Production URL** (or Test URL for testing)

### 4.2 Configure Twilio

1. Go to [Twilio Console](https://console.twilio.com)
2. Navigate to **Phone Numbers** > **Manage** > **Active Numbers**
3. Click your SMS-enabled number
4. Under **Messaging Configuration**:
   - Set **A message comes in** to: `Webhook`
   - Paste your n8n webhook URL
   - Method: `HTTP POST`
5. Save

---

## Step 5: Test the Automation

### 5.1 Demo Mode Test

1. Ensure `Config` sheet has `mode` = `demo`
2. Add a test row to `Quote_Followups`:

| quote_id | customer_name | customer_email | customer_phone | quote_amount | quote_date | preferred_channel | follow_up_count | last_follow_up | next_follow_up | status | opted_out | notes |
|----------|---------------|----------------|----------------|--------------|------------|-------------------|-----------------|----------------|----------------|--------|-----------|-------|
| TEST001 | Test User | your@email.com | +1XXXXXXXXXX | $1,500 | 2024-01-15 | both | 0 | | [NOW] | active | FALSE | |

**Note:** Set `next_follow_up` to the current time (ISO format) to trigger immediately.

3. Activate both workflows
4. Wait for the schedule trigger (1 minute)
5. Check:
   - Email inbox
   - SMS received
   - Sheet updated (follow_up_count, last_follow_up, next_follow_up)

### 5.2 Test Opt-Out

1. Reply "STOP" to the SMS
2. Check:
   - Sheet shows `opted_out = TRUE` and `status = opted_out`
   - Confirmation SMS received

### 5.3 Test Reply Detection

1. Add another test row
2. Wait for SMS
3. Reply with anything other than "STOP"
4. Check:
   - Sheet shows `status = replied`
   - Owner received notification email

---

## Step 6: Go Live

### 6.1 Switch to Production Mode

1. In `Config` sheet, change `mode` to `production`
2. Timing will now use:
   - Attempt 1: 2 days
   - Attempt 2: 5 days
   - Attempt 3: 10 days
   - Attempt 4: 14 days

### 6.2 Adjust Schedule Trigger (Optional)

For production, you may want to reduce polling frequency:
1. Open Main Follow-up Workflow
2. Edit **Schedule Trigger** node
3. Change interval (e.g., every 15 minutes or hourly)

---

## Workflow Logic Summary

### Main Follow-up Flow

```
Schedule Trigger (every 1 min)
    │
    ▼
Get Config (reads settings)
    │
    ▼
Get Active Quotes (status=active, opted_out=FALSE)
    │
    ▼
Filter Due Quotes (next_follow_up <= NOW, under max_attempts)
    │
    ▼
Route by Channel
    ├─► email → Select Template → Send Email
    ├─► sms → Select Template → Send SMS
    └─► both → Send Email + Send SMS
    │
    ▼
Calculate Next Follow-up
    │
    ▼
Update Sheet
```

### Inbound SMS Flow

```
Twilio Webhook (receives SMS)
    │
    ▼
Parse Incoming SMS
    │
    ▼
Find Customer by Phone
    │
    ▼
Customer Found?
    │
    ▼
Opt-Out or Reply?
    ├─► STOP → Mark Opted Out → Send Confirmation
    └─► Other → Mark Replied → Notify Owner
    │
    ▼
Respond to Twilio
```

---

## Troubleshooting

### Issue: Emails not sending

- Verify Gmail credential is authorized
- Check Gmail sending limits (500/day for free accounts)
- Review n8n execution logs

### Issue: SMS not sending

- Verify Twilio credential
- Check Twilio phone number is SMS-enabled
- Verify customer phone is E.164 format (+1XXXXXXXXXX)
- Check Twilio account balance

### Issue: Webhook not receiving

- Confirm webhook URL is correct in Twilio
- Test webhook URL is accessible (not behind firewall)
- Check n8n workflow is active

### Issue: Sheet not updating

- Verify Google Sheets credential
- Check sheet ID is correct
- Ensure quote_id column has unique values

---

## Customization Options

### Change Follow-up Cadence

Edit the **Calculate Next Follow-up** node expression to adjust timing intervals.

### Add More Channels

Extend the **Route by Channel** switch node to add additional channels (e.g., Slack, WhatsApp).

### Custom Templates

Edit the template Set nodes to customize message content.

### Add Owner Notifications

Add notification nodes after Send Email/SMS to alert owner of each outreach.

---

## File Inventory

| File | Purpose |
|------|---------|
| `main-followup-workflow.json` | Primary automation workflow |
| `inbound-sms-handler.json` | SMS reply/opt-out handler |
| `SETUP.md` | This documentation |

---

## Support

For issues or enhancements, document the following:
- n8n version
- Node versions
- Error messages from execution logs
- Screenshots of node configuration
