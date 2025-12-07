# Dead Estimate Revival

Automated follow-up system for reviving cold/ghosted sales estimates through scheduled email and SMS communications.

## Overview

This n8n automation sends escalating follow-up messages to prospects who haven't responded to quotes. It supports email, SMS, or both channels, with configurable timing and opt-out handling.

## Features

- **Multi-channel outreach**: Email (Gmail), SMS (Twilio), or both
- **Escalating message templates**: 4 progressively urgent messages per channel
- **Configurable timing**: Demo mode (minutes) vs Production mode (days)
- **Opt-out handling**: STOP keyword detection + manual sheet toggle
- **Reply detection**: Auto-updates status, notifies owner via email and SMS
- **Google Sheets backend**: Easy data management, no database required

## Architecture

```
┌─────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│  Google Sheets  │────▶│  Main Follow-up  │────▶│  Gmail/Twilio   │
│  (Config/Data)  │     │    Workflow      │     │  (Send Msgs)    │
└─────────────────┘     └──────────────────┘     └─────────────────┘
                                                          │
                        ┌──────────────────┐              │
                        │  Inbound SMS     │◀─────────────┘
                        │    Handler       │     (Customer replies)
                        └──────────────────┘
```

## Workflows

| Workflow | Purpose | Trigger |
|----------|---------|---------|
| Main Follow-up | Sends scheduled follow-ups | Schedule (1 min polling) |
| Inbound SMS Handler | Processes replies and opt-outs | Twilio webhook |

## Quick Start

1. Import workflows into n8n
2. Create Google Sheet with Config and Quote_Followups tabs
3. Configure credentials (Google Sheets, Gmail, Twilio)
4. Add test quote with `next_follow_up` set to past date
5. Run workflow

See [docs/SETUP.md](docs/SETUP.md) for detailed instructions.

## Project Structure

```
dead-estimate-revival/
├── README.md                           # This file
├── docs/
│   ├── project-context.md              # Project requirements and context
│   ├── SETUP.md                        # Full setup documentation
│   ├── TESTING.md                      # Testing procedures and checklist
│   └── system-prompt.md                # AI assistant instructions
├── scratchpad/
│   └── scratchpad.md                   # Working notes
├── secrets/
│   └── secrets.md                      # API keys and credentials
├── templates/
│   ├── config_template.csv             # Config sheet template
│   └── quote_followups_template.csv    # Data sheet template
└── workflows/
    ├── main-followup-workflow.json     # Primary automation
    └── inbound-sms-handler.json        # Reply/opt-out handler
```

## Configuration

All settings are managed in the Google Sheet Config tab:

| Setting | Description | Example |
|---------|-------------|---------|
| mode | demo or production | demo |
| max_attempts | Follow-ups before marking lost | 4 |
| sender_name | Name in email signature | John Smith |
| sender_email | Reply-to address | john@company.com |
| sender_phone | Phone in email signature | (555) 123-4567 |
| twilio_phone_number | Twilio sender number | +15551234567 |

### Follow-up Intervals

**Demo Mode** (minutes):
- After 1st: 1 min
- After 2nd: 2 min
- After 3rd: 3 min
- After 4th: 4 min

**Production Mode** (days):
- After 1st: 5 days
- After 2nd: 10 days
- After 3rd: 14 days
- After 4th: 30 days

## Data Requirements

Each quote requires:

| Field | Required | Example |
|-------|----------|---------|
| quote_id | Yes | Q-2024-001 |
| customer_name | Yes | Jane Smith |
| customer_email | Yes* | jane@example.com |
| customer_phone | Yes* | +15551234567 |
| quote_amount | Yes | $2,500 |
| quote_date | Yes | 2024-01-15 |
| preferred_channel | Yes | email/sms/both |
| follow_up_count | Yes | 0 |
| next_follow_up | Yes | Past ISO timestamp |
| status | Yes | active |
| opted_out | Yes | FALSE |

*Email required for email/both channels. Phone required for sms/both channels.

## Requirements

- n8n instance (self-hosted or cloud)
- Google Workspace account
- Twilio account (for SMS)
- Gmail OAuth credentials

## License

Internal use only.
