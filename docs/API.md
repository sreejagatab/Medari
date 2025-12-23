# Medari API Documentation

Complete API reference for the Medari backend service.

## Base Information

| Property | Value |
|----------|-------|
| Base URL (Dev) | `http://localhost:8000` |
| Base URL (Prod) | `https://api.medari.app` |
| Content-Type | `application/json` |
| Authentication | None required (v1.0) |

---

## Endpoints

### 1. Health Check

Check if the API server is running.

```
GET /health
```

#### Response

```json
{
  "status": "healthy",
  "timestamp": "2025-12-23T09:00:00.000000"
}
```

#### Status Codes

| Code | Description |
|------|-------------|
| 200 | Server is healthy |
| 500 | Server error |

---

### 2. Chat

Process a chat message and return AI response with extracted symptom data.

```
POST /api/chat
```

**AI Behavior:**
- First response always asks for severity (1-10 scale)
- Follows up with duration, frequency, timing, triggers
- Validates user concerns before asking questions
- Never provides diagnoses or medical advice

#### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| conversation_id | string | Yes | Unique conversation identifier |
| messages | array | Yes | Array of message objects |
| extracted_data | object | No | Previously extracted symptom data |

**Message Object:**

| Field | Type | Description |
|-------|------|-------------|
| role | string | "user" or "assistant" |
| content | string | Message text |

**Extracted Data Object:**

| Field | Type | Description |
|-------|------|-------------|
| symptoms | array | Array of Symptom objects |
| patterns | array | Array of Pattern objects |
| triggers | array | Array of trigger strings |
| timeline | array | Array of Timeline objects |

#### Example Request

```json
{
  "conversation_id": "conv_abc123",
  "messages": [
    {
      "role": "user",
      "content": "I've been having headaches for the past week"
    },
    {
      "role": "assistant",
      "content": "I'm sorry to hear that. Can you tell me more about these headaches?"
    },
    {
      "role": "user",
      "content": "They happen mostly in the morning when I wake up"
    }
  ],
  "extracted_data": {
    "symptoms": [],
    "patterns": [],
    "triggers": [],
    "timeline": []
  }
}
```

#### Response

```json
{
  "message": "Thank you for that detail. Morning headaches can have various causes. On a scale of 1-10, how would you rate the pain?",
  "extractedData": {
    "symptoms": [
      {
        "id": "sym_001",
        "name": "headache",
        "severity": 3,
        "frequency": "daily",
        "duration": "1 week",
        "bodyLocation": "head",
        "characteristics": ["morning onset"]
      }
    ],
    "patterns": [
      {
        "type": "time_of_day",
        "description": "Occurs upon waking in the morning",
        "correlation": "strong"
      }
    ],
    "triggers": [],
    "timeline": [
      {
        "date": "1 week ago",
        "event": "Headaches started",
        "symptomIds": ["sym_001"]
      }
    ]
  },
  "suggestedFollowUps": [],
  "readyForNotes": false,
  "research": null
}
```

#### Status Codes

| Code | Description |
|------|-------------|
| 200 | Success |
| 500 | Server error (check API key) |

---

### 3. Chat with Research

Same as Chat but includes clinical research context from PubMed and ClinicalTrials.gov.

```
POST /api/chat-with-research
```

#### Request Body

Same as `/api/chat` plus:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| include_research | boolean | No | Whether to fetch research (default: false) |

#### Example Request

```json
{
  "conversation_id": "conv_abc123",
  "messages": [...],
  "extracted_data": {...},
  "include_research": true
}
```

#### Response

Same as `/api/chat` plus research field:

```json
{
  "message": "...",
  "extractedData": {...},
  "readyForNotes": true,
  "research": {
    "articles": [
      {
        "pmid": "41429157",
        "title": "Unraveling the relationship between white matter lesions in MRI and migraine: a systematic review",
        "authors": "Viuniski VS, Ruffini ML, Souza AM et al.",
        "journal": "Arquivos de neuro-psiquiatria",
        "year": "2025",
        "abstract": "White matter hyperintensities (WMH) are commonly detected on brain magnetic resonance imaging...",
        "url": "https://pubmed.ncbi.nlm.nih.gov/41429157/",
        "citation": "Viuniski VS, Ruffini ML, Souza AM et al.. Unraveling the relationship between white matter lesions in MRI and migraine: a systematic review. Arquivos de neuro-psiquiatria. 2025. PMID: 41429157",
        "inlineCitation": "[Viuniski et al., 2025]"
      }
    ],
    "trials": [
      {
        "nctId": "NCT06274255",
        "title": "Serum Magnesium Level and Pediatric Migraine",
        "status": "NOT_YET_RECRUITING",
        "phase": "Not specified",
        "conditions": ["Migraine, Magnesium, Pediatric"],
        "interventions": [],
        "url": "https://clinicaltrials.gov/study/NCT06274255"
      }
    ],
    "summary": "Research context for headache: Found 3 relevant research articles."
  }
}
```

