# Fatsoma Automation Challenge — n8n Workflow

## Setup

### Prerequisites
- n8n (cloud or local) — see [n8n docs](https://docs.n8n.io/hosting/)
- OpenAI API key
- Google account with Sheets access 

### Credentials
Configure and assign credentials in n8n for:
- **OpenAI API** → OpenAI Chat Model1
- **Google Sheets OAuth2** → Read Existing Records, Append New Record, Update Existing Record

### Google Sheet
Create a sheet with these headers in row 1:
```
event_id | event_name | promoter_name | email | phone | instagram_handle | website | extraction_timestamp | validation_status
```

### Run
Import `FATSOMA_INTERVIEW.json`, click **Test Workflow**.
---

## Where output goes

All records land in Google Sheets regardless of outcome. The `validation_status` column is the audit trail:

| Status | Meaning |
|---|---|
| `OK` | All four contact fields extracted |
| `PARTIAL` | At least one field extracted, some missing |
| `NO DATA` | LLM returned nothing extractable |

---

## Error handling

| Failure | Behaviour | Alert |
|---|---|---|
| LLM API/credential failure | Routes to error pin | Slack message |
| LLM returns no data | Writes `NO DATA` row to sheet | None - sheet row is the record |
| Sheets read failure | Continues via `continueRegularOutput` | None |
| Sheets write failure | Routes to error via `continueErrorOutput` | Slack message |

---

## Assumptions

- Assumed data comes in consistent format and does not need validation or formatting at input

- Assumed upsert/append behaviour – currently only upserts on event ID (could be and/or email/name)

- JSON structure is enforced by the structured output parser rather than a manual validation step - a lightweight Code node could be added for content-level checks (email format, URL normalisation) but was considered overkill for this brief.

- Assumed promoters with no bio do not need flagged separately in validation, flagged as "NO DATA"

- Event ID is assumed to be unique and a stable key for idempotency

- Sequential processing is used in this example but at higher volumes may need rate limiting to avoid hitting LLM/Sheets API write quotas

---

## Design Decisions

* Missing fields return empty strings, not nulls.

* I included slack error messages but disabled these, just to show where I would look to put them, and I think Slack is ideal for key failure messages.
 
* All four contact fields are required by the JSON schema — the LLM is instructed to return an empty string for any field it cannot find. This keeps the Google Sheets schema consistent across every row and avoids type errors or blank cell handling issues downstream. A null value would require conditional handling at every point the data is consumed; an empty string is unambiguous and safe.

