# Actserv AI Client Booking — n8n Setup Guide

> Import, configure, and run the booking automation workflow in n8n

![n8n](https://img.shields.io/badge/n8n-workflow-orange) ![Mistral AI](https://img.shields.io/badge/AI-Mistral-blue) ![Airtable](https://img.shields.io/badge/database-Airtable-green)


## Prerequisites

Make sure you have the following before starting:

- n8n instance (cloud or self-hosted)
- Mistral AI API key
- Google account (Sheets + Gmail + Calendar access)
- Airtable account with a base ready
- The workflow JSON file: `actserv_booking.json`

---

## Step-by-Step Setup

### Step 1 — Import the workflow JSON

In your n8n dashboard:

1. Click **New Workflow**
2. Open the top-right menu **(⋯)** → **Import from file**
3. Select the `actserv_booking.json` file

The full 11-step workflow will load automatically. All nodes, connections, and settings are embedded in the JSON — you won't need to rebuild anything manually.

---

### Step 2 — Connect your credentials

n8n will flag nodes with missing credentials. Click each flagged node and connect the following:

| Node | Credential needed |
|------|------------------|
| Mistral AI | Mistral API key |
| Google Sheets | Google OAuth2 |
| Airtable | Airtable Personal Access Token |
| Gmail | Google OAuth2 (same or separate account) |
| Google Calendar | Google OAuth2 |

---

### Step 3 — Configure your Airtable base

Open the Airtable node **(Step 8)** and set your **Base ID** and **Table name**.

Make sure your Airtable table has these columns:

| Column | Type |
|--------|------|
| `name` | Single line text |
| `phone` | Single line text *(must be text, not number)* |
| `email` | Email or single line text |
| `service` | Single line text |
| `preferred_date` | Date or single line text |
| `timestamp` | Single line text or Date |

> ⚠️ **Important:** `phone` must be stored as **text/string** — not a number field. The `+` prefix in `+254...` will cause an error if the field type is set to Number.

---

### Step 4 — Configure your Google Sheet

Open the Google Sheets node **(Step 7)**. Select your spreadsheet and sheet tab.

Make sure the header row matches exactly:

```
name | phone | email | service | preferred_date | timestamp
```

---

### Step 5 — Set up the Google Calendar event

Open **Step 10 (Client Inductory Meeting)** and confirm the start and end time mapping:

```javascript
// Start time
{{ $('Step 8: Updating New Client Info to our Client Database').item.json.fields.preferred_date }}T09:00:00

// End time
{{ $('Step 8: Updating New Client Info to our Client Database').item.json.fields.preferred_date }}T10:00:00
```

> The date field must be in `YYYY-MM-DD` format for the calendar event to work correctly.

---

### Step 6 — Test with your own details

Click **Test Workflow**, then open the chat trigger **(Step 1)** and send:

```
Hi, I'd like to book an appointment.
```

The bot will ask for your details one at a time. Use these example values:

| Field | Example value |
|-------|--------------|
| Name | John Doe |
| Phone | +254712345678 |
| Email | johndoe@gmail.com |
| Service | Health Insurance |
| Preferred Date | 20th April 2026 |

> ✅ Check each node's output panel after the run to confirm data is flowing correctly through all 11 steps.

---

### Step 7 — Activate the workflow

Once testing passes, toggle the workflow to **Active** using the switch in the top-right of the editor.

The system will now handle live client bookings automatically.

---

## Workflow Overview

| Step | Node | Action |
|------|------|--------|
| 1 | Chat Trigger | Client reaches out via chat |
| 2 | AI Agent (Mistral) | Extracts and structures client data |
| 3 | Set Node | Passes client info as rows/items |
| 4 | Code / Set Node | Adds timestamp to the record |
| 5 | Switch (Rules) | Routes based on insurance type |
| 6 | Manual Assignment | Assigns staff to the client |
| 7 | Google Sheets | Logs record (append or update) |
| 8 | Airtable | Upserts record to client database |
| 9 | Gmail | Sends welcome email to client |
| 10 | Google Calendar | Creates inductory meeting event |
| 11 | Gmail / Messaging | Notifies assigned staff |

---

## Expected JSON Output

After a successful booking, the system produces:

```json
{
  "name": "John Doe",
  "phone": "+254712345678",
  "email": "johndoe@example.com",
  "service": "Life Insurance",
  "preferred_date": "2026-04-20",
  "timestamp": "2026-04-13T10:00:00.000Z"
}
```

---

## Common Issues

| Error | Fix |
|-------|-----|
| Phone field error in Airtable | Change the phone column type to **Single line text** |
| Yellow triangle on a node | Click it — check for null fields or mismatched column names |
| Calendar event fails | Ensure date is `YYYY-MM-DD` and time is appended as `T09:00:00` |
| AI not extracting fields | Check Mistral API key and Output Parser schema matches field names |
| Email not sending | Re-authenticate the Gmail OAuth2 credential |
| Data not saving to Sheets | Verify header row matches field names exactly (case-sensitive) |

---

## Integrations Used

- **Mistral AI** — Conversational AI and data extraction
- **Simple Memory** — Maintains conversation context across turns
- **Google Sheets** — Activity log and record backup
- **Airtable** — Primary structured client database
- **Gmail** — Client confirmation and staff notification emails
- **Google Calendar** — Appointment scheduling

---

*Actserv Insurance Solutions — Confidential. For internal use only.*
