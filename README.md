# Dead Estimate Revival

Automated follow-up system for reviving cold/ghosted estimates via email and SMS.

---

## Quick Start

1. **Create Google Sheet** using templates in `/templates/`
2. **Import workflows** from `/workflows/` into n8n
3. **Configure credentials** (Google Sheets, Gmail, Twilio)
4. **Set up Twilio webhook** pointing to your n8n instance
5. **Activate workflows**

See `/docs/SETUP.md` for detailed instructions.

---

## Project Structure

```
dead-estimate-revival/
├── README.md                           # This file
├── docs/
│   └── SETUP.md                        # Full setup documentation
|-- scratchpad
|   |-- scratchpad.md                   # Scratchpad for copy/paste etc. 
|-- secrets
|   |-- secrets.md                      # API keys etc. needed
├── templates/
│   ├── config_template.csv             # Config sheet template
│   └── quote_followups_template.csv    # Data sheet template
└── workflows/
    ├── main-followup-workflow.json     # Primary automation
    └── inbound-sms-handler.json        # Reply/opt-out handler
```

---

## Features

- **Multi-channel outreach**: Email, SMS, or both
- **Configurable cadence**: Demo mode (minutes) or Production mode (days)
- **Opt-out handling**: STOP keyword + manual sheet toggle
- **Reply detection**: Auto-updates status, notifies owner
- **Template progression**: 4 escalating message templates
- **Modular design**: Easy to extend or customize

---

## Configuration

### Cadence Timing

| Attempt | Demo Mode | Production Mode |
|---------|-----------|-----------------|
| 1 | 2 min | 2 days |
| 2 | 5 min | 5 days |
| 3 | 10 min | 10 days |
| 4 | 15 min | 14 days |

### End Conditions

Automation stops when:
- Customer replies (any channel)
- Status changed to `won`, `lost`, or `opted_out`
- Max attempts reached (default: 4)

---

## Requirements

- n8n (self-hosted or cloud)
- Google Workspace (Sheets + Gmail)
- Twilio account with SMS-enabled number

---

## License

Internal use only. Customize as needed for client deployments.
