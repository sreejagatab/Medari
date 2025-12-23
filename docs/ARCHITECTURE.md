# Medari Architecture Documentation

Detailed technical architecture of the Medari health companion application.

---

## Table of Contents

1. [System Overview](#system-overview)
2. [Frontend Architecture](#frontend-architecture)
3. [Backend Architecture](#backend-architecture)
4. [Data Flow](#data-flow)
5. [AI Integration](#ai-integration)
6. [Research Integration](#research-integration)
7. [State Management](#state-management)
8. [Security Considerations](#security-considerations)

---

## System Overview

Medari is a full-stack health companion application built with:

- **Frontend**: React Native + Expo (cross-platform mobile + web)
- **Backend**: FastAPI (Python async web framework)
- **AI**: OpenAI GPT-4o API (conversation + extraction + generation)
- **Research**: PubMed E-utilities + ClinicalTrials.gov API

### High-Level Architecture

```
+------------------+     +------------------+     +------------------+
|                  |     |                  |     |                  |
|   Mobile App     |<--->|   FastAPI        |<--->|   OpenAI API     |
|  (React Native)  |     |   Backend        |     |   (GPT-4o)       |
|                  |     |                  |     |                  |
+------------------+     +--------+---------+     +------------------+
                                 |
                    +------------+------------+
                    |                         |
           +--------v--------+       +--------v--------+
           |                 |       |                 |
           |  PubMed API     |       | ClinicalTrials  |
           |  (NCBI)         |       |    .gov API     |
           |                 |       |                 |
           +-----------------+       +-----------------+
```

---

## Frontend Architecture

### Technology Stack

| Layer | Technology | Purpose |
|-------|------------|---------|
| Framework | React Native 0.73 | Cross-platform UI |
| Platform | Expo 50 | Build tools & native APIs |
| Navigation | Expo Router 3.4 | File-based routing |
| State | Zustand 4.5 | Global state management |
| Storage | AsyncStorage | Local persistence |
| Language | TypeScript 5.3 | Type safety |

### Directory Structure

```
app/
+-- app/                    # Expo Router screens
|   +-- _layout.tsx         # Root navigation layout
|   +-- index.tsx           # Chat screen (home)
|   +-- prep-notes.tsx      # Dr. Prep Notes viewer
|   +-- history.tsx         # Conversation history
|
+-- src/
    +-- api/
    |   +-- client.ts       # REST API client singleton
    |
    +-- components/
    |   +-- Button.tsx      # Reusable button (4 variants, 3 sizes)
    |   +-- ChatBubble.tsx  # Message display component
    |   +-- ChatInput.tsx   # Text input with send button
    |   +-- EmptyState.tsx  # Welcome screen with suggestions
    |   +-- index.ts        # Barrel exports
    |
    +-- hooks/
    |   +-- useChat.ts      # Chat logic hook
    |
    +-- stores/
    |   +-- medariStore.ts  # Zustand store definition
    |
    +-- types/
    |   +-- index.ts        # TypeScript interfaces
    |
    +-- utils/
        +-- theme.ts        # Design system constants
        +-- index.ts        # Utility functions + PDF HTML
```

### Screen Flow

```
+-------------+     +---------------+     +-------------+
|             |     |               |     |             |
|   Chat      |---->|  Prep Notes   |     |   History   |
|  (index)    |     |   (modal)     |     |   (stack)   |
|             |     |               |     |             |
+------+------+     +---------------+     +------+------+
       |                                         |
       +-----------------------------------------+
                     |
              Load Conversation
```

### Component Hierarchy

```
_layout (Stack Navigator)
|
+-- index (Chat Screen)
|   +-- Header (History + New Chat buttons)
|   +-- FlatList
|   |   +-- EmptyState (when no messages)
|   |   +-- ChatBubble (for each message)
|   +-- Button (Create Dr. Prep Notes)
|   +-- ChatInput
|
+-- prep-notes (Modal Screen)
|   +-- ScrollView
|   |   +-- Header (title + date)
|   |   +-- Symptom Summary Section
|   |   +-- Patterns Section
|   |   +-- Triggers Section
|   |   +-- Questions Section
|   |   +-- Clinical Research Section
|   |   +-- User Notes Input
|   +-- Action Buttons (PDF + Share)
|
+-- history (Stack Screen)
    +-- FlatList
        +-- ConversationCard (for each conversation)
```

---

## Backend Architecture

### Technology Stack

| Layer | Technology | Purpose |
|-------|------------|---------|
| Framework | FastAPI 0.127 | Async web framework |
| Server | Uvicorn 0.40 | ASGI server |
| Validation | Pydantic 2.12 | Data models & validation |
| AI | OpenAI SDK 2.14 | GPT-4o API client |
| Research | pymed 0.8.9 | PubMed client |
| Research | pytrials 1.0.0 | ClinicalTrials.gov client |

### File Structure

```
backend/
+-- main.py               # FastAPI app + all endpoints
+-- research_service.py   # PubMed + ClinicalTrials integration
+-- requirements.txt      # Python dependencies
+-- .env                  # Environment variables (git-ignored)
+-- .env.example          # Environment template
```

### API Endpoint Architecture

```
FastAPI App
|
+-- Middleware
|   +-- CORS (allow all origins for dev)
|
+-- Endpoints
|   +-- GET  /health
|   +-- POST /api/chat
|   +-- POST /api/chat-with-research
|   +-- POST /api/generate-notes
|   +-- POST /api/research
|
+-- Services
    +-- OpenAI Client (GPT-4o API)
    +-- Research Service (PubMed + ClinicalTrials)
```

### Request Processing Flow

```
Request --> Pydantic Validation --> Handler --> Response
                |                      |
                v                      v
           Validate JSON         Process Logic
           Check Types           Call External APIs
           Set Defaults          Format Response
```

---

## Data Flow

### Chat Message Flow

```
1. User types message in ChatInput
              |
              v
2. Message added to Zustand store
              |
              v
3. API client sends POST /api/chat
              |
              v
4. Backend receives ChatRequest
              |
              +----------------+
              |                |
              v                v
5. OpenAI: Generate     6. OpenAI: Extract
   Response                 Symptoms
              |                |
              +----------------+
              |
              v
7. Return ChatResponse
              |
              v
8. Store updates with:
   - Assistant message
   - Extracted data
              |
              v
9. UI re-renders with new message
```

### Notes Generation Flow

```
1. User taps "Create Dr. Prep Notes"
              |
              v
2. Navigate to prep-notes screen
              |
              v
3. API client sends POST /api/generate-notes
              |
              v
4. Backend processes request
              |
              +------------------+------------------+
              |                  |                  |
              v                  v                  v
5. OpenAI: Generate    6. PubMed: Fetch    7. ClinicalTrials:
   Structured Notes        Articles            Fetch Trials
              |                  |                  |
              +------------------+------------------+
              |
              v
8. Combine into DrPrepNotes response
              |
              v
9. Store saves prepNotes to conversation
              |
              v
10. UI renders notes with clinical context
```

---

## AI Integration

### OpenAI Model Usage

| Use Case | Model | Max Tokens | System Prompt |
|----------|-------|------------|---------------|
| Chat Response | gpt-4o | 1024 | CHAT_SYSTEM_PROMPT |
| Symptom Extraction | gpt-4o | 2048 | EXTRACTION_PROMPT |
| Notes Generation | gpt-4o | 2048 | NOTES_SYSTEM_PROMPT |
| Chat with Research | gpt-4o | 1024 | CHAT_WITH_RESEARCH_PROMPT |

### System Prompts

#### Chat System Prompt
- Defines Medari as a compassionate health companion
- **Prioritizes critical questions in order:**
  1. Severity (1-10 scale) - ALWAYS asked first
  2. Duration of each episode
  3. Frequency of occurrence
  4. Timing and patterns
  5. Triggers and relieving factors
- Never diagnose or prescribe
- Warm, supportive tone
- Always includes a follow-up question

#### Extraction Prompt
- Returns structured JSON only
- Extracts: symptoms, patterns, triggers, timeline
- Only includes explicitly mentioned information
- Handles severity on 1-5 scale internally

#### Notes System Prompt
- Creates structured Dr. Prep Notes
- **Smart handling of missing data:**
  - Severity: Shows "Moderate (6/10)" or "Ask your doctor to assess"
  - Duration: Shows "Constant (all day)" for chronic pain or "Discuss with your doctor"
- Generates 4-6 **specific** questions based on actual symptoms
- Includes symptom summaries, patterns, triggers
- Plain language, concise format
- Returns JSON only

### Response Processing

```python
# Clean markdown code blocks from OpenAI response
response_text = response.choices[0].message.content.strip()
if response_text.startswith("```"):
    response_text = response_text.split("\n", 1)[1]
if response_text.endswith("```"):
    response_text = response_text.rsplit("```", 1)[0]
response_text = response_text.strip()

# Parse JSON
data = json.loads(response_text)
```

---

## Research Integration

### PubMed Integration

**Library**: pymed 0.8.9

**Search Strategy**:
```python
# Enhanced query for high-quality articles
enhanced_query = f"({query}) AND (systematic review[pt] OR meta-analysis[pt] OR clinical trial[pt] OR review[pt])"
```

**Extracted Fields**:
- PMID
- Title
- Authors (first 3 + et al.)
- Journal
- Year
- Abstract
- URL

### ClinicalTrials.gov Integration

**Library**: pytrials 1.0.0

**API Version**: v2

**Extracted Fields**:
- NCT ID
- Brief Title
- Overall Status
- Phase
- Conditions
- Interventions

### Research Service Architecture

```python
class ClinicalResearchService:
    def __init__(self):
        self.pubmed = PubMed(tool="Medari", email="...")
        self.clinical_trials = ClinicalTrials()

    async def search_pubmed(query, max_results) -> list[ResearchArticle]
    async def search_clinical_trials(condition, max_results) -> list[ClinicalTrial]
    async def get_research_for_symptoms(symptoms) -> ResearchContext

    def format_citation(article) -> str
    def format_citation_inline(article) -> str  # [Author et al., Year]
```

---

## State Management

### Zustand Store Structure

```typescript
interface MedariState {
  // State
  currentConversation: Conversation | null;
  conversations: Conversation[];
  isLoading: boolean;
  error: string | null;

  // Actions
  startNewConversation: () => void;
  addMessage: (message: Message) => void;
  updateExtractedData: (data: ExtractedSymptomData) => void;
  setPrepNotes: (notes: DrPrepNotes) => void;
  setLoading: (loading: boolean) => void;
  setError: (error: string | null) => void;
  loadConversation: (id: string) => void;
  deleteConversation: (id: string) => void;
  clearCurrentConversation: () => void;
}
```

### Persistence Strategy

```typescript
// Using zustand persist middleware with AsyncStorage
persist(
  (set, get) => ({
    // ... state and actions
  }),
  {
    name: 'medari-storage',
    storage: createJSONStorage(() => AsyncStorage),
    partialize: (state) => ({
      conversations: state.conversations,
      // Don't persist currentConversation, isLoading, error
    }),
  }
)
```

### State Flow

```
User Action
     |
     v
Zustand Action Called
     |
     v
State Updated (immutable)
     |
     v
React Components Re-render
     |
     v
AsyncStorage Persisted (debounced)
```

---

## Security Considerations

### Current Implementation (v1.0)

| Aspect | Status | Notes |
|--------|--------|-------|
| Authentication | None | Planned for v1.1 |
| Data Storage | Local only | AsyncStorage on device |
| API Security | CORS enabled | Open for development |
| Sensitive Data | On device only | Not sent to external servers |
| API Keys | Server-side only | OpenAI key in backend .env |

### Data Privacy

1. **Conversation Data**: Stored locally on user's device
2. **Health Information**: Never stored on backend
3. **Research Queries**: Symptoms sent to PubMed/ClinicalTrials (anonymized)
4. **AI Processing**: Conversation sent to OpenAI API (OpenAI's privacy policy applies)

### Planned Security (v1.1+)

- [ ] User authentication (OAuth/JWT)
- [ ] End-to-end encryption for data sync
- [ ] Secure doctor sharing links with expiration
- [ ] Audit logging
- [ ] HIPAA compliance review

---

## Performance Considerations

### Frontend Optimizations

- **FlatList**: Virtualized list for messages
- **Memoization**: React.memo for chat bubbles
- **Lazy Loading**: Screens loaded on demand
- **Image Optimization**: Placeholder icons (SVG-like approach)

### UI/UX Enhancements

- **Missing Data Indicators**: Visual warnings (⚠️) for undiscussed fields
- **Semantic Color System**: Extended color palette for success, warning, error, info states
- **Smart Field Styling**: Orange/italic text for fields requiring doctor discussion
- **Research Cards**: Blue-themed cards with study/trial badges

### Backend Optimizations

- **Async/Await**: Non-blocking I/O
- **Parallel Requests**: Research fetched concurrently
- **Response Streaming**: Potential for future chat streaming
- **Caching**: Research results could be cached (not implemented)

### API Rate Limits

| Service | Limit | Strategy |
|---------|-------|----------|
| OpenAI API | Plan-dependent | Batch extraction with conversation |
| PubMed | 3-10 req/sec | Limit to 3 symptoms, 3 articles each |
| ClinicalTrials | Unlimited | Limit to 2 trials per symptom |

---

## Deployment Architecture (Production)

### Recommended Setup

```
                     +------------------+
                     |                  |
                     |   Load Balancer  |
                     |                  |
                     +--------+---------+
                              |
              +---------------+---------------+
              |                               |
     +--------v--------+             +--------v--------+
     |                 |             |                 |
     |  API Server 1   |             |  API Server 2   |
     |   (FastAPI)     |             |   (FastAPI)     |
     |                 |             |                 |
     +--------+--------+             +--------+--------+
              |                               |
              +---------------+---------------+
                              |
                     +--------v--------+
                     |                 |
                     |   PostgreSQL    |  (Future: user data)
                     |                 |
                     +-----------------+
```

### Environment Variables

```bash
# Required
OPENAI_API_KEY=sk-proj-...

# Optional
HOST=0.0.0.0
PORT=8000
NCBI_API_KEY=...  # For higher PubMed rate limits

# Future
DATABASE_URL=postgresql://...
JWT_SECRET=...
```

---

## Future Architecture Considerations

### v1.1 - Authentication & Cloud Sync

```
+-- Auth Service (NextAuth/Clerk)
|
+-- User Database (PostgreSQL)
|
+-- Conversation Sync API
```

### v1.2 - Analytics & Insights

```
+-- Analytics Service
|   +-- Symptom Trends
|   +-- Pattern Detection
|   +-- Correlation Analysis
|
+-- Time-Series Database
```

### v2.0 - Healthcare Integration

```
+-- FHIR Gateway
|
+-- EHR Connectors
|   +-- Epic
|   +-- Cerner
|   +-- etc.
|
+-- HL7 Message Processing
```

---

## Development Guidelines

### Adding New Features

1. **Frontend**: Create component in `src/components/`, add to store if needed
2. **Backend**: Add endpoint in `main.py`, create service if complex
3. **Types**: Update `src/types/index.ts` and Pydantic models
4. **Tests**: Add tests for new functionality

### Code Style

- **TypeScript**: Strict mode, explicit types
- **Python**: Type hints, async/await
- **Components**: Functional with hooks
- **State**: Immutable updates

### Git Workflow

```bash
main (production)
  |
  +-- develop (integration)
        |
        +-- feature/xxx (feature branches)
        +-- fix/xxx (bug fixes)
```

---

*Last Updated: 2025-12-23*

Built by [Jagatab.uk](https://jagatab.uk) with love.
