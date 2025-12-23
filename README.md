# Medari - AI Health Companion

> An intelligent health companion that helps users prepare for doctor visits through conversational symptom tracking, clinical research integration, and structured note generation.

![Version](https://img.shields.io/badge/version-1.0.0-blue)
![License](https://img.shields.io/badge/license-MIT-green)
![Platform](https://img.shields.io/badge/platform-iOS%20%7C%20Android%20%7C%20Web-lightgrey)
![Python](https://img.shields.io/badge/python-3.11+-blue)
![Node](https://img.shields.io/badge/node-18+-green)

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Architecture](#architecture)
- [Tech Stack](#tech-stack)
- [Quick Start](#quick-start)
- [API Documentation](#api-documentation)
- [Project Structure](#project-structure)
- [Testing](#testing)
- [Roadmap](#roadmap)
- [Contributing](#contributing)

---

## Overview

Medari transforms anxious, scattered health concerns into clear, organized Dr. Prep Notes through natural conversation. Chat about your symptoms, and Medari extracts the important details to help you communicate better with your healthcare provider.

### The Core Loop

```
User describes symptoms
        |
        v
AI asks follow-up questions
        |
        v
Symptoms extracted automatically (invisible to user)
        |
        v
Clinical research fetched (PubMed + ClinicalTrials.gov)
        |
        v
"Create Dr. Prep Notes" button appears
        |
        v
Structured summary generated with citations
        |
        v
PDF download / Share with doctor
```

**Important:** Medari is NOT a diagnostic tool. It helps organize symptoms for discussion with healthcare providers.

---

## Features

### Phase 1: Core Loop - COMPLETE
- [x] Conversational chat interface with empathetic AI
- [x] Automatic symptom extraction (severity, frequency, duration, triggers)
- [x] Pattern identification (time of day, activities, food, stress)
- [x] Dr. Prep Notes generation with structured summary
- [x] PDF export and native sharing
- [x] Conversation history with persistence
- [x] Local data storage (AsyncStorage)

### Phase 2: Clinical Intelligence - COMPLETE
- [x] PubMed research integration (systematic reviews, meta-analyses)
- [x] ClinicalTrials.gov integration (active trials)
- [x] Inline citations in AI responses `[Author et al., Year]`
- [x] Research context in Dr. Prep Notes
- [x] Dedicated `/api/research` endpoint

### Phase 2.5: UX Improvements - COMPLETE
- [x] AI prioritizes critical questions (severity 1-10, duration, frequency)
- [x] Visual warnings for missing data in Dr. Prep Notes
- [x] Better handling of constant/chronic symptoms
- [x] Specific doctor questions based on actual symptoms
- [x] Enhanced color system for semantic feedback

### Phase 3: Polish - PLANNED
- [ ] Secure sharing with doctors (unique links)
- [ ] Pattern insights over time ("headaches up 40% this month")
- [ ] Multi-symptom correlation analysis
- [ ] Push notification reminders
- [ ] User authentication and cloud sync

---

## Architecture

### System Diagram

```
+---------------------------------------------------------------------+
|                        MEDARI ARCHITECTURE                          |
+---------------------------------------------------------------------+
|                                                                     |
|  +---------------------------------------------------------------+  |
|  |                 MOBILE APP (React Native/Expo)                |  |
|  |                                                               |  |
|  |  +-------------+  +-------------+  +-------------+            |  |
|  |  | Chat Screen |  | Prep Notes  |  |  History    |            |  |
|  |  |  (index)    |  |   Screen    |  |   Screen    |            |  |
|  |  +------+------+  +------+------+  +------+------+            |  |
|  |         |                |                |                   |  |
|  |         +----------------+----------------+                   |  |
|  |                          |                                    |  |
|  |                          v                                    |  |
|  |              +-----------------------+                        |  |
|  |              |   Zustand Store       |                        |  |
|  |              |   (+ AsyncStorage)    |                        |  |
|  |              +-----------+-----------+                        |  |
|  |                          |                                    |  |
|  |                          v                                    |  |
|  |              +-----------------------+                        |  |
|  |              |     API Client        |                        |  |
|  |              |  (REST/JSON fetch)    |                        |  |
|  |              +-----------+-----------+                        |  |
|  |                          |                                    |  |
|  +--------------------------|------------------------------------+  |
|                             |                                       |
|                      HTTP/JSON                                      |
|                             |                                       |
|  +--------------------------|------------------------------------+  |
|  |                          v                                    |  |
|  |              BACKEND API (FastAPI/Python)                     |  |
|  |                                                               |  |
|  |  +---------------------------------------------------------+  |  |
|  |  |                   Endpoints                             |  |  |
|  |  |  * GET  /health              - Health check             |  |  |
|  |  |  * POST /api/chat            - Process message          |  |  |
|  |  |  * POST /api/chat-with-research - Chat + citations      |  |  |
|  |  |  * POST /api/generate-notes  - Create Dr. Prep Notes    |  |  |
|  |  |  * POST /api/research        - Fetch research           |  |  |
|  |  +---------------------------------------------------------+  |  |
|  |                          |                                    |  |
|  |         +----------------+----------------+                   |  |
|  |         v                v                v                   |  |
|  |  +-----------+   +---------------+  +--------------+          |  |
|  |  |  OpenAI   |   |    PubMed     |  | ClinicalTri- |          |  |
|  |  |   API     |   |  E-utilities  |  |  als.gov API |          |  |
|  |  |  (GPT-4o) |   |    (NCBI)     |  |              |          |  |
|  |  +-----------+   +---------------+  +--------------+          |  |
|  |                                                               |  |
|  +---------------------------------------------------------------+  |
|                                                                     |
+---------------------------------------------------------------------+
```

### Data Flow Diagrams

#### Chat Flow
```
+------------------------------------------------------------------+
|                        CHAT FLOW                                  |
+------------------------------------------------------------------+
|                                                                  |
|  User Input --> Store --> API Client --> Backend                 |
|       |                                      |                   |
|       |                    +-----------------+------------------+|
|       |                    |                                    ||
|       |                    v                                    v|
|       |              OpenAI API                          OpenAI API
|       |           (Conversation)                       (Extraction)
|       |                    |                                    ||
|       |                    +-------------------+----------------+|
|       |                                        |                 |
|       |                                        v                 |
|       |                         Response + ExtractedData         |
|       |                                        |                 |
|       <----------------------------------------+                 |
|                                                                  |
+------------------------------------------------------------------+
```

#### Notes Generation Flow
```
+------------------------------------------------------------------+
|                    NOTES GENERATION FLOW                         |
+------------------------------------------------------------------+
|                                                                  |
|  "Create Notes" --> API --> Backend                              |
|                              |                                   |
|            +-----------------+-----------------+                  |
|            |                 |                 |                  |
|            v                 v                 v                  |
|       OpenAI API       PubMed API      ClinicalTrials           |
|     (Notes Gen)        (Research)         (Trials)               |
|            |                 |                 |                  |
|            +-----------------+-----------------+                  |
|                              |                                   |
|                              v                                   |
|                    DrPrepNotes + ClinicalContext                 |
|                              |                                   |
|                              v                                   |
|                     Display / PDF Export                         |
|                                                                  |
+------------------------------------------------------------------+
```

---

## Tech Stack

### Frontend (Mobile App)

| Technology | Version | Purpose |
|------------|---------|---------|
| React Native | 0.73.2 | Cross-platform mobile framework |
| Expo | 50.0.0 | Development platform & tools |
| TypeScript | 5.3.0 | Type safety |
| Expo Router | 3.4.0 | File-based navigation |
| Zustand | 4.5.0 | State management |
| AsyncStorage | 1.21.0 | Local persistence |
| date-fns | 3.3.0 | Date formatting |
| expo-print | 12.6.0 | PDF generation |
| expo-sharing | 11.10.0 | Native sharing |
| react-native-web | 0.19.6 | Web support |

### Backend (API Server)

| Technology | Version | Purpose |
|------------|---------|---------|
| Python | 3.11+ | Runtime |
| FastAPI | 0.127.0 | Web framework |
| Uvicorn | 0.40.0 | ASGI server |
| OpenAI SDK | 2.14.0 | GPT-4o AI integration |
| Pydantic | 2.12.5 | Data validation |
| pymed | 0.8.9 | PubMed API client |
| pytrials | 1.0.0 | ClinicalTrials.gov client |
| python-dotenv | 1.2.1 | Environment management |

### AI Model

| Model | ID | Usage |
|-------|----|----|
| GPT-4o | gpt-4o | Chat responses, symptom extraction, notes generation |

---

## Quick Start

### Prerequisites

- Node.js 18+ and npm
- Python 3.11+
- OpenAI API key ([Get one here](https://platform.openai.com/api-keys))

### 1. Clone Repository

```bash
git clone https://github.com/sreejagatab/Medari.git
cd medari
```

### 2. Backend Setup

```bash
cd backend

# Create virtual environment
python -m venv venv

# Activate (Windows)
venv\Scripts\activate
# Activate (macOS/Linux)
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Configure environment
cp .env.example .env
# Edit .env and add your OPENAI_API_KEY

# Start server
python main.py
```

**Backend runs at:** http://localhost:8000

### 3. Mobile App Setup

```bash
cd app

# Install dependencies
npm install

# Install web dependencies (optional)
npx expo install react-native-web react-dom @expo/metro-runtime

# Start development server
npx expo start
```

**Development Options:**
- Press `w` - Open in web browser (http://localhost:8081)
- Press `i` - Open iOS simulator
- Press `a` - Open Android emulator
- Scan QR - Open on physical device with Expo Go

### 4. Connect Physical Device

Edit `app/src/api/client.ts`:

```typescript
const API_BASE_URL = __DEV__
  ? 'http://YOUR_LOCAL_IP:8000'  // e.g., 'http://192.168.1.100:8000'
  : 'https://api.medari.app';
```

---

## API Documentation

### Base URL

| Environment | URL |
|-------------|-----|
| Development | `http://localhost:8000` |
| Production | `https://api.medari.app` |

### Endpoints Overview

| Endpoint | Method | Description | Auth Required |
|----------|--------|-------------|---------------|
| `/health` | GET | Health check | No |
| `/api/chat` | POST | Process chat message | No |
| `/api/chat-with-research` | POST | Chat with research context | No |
| `/api/generate-notes` | POST | Generate Dr. Prep Notes | No |
| `/api/research` | POST | Fetch clinical research | No |

---

### GET /health

Health check endpoint.

**Response:**
```json
{
  "status": "healthy",
  "timestamp": "2025-12-23T09:00:00.000000"
}
```

---

### POST /api/chat

Process a chat message and return AI response with extracted symptom data.

**Request:**
```json
{
  "conversation_id": "uuid-string",
  "messages": [
    {"role": "user", "content": "I've been having headaches"},
    {"role": "assistant", "content": "I'm sorry to hear that..."},
    {"role": "user", "content": "They happen mostly in the morning"}
  ],
  "extracted_data": {
    "symptoms": [],
    "patterns": [],
    "triggers": [],
    "timeline": []
  }
}
```

**Response:**
```json
{
  "message": "Thank you for sharing that detail. Morning headaches can have various causes. On a scale of 1-10, how would you rate the pain intensity?",
  "extractedData": {
    "symptoms": [
      {
        "id": "sym_001",
        "name": "headache",
        "severity": 3,
        "frequency": "daily",
        "duration": "unknown",
        "bodyLocation": "head",
        "characteristics": ["morning onset"]
      }
    ],
    "patterns": [
      {
        "type": "time_of_day",
        "description": "Occurs primarily in the morning",
        "correlation": "strong"
      }
    ],
    "triggers": [],
    "timeline": []
  },
  "suggestedFollowUps": [],
  "readyForNotes": false,
  "research": null
}
```

---

### POST /api/chat-with-research

Same as `/api/chat` but includes clinical research context.

**Request:**
```json
{
  "conversation_id": "uuid-string",
  "messages": [...],
  "extracted_data": {...},
  "include_research": true
}
```

**Response:** Same as `/api/chat` plus `research` field:
```json
{
  "research": {
    "articles": [
      {
        "pmid": "41429157",
        "title": "Unraveling the relationship between white matter lesions in MRI and migraine",
        "authors": "Viuniski VS, Ruffini ML, Souza AM et al.",
        "journal": "Arquivos de neuro-psiquiatria",
        "year": "2025",
        "abstract": "White matter hyperintensities (WMH) are commonly detected...",
        "url": "https://pubmed.ncbi.nlm.nih.gov/41429157/",
        "citation": "Viuniski VS et al. Unraveling the relationship...",
        "inlineCitation": "[Viuniski et al., 2025]"
      }
    ],
    "trials": [
      {
        "nctId": "NCT06274255",
        "title": "Serum Magnesium Level and Pediatric Migraine",
        "status": "NOT_YET_RECRUITING",
        "phase": "Not specified",
        "conditions": ["Migraine", "Magnesium"],
        "interventions": [],
        "url": "https://clinicaltrials.gov/study/NCT06274255"
      }
    ],
    "summary": "Research context for headache: Found 3 relevant articles."
  }
}
```

---

### POST /api/generate-notes

Generate structured Dr. Prep Notes from conversation.

**Request:**
```json
{
  "conversation_id": "uuid-string",
  "messages": [...],
  "extracted_data": {...},
  "user_notes": "I also noticed the headaches are worse after screen time",
  "include_research": true
}
```

**Response:**
```json
{
  "id": "note_uuid",
  "createdAt": "2025-12-23T09:00:00.000000",
  "updatedAt": "2025-12-23T09:00:00.000000",
  "conversationId": "uuid-string",
  "summary": {
    "mainSymptoms": [
      {
        "what": "Recurring headaches",
        "when": "Morning, upon waking",
        "howOften": "Daily for past 2 weeks",
        "severity": "Moderate (5/10)",
        "duration": "1-2 hours"
      }
    ],
    "observedPatterns": [
      "Symptoms occur consistently in the morning",
      "May worsen with screen time"
    ],
    "possibleTriggers": [
      "Sleep position",
      "Dehydration overnight",
      "Screen exposure"
    ]
  },
  "questionsForDoctor": [
    "Could this be related to sleep apnea?",
    "Should I get a sleep study done?",
    "Are there lifestyle changes that might help?"
  ],
  "userNotes": "I also noticed the headaches are worse after screen time",
  "clinicalContext": [
    {
      "type": "research",
      "title": "Morning Headaches: A Systematic Review",
      "source": "Neurology Today",
      "year": "2024",
      "citation": "...",
      "url": "https://pubmed.ncbi.nlm.nih.gov/...",
      "summary": "Morning headaches affect approximately 1 in 13 people..."
    },
    {
      "type": "trial",
      "title": "Study of Sleep-Related Headache Treatment",
      "source": "ClinicalTrials.gov",
      "status": "RECRUITING",
      "phase": "Phase 3",
      "url": "https://clinicaltrials.gov/study/...",
      "summary": "Clinical trial studying headache, sleep disorders"
    }
  ]
}
```

---

### POST /api/research

Fetch clinical research for specific symptoms.

**Request:**
```json
{
  "symptoms": ["headache", "fatigue", "nausea"],
  "include_trials": true
}
```

**Response:**
```json
{
  "articles": [
    {
      "pmid": "12345678",
      "title": "...",
      "authors": "...",
      "journal": "...",
      "year": "2025",
      "abstract": "...",
      "url": "https://pubmed.ncbi.nlm.nih.gov/12345678/",
      "citation": "...",
      "inlineCitation": "[Author et al., 2025]"
    }
  ],
  "trials": [...],
  "summary": "Research context for headache, fatigue, nausea: Found 5 relevant articles and 3 active trials."
}
```

---

## Project Structure

```
medari/
|
+-- app/                              # React Native Mobile App
|   +-- app/                          # Screens (Expo Router)
|   |   +-- _layout.tsx               # Navigation configuration
|   |   +-- index.tsx                 # Main chat screen
|   |   +-- prep-notes.tsx            # Dr. Prep Notes viewer
|   |   +-- history.tsx               # Conversation history
|   |
|   +-- src/
|   |   +-- api/
|   |   |   +-- client.ts             # REST API client
|   |   |
|   |   +-- components/
|   |   |   +-- Button.tsx            # Reusable button (4 variants)
|   |   |   +-- ChatBubble.tsx        # Message display
|   |   |   +-- ChatInput.tsx         # Message input with send
|   |   |   +-- EmptyState.tsx        # Welcome screen
|   |   |   +-- index.ts              # Barrel exports
|   |   |
|   |   +-- hooks/
|   |   |   +-- useChat.ts            # Chat logic hook
|   |   |   +-- index.ts
|   |   |
|   |   +-- stores/
|   |   |   +-- medariStore.ts        # Zustand global state
|   |   |   +-- index.ts
|   |   |
|   |   +-- types/
|   |   |   +-- index.ts              # TypeScript interfaces
|   |   |
|   |   +-- utils/
|   |       +-- theme.ts              # Design system (colors, spacing)
|   |       +-- index.ts              # Utilities + PDF HTML generation
|   |
|   +-- assets/                       # App icons and splash
|   +-- app.json                      # Expo configuration
|   +-- package.json                  # Dependencies
|   +-- tsconfig.json                 # TypeScript config
|
+-- backend/                          # FastAPI Python Server
|   +-- main.py                       # All endpoints + OpenAI integration
|   +-- research_service.py           # PubMed + ClinicalTrials integration
|   +-- requirements.txt              # Python dependencies
|   +-- .env.example                  # Environment template
|   +-- .env                          # Environment variables (git-ignored)
|
+-- docs/                             # Additional documentation
|   +-- API.md                        # Detailed API docs
|   +-- ARCHITECTURE.md               # Architecture deep dive
|
+-- README.md                         # This file
+-- LICENSE                           # MIT License
```

---

## Testing

### Test Results Summary (2025-12-23)

| Category | Tests | Passed | Status |
|----------|-------|--------|--------|
| Backend API | 5 | 5 | ✅ ALL PASS |
| Frontend UI | 4 | 4 | ✅ ALL PASS |
| AI Integration | 4 | 4 | ✅ ALL PASS |
| Research Integration | 3 | 3 | ✅ ALL PASS |
| **Total** | **16** | **16** | **100%** |

### Detailed Test Results

#### Backend API Tests

| Test | Endpoint | Status | Response Time | Notes |
|------|----------|--------|---------------|-------|
| Health Check | `GET /health` | ✅ PASS | <100ms | Returns healthy status with timestamp |
| Chat API | `POST /api/chat` | ✅ PASS | ~2-3s | GPT-4o responds with follow-up questions |
| Chat with Research | `POST /api/chat-with-research` | ✅ PASS | ~4-5s | Includes research citations |
| Generate Notes | `POST /api/generate-notes` | ✅ PASS | ~5-6s | Full notes with clinical context |
| Research API | `POST /api/research` | ✅ PASS | ~2-3s | PubMed + ClinicalTrials.gov data |

#### Frontend UI Tests

| Test | Component | Status | Notes |
|------|-----------|--------|-------|
| Web Build | Expo Web | ✅ PASS | Bundles successfully |
| Chat Screen | index.tsx | ✅ PASS | Renders chat interface |
| Prep Notes | prep-notes.tsx | ✅ PASS | Displays structured notes |
| History | history.tsx | ✅ PASS | Shows conversation list |

#### AI Conversation Tests

| Test | Scenario | Status | Notes |
|------|----------|--------|-------|
| Severity Question | First response | ✅ PASS | AI asks "On a scale of 1-10..." |
| Duration Capture | Follow-up | ✅ PASS | Asks "How long does each episode last?" |
| Pattern Detection | Analysis | ✅ PASS | Identifies time-of-day patterns |
| Trigger Identification | Extraction | ✅ PASS | Captures activities, food, stress |

#### Research Integration Tests

| Test | Source | Status | Notes |
|------|--------|--------|-------|
| PubMed Fetch | E-utilities API | ✅ PASS | Returns systematic reviews, meta-analyses |
| ClinicalTrials Fetch | ClinicalTrials.gov | ✅ PASS | Returns active/recruiting trials |
| Citation Formatting | Backend | ✅ PASS | Proper [Author et al., Year] format |

---

### Live Test Conversation (Chronic Back Pain)

**Test Date:** 2025-12-23

#### Conversation Flow

```
User: "I have chronic back pain"

AI: "I'm sorry to hear you're dealing with that. Let's gather some details
     to understand it better. On a scale of 1 to 10, how would you rate
     your back pain?"

User: "10"

AI: "That sounds incredibly challenging. How long does each episode of
     back pain last, or is it a constant feeling?"

User: "morning, whole day, when i do anything, by night i am exhausted
       and tired and in full pain"

AI: "That sounds like it's really affecting your daily life. How often
     does this pain occur, and is it something you've been experiencing
     every day?"

User: "since 5 years and every day"

AI: "That sounds difficult to manage. Have you noticed anything that
     makes it better or worse, like certain activities or positions?"

User: "all activities so i stopped doing any of them and simply sitting
       and lying down in between 20 mins each"
```

#### Generated Dr. Prep Notes

| Field | Captured Value | Status |
|-------|----------------|--------|
| What | Chronic back pain | ✅ |
| When | Morning through night | ✅ |
| How Often | Every day | ✅ |
| Severity | Severe (10/10) | ✅ |
| Duration | Constant (all day) | ✅ |
| Patterns | Pain worsens by night | ✅ |
| Triggers | All activities | ✅ |

#### Clinical Research Retrieved

| Type | Title | Source | Year |
|------|-------|--------|------|
| 📄 Study | Engineered Extracellular Vesicles for Intervertebral Disc Regeneration | Cureus | 2025 |
| 📄 Study | Efficacy of Neuromodulation in Postoperative Low Back Pain | Journal of Clinical Neuroscience | 2025 |
| 📄 Study | Chiropractic Care and Spinal Manipulation: Evidence and Risks | JBJS Reviews | 2025 |
| 🧪 Trial | Novel Minimally Invasive Posterior Sacroiliac Fusion Device | ClinicalTrials.gov | COMPLETED |
| 🧪 Trial | Non-medicated Plaster in Chronic Lumbar Back Pain | ClinicalTrials.gov | COMPLETED |

#### Questions Generated for Doctor

1. What could be causing my chronic back pain that worsens throughout the day?
2. Are there specific tests you recommend to diagnose the cause of my back pain?
3. What are the treatment options available for managing severe back pain?
4. How can I modify my daily activities to reduce pain and improve my quality of life?
5. Are there any lifestyle changes or exercises that can help alleviate my back pain?

---

### Running Tests Manually

```bash
# 1. Health Check
curl http://localhost:8000/health
# Expected: {"status":"healthy","timestamp":"2025-12-23T..."}

# 2. Test Chat API
curl -X POST http://localhost:8000/api/chat \
  -H "Content-Type: application/json" \
  -d '{
    "conversation_id": "test123",
    "messages": [{"role": "user", "content": "I have headaches"}],
    "extracted_data": {"symptoms": [], "patterns": [], "triggers": [], "timeline": []}
  }'
# Expected: AI asks for severity (1-10 scale)

# 3. Test Research API
curl -X POST http://localhost:8000/api/research \
  -H "Content-Type: application/json" \
  -d '{"symptoms": ["back pain"], "include_trials": true}'
# Expected: Returns PubMed articles and clinical trials

# 4. Frontend Check
curl http://localhost:3000
# Expected: HTML response with React app
```

### Sample API Responses

#### Health Check Response
```json
{
  "status": "healthy",
  "timestamp": "2025-12-23T10:20:59.604883"
}
```

#### Chat API Response
```json
{
  "message": "I'm sorry to hear that you're experiencing headaches. On a scale of 1-10, how would you rate the severity of these headaches?",
  "extractedData": null,
  "suggestedFollowUps": [],
  "readyForNotes": false,
  "research": null
}
```

#### Research API Response
```json
{
  "articles": [
    {
      "pmid": "41429157",
      "title": "Unraveling the relationship between white matter lesions in MRI and migraine: a systematic review",
      "authors": "Viuniski VS, Ruffini ML, Souza AM et al.",
      "journal": "Arquivos de neuro-psiquiatria",
      "year": "2025",
      "url": "https://pubmed.ncbi.nlm.nih.gov/41429157/",
      "inlineCitation": "[Viuniski et al., 2025]"
    }
  ],
  "trials": [
    {
      "nctId": "NCT06274255",
      "title": "Serum Magnesium Level and Pediatric Migraine",
      "status": "NOT_YET_RECRUITING",
      "phase": "Not specified",
      "conditions": ["Migraine", "Magnesium", "Pediatric"],
      "url": "https://clinicaltrials.gov/study/NCT06274255"
    }
  ],
  "summary": "Research context for headache: Found 3 relevant research articles."
}
```

---

## Design System

### Colors

| Name | Hex | Usage |
|------|-----|-------|
| Primary | `#0F766E` | Buttons, headers, accents |
| Primary Light | `#14B8A6` | Hover states |
| Primary Dark | `#0D5D56` | Pressed states |
| Background | `#FFFFFF` | Main background |
| Surface | `#FAFAFA` | Cards, input backgrounds |
| Text Primary | `#18181B` | Main text |
| Text Secondary | `#52525B` | Secondary text |
| Text Tertiary | `#A1A1AA` | Placeholder text |
| Success | `#22C55E` | Success states |
| Warning | `#F59E0B` | Warning states |
| Error | `#EF4444` | Error states |
| Info | `#3B82F6` | Info, research cards |

### Typography

| Size | Value | Usage |
|------|-------|-------|
| xs | 12px | Badges, metadata |
| sm | 14px | Secondary text |
| base | 16px | Body text |
| lg | 18px | Section titles |
| xl | 20px | Card titles |
| 2xl | 24px | Page titles |
| 3xl | 30px | Hero text |

### Spacing

| Name | Value |
|------|-------|
| xs | 4px |
| sm | 8px |
| md | 16px |
| lg | 24px |
| xl | 32px |
| xxl | 48px |

---

## Roadmap

### v1.0.0 - Current Release
- [x] Core chat functionality
- [x] Symptom extraction
- [x] Dr. Prep Notes generation
- [x] PDF export
- [x] Clinical research integration (PubMed + ClinicalTrials.gov)

### v1.1.0 - Q1 2025
- [ ] User authentication (NextAuth/Clerk)
- [ ] Cloud sync (PostgreSQL)
- [ ] Secure doctor sharing links

### v1.2.0 - Q2 2025
- [ ] Pattern insights dashboard
- [ ] Symptom trend analysis
- [ ] Push notifications
- [ ] Appointment reminders

### v2.0.0 - Future
- [ ] EHR integration
- [ ] Multi-language support
- [ ] Wearable device integration
- [ ] Voice input support

---

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit changes (`git commit -m 'Add amazing feature'`)
4. Push to branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

### Development Guidelines

- Use TypeScript strict mode
- Follow existing code style
- Add tests for new features
- Update documentation

---

## License

MIT License - see [LICENSE](LICENSE) for details.

---

## Disclaimer

Medari is designed to help users organize and articulate their health concerns for discussion with healthcare providers. It is **NOT** a diagnostic tool and should **NOT** be used as a substitute for professional medical advice, diagnosis, or treatment. Always seek the advice of your physician or other qualified health provider with any questions you may have regarding a medical condition.

---

## Links & Resources

### Documentation
- [API Reference](docs/API.md) - Complete API documentation
- [Architecture](docs/ARCHITECTURE.md) - Technical architecture details
- [Setup Guide](docs/SETUP.md) - Installation and deployment
- [Enhancements](docs/ENHANCEMENTS.md) - Future improvements and roadmap

### APIs & Services
- [OpenAI API](https://platform.openai.com/docs)
- [PubMed E-utilities](https://www.ncbi.nlm.nih.gov/home/develop/api/)
- [ClinicalTrials.gov API](https://clinicaltrials.gov/data-api/api)

### Libraries
- [Expo Documentation](https://docs.expo.dev/)
- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [Zustand](https://github.com/pmndrs/zustand)
- [pymed](https://pypi.org/project/pymed/)
- [pytrials](https://pypi.org/project/pytrials/)

---

**Medari** - *Clarity before your doctor visit*

Built by [Jagatab.uk](https://jagatab.uk) with love.
