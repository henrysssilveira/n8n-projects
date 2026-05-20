# IT Ticket Automation via WhatsApp

> n8n workflow for automatically opening, tracking, and closing IT tickets in Aranda ASMS, integrated with WhatsApp via Evolution API.

---

## Overview

AVBot is a technical support bot that monitors WhatsApp groups and allows users to open, track, and close IT tickets directly from the chat — no need to access any external system. All communication is bidirectional: the bot replies in the same group in up to 3 languages (🇧🇷 Portuguese, 🇪🇸 Spanish, 🇺🇸 English).

---

## Architecture

```
WhatsApp (Evolution API)
        │
        ▼
   n8n Webhook
        │
        ▼
 Message Classification
  ┌─────┴──────┐
  │            │
 Text        Media
  │       (Image/Audio/PDF/Video)
  │            │ AI Processing (GPT-4o / Gemini / Whisper)
  │            │
  └────────────┤
               ▼
     Intent Routing
   ┌──────┬──────┬──────┐
   │      │      │      │
#open# #close# Collecting Ignore
   │      │      │
   ▼      ▼      ▼
Aranda  Aranda Supabase
ASMS   ASMS    (log)
   │      │
   ▼      ▼
 WhatsApp (group reply)
```

---

## Features

### 1. Open a Ticket (`#open#`)
- User types `#open#` in the group
- Bot checks if there is already an active ticket for that group
- If none exists: creates the ticket in Aranda ASMS and saves it to Supabase
- Replies in the group with the generated ticket number
- If one already exists: notifies the user and displays the active ticket number

### 2. Close a Ticket (`#close#`)
- User types `#close#` in the group
- Bot retrieves the active ticket from Supabase
- AI analyzes the full message history and automatically generates:
  - **Description** — what happened, symptoms, and impact
  - **Resolution** — how it was resolved and next steps
- Closes the ticket in Aranda ASMS with the AI-generated summary
- Updates the status in Supabase to `CLOSED`
- Notifies the group with a closure confirmation

### 3. Message Collection (`Collecting`)
- Every message sent while a ticket is active is logged in Supabase
- Media files (images, PDFs, videos) are automatically attached to the active ticket in Aranda

### 4. Media Processing
| Type | Processing |
|------|------------|
| 🖼️ Image | GPT-4o describes the image in detail |
| 🎙️ Audio | Whisper transcribes the audio to text (pt) |
| 📄 PDF / Document | Gemini 2.5 Flash analyzes and extracts content as Markdown |
| 🎥 Video | Attached directly to the ticket |

### 5. Multilingual Support
Language is automatically detected by `groupJid` (WhatsApp group ID):

| Group | Language |
|-------|----------|
| Brazil groups | 🇧🇷 Portuguese |
| Mexico groups | 🇪🇸 Spanish |
| USA groups | 🇺🇸 English |

---

## 🔗 Integrations

| Service | Role |
|---------|------|
| **Evolution API** (`evo.imberabrasil.com`) | WhatsApp gateway — receives and sends messages |
| **Aranda ASMS** (`aeritek.arandasoft.com`) | Ticketing system — creation, update, and closure |
| **Supabase** (`olxmnkdlhvkqmnkoszlj.supabase.co`) | Database — ticket and message history |
| **OpenAI GPT-4o** | Image analysis |
| **OpenAI Whisper** | Audio transcription |
| **OpenAI o4-mini** | AI-generated description and resolution summary |
| **Google Gemini 2.5 Flash Lite** | PDF document analysis |

---

## 🗄️ Database Structure (Supabase)

### Schema: `TI - AV - Chamados Whatsapp`

**Table `chamados`**
| Field | Type | Description |
|-------|------|-------------|
| `groupJid` | string | WhatsApp group ID |
| `groupKey` | string | Group number |
| `status` | string | `COLLECTING` / `CERRADO` |
| `openedBy` | string | Phone of who opened the ticket |
| `openedByName` | string | Name of who opened the ticket |
| `openedAt` | timestamp | Opening date/time |
| `closedBy` | string | Phone of who closed the ticket |
| `closedAt` | timestamp | Closing date/time |
| `arandaTicketId` | string | Aranda ticket ID (e.g. RF-TI-XXXXX) |
| `case-id` | string | Aranda internal ID |
| `lang` | string | `pt` / `es` / `en` |
| `log` | JSON | Action history |

**Table `mensagens_whatsapp`**
| Field | Type | Description |
|-------|------|-------------|
| `whatsapp_id` | string | Message ID |
| `group_jid` | string | Group ID |
| `sender_name` | string | Sender name |
| `sender_phone` | string | Sender phone |
| `message_body` | string | Message content |
| `message_type` | string | Message type |
| `aranda_ticket_id` | string | Linked ticket |
| `timestamp_whatsapp` | timestamp | Message date/time |

---

## Required Credentials

| Credential | n8n Name | Usage |
|------------|----------|-------|
| OpenAI API Key | `OPENAI - ti.admbr` | GPT-4o, Whisper, o4-mini |
| Aranda Header Auth | `ARANDA - HEADER` | Aranda API authentication |
| Evolution API Header | `EVOL - HEADER` | Sending WhatsApp messages |
| Supabase API | `SUPABASE - IMBERA PRODUCTION` | Database access |
| Google Gemini API | `GEMINI - Automacao` | Document analysis |

---

## Webhook Endpoint

```
POST https://flows.imberabrasil.com/webhook/{endpoint}
```

Configure in the Evolution API as the destination for `messages.upsert` events on the `BOTIT` instance.

---

## Flow Summary

```
Message received
       │
       ▼
Classify media type (text / image / audio / PDF / video)
       │
       ▼
Process media with AI (if applicable)
       │
       ▼
Look up active ticket in Supabase
       │
       ▼
Determine intent:
  • #open#      → Create ticket in Aranda + notify group
  • #close#     → AI summary + close in Aranda + notify group
  • Collecting  → Save message + attach media to ticket
  • Bot itself  → Ignore (prevent loop)
  • No ticket   → Guide user to open one
```

---

## How to Add New Groups

Edit the **"Tratamento de Informações e direcionamento de chaves"** node and add the group to the language map:

```javascript
const GROUP_LANG_MAP = {
  "120363xxxxxxxx1@g.us": "pt",    // Brazil
  "120363425589076485@g.us": "es", // Mexico
  "120363424210222326@g.us": "en", // USA
  "YOUR_NEW_GROUP@g.us": "pt",     // ← add here
};
```

---

## Notes

- The bot automatically ignores messages sent by itself (detects `Suporte TI - Imbera Brasil` in the sender name)
- Each group can only have **one active ticket at a time**
- The AI-generated description and resolution are always written in the group's predominant language
- Aranda tickets use `stateId: 214` to move to "closing" and `217` for the final "closed" state

---