---

### 4. Generate Notes

Generate structured Dr. Prep Notes from a conversation.

```
POST /api/generate-notes
```

**Output Features:**
- Severity displayed as "Moderate (6/10)" or "Ask your doctor to assess"
- Duration handles constant pain: "Constant (all day)" vs episodic durations
- Generates 4-6 specific questions based on actual symptoms discussed
- Includes clinical research from PubMed and ClinicalTrials.gov

#### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| conversation_id | string | Yes | Conversation identifier |
| messages | array | Yes | Full conversation messages |
| extracted_data | object | Yes | Extracted symptom data |
| user_notes | string | No | Additional user notes |
| include_research | boolean | No | Include clinical research (default: true) |

#### Example Request

```json
{
  "conversation_id": "conv_abc123",
  "messages": [
    {"role": "user", "content": "I've been having headaches..."},
    {"role": "assistant", "content": "Can you tell me more..."},
    {"role": "user", "content": "They're worse in the morning..."},
    {"role": "assistant", "content": "How severe are they..."},
    {"role": "user", "content": "About 6 out of 10..."}
  ],
  "extracted_data": {
    "symptoms": [
      {
        "id": "sym_001",
        "name": "headache",
        "severity": 3,
        "frequency": "daily",
        "duration": "1 week",
        "bodyLocation": "head",
        "characteristics": ["morning onset"]
      }
    ],
    "patterns": [],
    "triggers": [],
    "timeline": []
  },
  "user_notes": "I think it might be related to my new pillow",
  "include_research": true
}
```

#### Response

```json
{
  "id": "note_xyz789",
  "createdAt": "2025-12-23T09:30:00.000000",
  "updatedAt": "2025-12-23T09:30:00.000000",
  "conversationId": "conv_abc123",
  "summary": {
    "mainSymptoms": [
      {
        "what": "Recurring headaches",
        "when": "Morning, upon waking",
        "howOften": "Daily for the past week",
        "severity": "Moderate (6/10)",
        "duration": "Variable"
      }
    ],
    "observedPatterns": [
      "Symptoms occur consistently upon waking",
      "May be related to sleep position or pillow"
    ],
    "possibleTriggers": [
      "Sleep position",
      "New pillow",
      "Possible neck strain"
    ]
  },
  "questionsForDoctor": [
    "Could my headaches be related to cervicogenic issues?",
    "Should I try a different pillow type?",
    "Would physical therapy help?",
    "Should we rule out sleep apnea?"
  ],
  "userNotes": "I think it might be related to my new pillow",
  "clinicalContext": [
    {
      "type": "research",
      "title": "Cervicogenic Headache: A Review",
      "source": "Pain Medicine Journal",
      "year": "2024",
      "citation": "...",
      "url": "https://pubmed.ncbi.nlm.nih.gov/...",
      "summary": "Cervicogenic headaches are secondary headaches caused by disorders of the cervical spine..."
    },
    {
      "type": "trial",
      "title": "Physical Therapy for Morning Headaches",
      "source": "ClinicalTrials.gov",
      "status": "RECRUITING",
      "phase": "Phase 2",
      "url": "https://clinicaltrials.gov/study/...",
      "summary": "Clinical trial studying headache, cervical spine"
    }
  ]
}
```

#### Status Codes

| Code | Description |
|------|-------------|
| 200 | Notes generated successfully |
| 500 | Failed to generate notes |

---

### 5. Research

Fetch clinical research for specific symptoms without chat context.

```
POST /api/research
```

#### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| symptoms | array | Yes | Array of symptom strings |
| include_trials | boolean | No | Include clinical trials (default: true) |

#### Example Request

```json
{
  "symptoms": ["migraine", "nausea", "light sensitivity"],
  "include_trials": true
}
```

#### Response

```json
{
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
    },
    {
      "pmid": "41423523",
      "title": "Occult spinal CSF leak: to patch or not to patch",
      "authors": "Wang J, Thome AP, Brook AL et al.",
      "journal": "Child's nervous system",
      "year": "2025",
      "abstract": "...",
      "url": "https://pubmed.ncbi.nlm.nih.gov/41423523/",
      "citation": "...",
      "inlineCitation": "[Wang et al., 2025]"
    }
  ],
  "trials": [
    {
      "nctId": "NCT06274255",
      "title": "Serum Magnesium Level and Pediatric Migraine",
      "status": "NOT_YET_RECRUITING",
      "phase": "Not specified",
      "conditions": ["Migraine", "Magnesium", "Pediatric"],
      "interventions": [],
      "url": "https://clinicaltrials.gov/study/NCT06274255"
    },
    {
      "nctId": "NCT03076515",
      "title": "Migraine Treatment With Nerivio Migra Neurostimulation Device",
      "status": "TERMINATED",
      "phase": "NA",
      "conditions": ["Acute Migraine"],
      "interventions": ["Nerivio Migra neurostimulation", "Sham Nerivio Migra"],
      "url": "https://clinicaltrials.gov/study/NCT03076515"
    }
  ],
  "summary": "Research context for migraine, nausea, light sensitivity: Found 5 relevant research articles and 3 clinical trials."
}
```

