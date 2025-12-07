# Project Context: Dead Estimate Revival

## Project Overview

| Field | Value |
|-------|-------|
| Project Name | Dead Estimate Revival |
| Version | 1.0 |
| Status | Complete |
| Platform | n8n |
| Last Updated | 2024-12-07 |

## Stakeholders

### 1. Customer (Sales Manager/Salesperson)

**Problem Statement**: Estimates issued but prospects ghost. Manual follow-up is inconsistent and time-consuming.

**User Story**: As a sales manager, I want an automated customer follow-up system so that I can revive cold leads, shorten the sales cycle, and ensure consistency without manual effort.

**Acceptance Criteria**:
- [ ] System sends follow-up messages automatically
- [ ] Multiple channels supported (email, SMS)
- [ ] Customer can opt-out
- [ ] Owner notified when customer responds

### 2. Project Architect

**User Story**: As a project architect, I want a simple, reusable automation that is quick to deploy and modular, so it can be adapted for different clients without major changes.

**Acceptance Criteria**:
- [ ] Single Google Sheet for all configuration
- [ ] Configurable timing intervals
- [ ] Swappable message templates
- [ ] Channel selection per customer

### 3. Salesperson (Demo Requirements)

**User Story**: As a salesperson, I want an automation I can demo easily (on the fly) so I can show value and flexibility at demo time.

**Acceptance Criteria**:
- [ ] Demo mode with minute-based intervals
- [ ] Production mode with day-based intervals
- [ ] Mode switchable via config setting
- [ ] All 4 message templates testable in under 10 minutes

## Technical Decisions

### Data Storage
**Decision**: Google Sheets
**Rationale**: Easy to demo, no database setup, familiar to clients, supports real-time updates

### Communication Channels
| Channel | Provider | Notes |
|---------|----------|-------|
| Email | Gmail | OAuth2 integration |
| SMS | Twilio | Native n8n node, webhook support |

### Workflow Architecture
**Decision**: Schedule-based polling (not event-driven)
**Rationale**: 
- Simpler than Google Sheets trigger
- More reliable for batch processing
- Single workflow handles all follow-up logic
- Configurable polling interval

### Message Templates
**Decision**: 4 escalating templates per channel
**Rationale**: Industry standard follow-up sequence (friendly > value > urgency > final)

## Constraints

### Technical
- n8n self-hosted or cloud
- Google Workspace account required
- Twilio account required for SMS
- Phone numbers must be E.164 format (+15551234567)

### Business
- Max 4 follow-up attempts before marking as lost
- Customer must be able to opt-out via SMS reply
- Owner must be notified of all customer responses

### Scope Exclusions
- Voice/phone call automation (future enhancement)
- CRM integration (future enhancement)
- Multi-tenant support (future enhancement)

## Data Model

### Config Sheet

| key | value | description |
|-----|-------|-------------|
| mode | demo/production | Controls timing intervals |
| max_attempts | 4 | Follow-ups before lost |
| sender_name | string | Email signature name |
| sender_email | email | Reply-to address |
| sender_phone | string | Email signature phone |
| twilio_phone_number | +E.164 | Twilio sender number |

### Quote_Followups Sheet

| Column | Type | Required | Description |
|--------|------|----------|-------------|
| quote_id | string | Yes | Unique identifier |
| customer_name | string | Yes | Contact name |
| customer_email | email | Conditional | Required for email/both |
| customer_phone | E.164 | Conditional | Required for sms/both |
| quote_amount | number | Yes | Dollar value |
| quote_date | date | Yes | When quote was sent |
| preferred_channel | enum | Yes | email/sms/both |
| follow_up_count | number | Yes | Current attempt (0-4) |
| last_follow_up | ISO datetime | Auto | Last send timestamp |
| next_follow_up | ISO datetime | Yes | When to send next |
| status | enum | Yes | active/replied/won/lost/opted_out |
| opted_out | boolean | Yes | TRUE to stop messages |
| notes | string | No | Additional context |

## Workflow Logic

### Main Follow-up Workflow

```
Schedule Trigger (1 min)
    ↓
Get Config → Parse Config → Single Config Item
    ↓
Get Active Quotes → Filter Valid Quotes → Filter Due Quotes
    ↓
Route by Channel
    ├── email → Email Template Selector → Templates 1-4 → Send Email
    ├── sms → SMS Template Selector → Templates 1-4 → Send SMS
    └── both → Both branches execute
    ↓
Calculate Next Follow-up → Update Sheet
```

### Inbound SMS Handler

```
Twilio Trigger (webhook)
    ↓
Parse Incoming SMS (extract phone, body, check STOP)
    ↓
Find Customer by Phone → Customer Found?
    ↓
Opt-Out or Reply?
    ├── STOP → Mark Opted Out → Send Confirmation → Notify Owner SMS
    └── Reply → Mark as Replied → Notify Owner Email → Notify Owner SMS
```

## Message Templates

### Email Templates

| Attempt | Subject | Tone |
|---------|---------|------|
| 1 | Quick follow-up on your estimate | Friendly check-in |
| 2 | Still thinking it over? | Value reminder |
| 3 | Before this slips away | Urgency |
| 4 | Closing the loop | Final / graceful close |

### SMS Templates

| Attempt | Focus | Notes |
|---------|-------|-------|
| 1 | Check-in | Includes opt-out notice |
| 2 | Availability | Offer to discuss |
| 3 | Urgency | Time-sensitive |
| 4 | Closing | Door open for future |

## Timing Configuration

| Attempt | Demo Mode | Production Mode |
|---------|-----------|-----------------|
| After 1st | 1 minute | 5 days |
| After 2nd | 2 minutes | 10 days |
| After 3rd | 3 minutes | 14 days |
| After 4th | 4 minutes | 30 days |

## Known Issues / Lessons Learned

### Issue: Empty rows cause duplicate sends
**Cause**: Google Sheets data validation applied to entire columns creates empty rows
**Solution**: Limit validation to specific ranges (e.g., G2:G100)

### Issue: Email formatting - wall of text
**Cause**: Gmail node ignores \n newlines
**Solution**: Use HTML format with `<br>` tags

### Issue: Data type errors in filters
**Cause**: Google Sheets returns all values as strings
**Solution**: Use `Number()` wrapper and enable loose type validation

### Issue: Merge node blocks when only one channel active
**Cause**: Merge node waits for both inputs
**Solution**: Remove Merge node; connect both channels directly to Calculate node

### Issue: Config duplication causes multiple sends
**Cause**: Parse Config outputs multiple items (one per config row)
**Solution**: Add Limit node after Parse Config to output single item

## Future Enhancements

- [ ] Voice/phone call integration
- [ ] CRM sync (Salesforce, HubSpot)
- [ ] Email reply detection
- [ ] Custom template per customer
- [ ] A/B testing for message effectiveness
- [ ] Analytics dashboard
- [ ] Multi-tenant support

## References

- n8n Documentation: https://docs.n8n.io
- Twilio SMS: https://www.twilio.com/docs/sms
- Google Sheets API: https://developers.google.com/sheets/api
