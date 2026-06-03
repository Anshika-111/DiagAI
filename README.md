# DiagnosticAI 🩺

> AI-powered medical scribe and patient portal — built at Axxess Hackathon 2026 in 24 hours.

Doctors record consultations and get AI-generated clinical notes, ICD-10 codes, and medication suggestions instantly. Patients log in to a clean portal to view their visit summary, test results, and appointments — all in plain language.

---

## Features

**Provider Portal**
- 🎙️ Live audio recording with 8-second chunked transcription via Gemini
- 🤖 AI-generated SOAP notes, diagnosis, ICD-10 code, and medication suggestions
- ✏️ Editable summary before publishing to the patient
- 📅 Follow-up appointment scheduling

**Patient Portal**
- 📋 Visit summary in plain, non-clinical language
- 💊 Prescribed medications with dosage and reason
- 🧪 Test results with doctor notes and severity badges
- 📆 Appointment booking and management

---

## Architecture

```
Browser
  ├── /provider/record    → streams audio chunks every 8s
  │                             → POST /api/transcribe → Gemini (audio → text)
  ├── /provider/loading   → triggers analysis
  │                             → POST /api/analyze   → Gemini (transcript → JSON)
  │                             → returns: diagnosis, ICD-10, SOAP note, medications, patient summary
  ├── /provider/summary   → doctor reviews & edits
  │                             → POST /api/visits    → Supabase (saves full visit record)
  └── /patient/dashboard  → GET /api/visits           → Supabase (reads visit by patient)
```

---

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | Next.js 16 (App Router), TypeScript |
| Styling | Tailwind CSS |
| AI — Transcription | Google Gemini 1.5 Flash (base64 audio → transcript) |
| AI — Clinical Analysis | Google Gemini 1.5 Flash (transcript → structured JSON) |
| Database | Supabase (PostgreSQL) |
| Deployment | Vercel-ready |

---

## API Routes

All routes built with Next.js App Router. Input validation and error handling on every endpoint.

| Route | Method | What it does |
|---|---|---|
| `/api/transcribe` | `POST` | Accepts `audio/webm` chunk via FormData, converts to base64, sends to Gemini, returns raw transcript text. Skips chunks under 1KB to avoid silent-audio API waste. |
| `/api/analyze` | `POST` | Accepts full transcript + optional patient history. Sends structured clinical prompt to Gemini with `json_object` response format. Returns diagnosis, ICD-10 code, SOAP note, medications, and a plain-English patient summary. |
| `/api/patients` | `GET` | Returns all patients from Supabase ordered by name. |
| `/api/patients` | `POST` | Registers a new patient record in Supabase. |
| `/api/visits` | `POST` | Saves a completed visit with full clinical data — transcript, diagnosis, ICD code, medications (JSONB), SOAP note, patient summary, follow-up date. |
| `/api/visits` | `GET` | Returns visit history, optionally filtered by `patientId` query param. |

---

## Database Schema

```sql
create table patients (
  id uuid primary key default gen_random_uuid(),
  name text not null,
  date_of_birth date,
  created_at timestamptz default now()
);

create table visits (
  id uuid primary key default gen_random_uuid(),
  patient_id uuid references patients(id),
  transcript text,
  chief_complaint text,
  symptoms jsonb,
  diagnosis text,
  icd_code text,
  medications jsonb,
  clinical_note text,
  patient_summary text,
  follow_up_date date,
  status text default 'completed',
  created_at timestamptz default now()
);
```

---

## Setup

```bash
git clone https://github.com/Anshika-111/DiagAI.git
cd DiagAI
npm install
npm run dev
```

Open [http://localhost:3000](http://localhost:3000)

### Environment Variables

```bash
GEMINI_API_KEY=                  # aistudio.google.com → Get API Key (free)
NEXT_PUBLIC_SUPABASE_URL=        # Supabase project → Settings → API → Project URL
NEXT_PUBLIC_SUPABASE_ANON_KEY=   # Supabase project → Settings → API → anon public key
```

Run the schema SQL above in your Supabase SQL Editor to create the tables.

---

## Team

| Contributor | Role |
|---|---|
| **Anshika** ([@Anshika-111](https://github.com/Anshika-111)) | Backend — API routes, Gemini integration, Supabase schema & queries, TypeScript architecture |
| **Avneet** ([@avneetkaur17](https://github.com/avneetkaur17)) | Frontend — Provider & Patient portal UI, React component architecture, routing |