---

## Data Models

### Symptom

```typescript
interface Symptom {
  id: string;                    // Unique identifier
  name: string;                  // Symptom name
  severity: 1 | 2 | 3 | 4 | 5;   // Severity scale
  frequency: string;             // How often it occurs
  duration: string;              // How long it lasts
  bodyLocation?: string;         // Where on body
  characteristics: string[];     // Descriptive traits
}
```

### Pattern

```typescript
interface Pattern {
  type: 'time_of_day' | 'activity' | 'food' | 'stress' | 'weather' | 'other';
  description: string;           // Pattern description
  correlation: 'strong' | 'moderate' | 'weak';
}
```

### TimelineEntry

```typescript
interface TimelineEntry {
  date: string;                  // When it occurred
  event: string;                 // What happened
  symptomIds: string[];          // Related symptoms
}
```

### ResearchArticle

```typescript
interface ResearchArticle {
  pmid: string;                  // PubMed ID
  title: string;                 // Article title
  authors: string;               // Author list
  journal: string;               // Journal name
  year: string;                  // Publication year
  abstract: string;              // Article abstract
  url: string;                   // PubMed URL
  citation: string;              // Full citation
  inlineCitation: string;        // Short citation [Author et al., Year]
}
```

### ClinicalTrial

```typescript
interface ClinicalTrial {
  nctId: string;                 // ClinicalTrials.gov ID
  title: string;                 // Trial title
  status: string;                // Recruitment status
  phase: string;                 // Trial phase
  conditions: string[];          // Conditions studied
  interventions: string[];       // Interventions used
  url: string;                   // ClinicalTrials.gov URL
}
```

### DrPrepNotes

```typescript
interface DrPrepNotes {
  id: string;                    // Note ID
  createdAt: string;             // ISO timestamp
  updatedAt: string;             // ISO timestamp
  conversationId: string;        // Related conversation
  summary: {
    mainSymptoms: SymptomSummary[];
    observedPatterns: string[];
    possibleTriggers: string[];
  };
  questionsForDoctor: string[];  // Suggested questions
  userNotes: string;             // User's additional notes
  clinicalContext: ClinicalContext[];
}
```

---

## Error Handling

### Error Response Format

```json
{
  "detail": "Error message description"
}
```

### Common Errors

| Status | Error | Cause |
|--------|-------|-------|
| 400 | Invalid request body | Malformed JSON or missing fields |
| 500 | API key error | Missing or invalid OPENAI_API_KEY |
| 500 | JSON parse error | AI returned invalid JSON |
| 500 | Network error | External API unreachable |

---

## Rate Limits

### OpenAI API
- Depends on your API plan
- Medari uses gpt-4o

### PubMed E-utilities
- 3 requests/second without API key
- 10 requests/second with NCBI API key

### ClinicalTrials.gov
- No rate limits
- No authentication required

---

## Testing with cURL

### Health Check
```bash
curl http://localhost:8000/health
```

### Research Query
```bash
curl -X POST http://localhost:8000/api/research \
  -H "Content-Type: application/json" \
  -d '{"symptoms": ["headache"], "include_trials": true}'
```

### Chat Message
```bash
curl -X POST http://localhost:8000/api/chat \
  -H "Content-Type: application/json" \
  -d '{
    "conversation_id": "test123",
    "messages": [
      {"role": "user", "content": "I have been having headaches"}
    ],
    "extracted_data": {"symptoms": [], "patterns": [], "triggers": [], "timeline": []}
  }'
```

### Generate Notes
```bash
curl -X POST http://localhost:8000/api/generate-notes \
  -H "Content-Type: application/json" \
  -d '{
    "conversation_id": "test123",
    "messages": [
      {"role": "user", "content": "I have headaches"},
      {"role": "assistant", "content": "Tell me more"},
      {"role": "user", "content": "They happen in the morning"},
      {"role": "assistant", "content": "How severe?"},
      {"role": "user", "content": "About 6/10"}
    ],
    "extracted_data": {
      "symptoms": [{"id": "1", "name": "headache", "severity": 3, "frequency": "daily", "duration": "1 week", "characteristics": []}],
      "patterns": [],
      "triggers": [],
      "timeline": []
    }
  }'
```

---

Built by [Jagatab.uk](https://jagatab.uk) with love.
