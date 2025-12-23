# Medari Setup Guide

Complete setup instructions for development and deployment.

---

## Prerequisites

### Required Software

| Software | Version | Purpose |
|----------|---------|---------|
| Node.js | 18+ | Frontend runtime |
| npm | 9+ | Package manager |
| Python | 3.11+ | Backend runtime |
| Git | 2.0+ | Version control |

### Optional Software

| Software | Purpose |
|----------|---------|
| VS Code | Recommended IDE |
| Expo Go | Mobile testing app |
| Android Studio | Android emulator |
| Xcode | iOS simulator (macOS only) |

### API Keys Required

| Service | Required | Get It |
|---------|----------|--------|
| OpenAI API | Yes | [platform.openai.com](https://platform.openai.com/api-keys) |
| NCBI API | No | [ncbi.nlm.nih.gov/account](https://www.ncbi.nlm.nih.gov/account/) |

---

## Quick Start

### 1. Clone Repository

```bash
git clone https://github.com/sreejagatab/Medari.git
cd medari
```

### 2. Backend Setup

```bash
# Navigate to backend
cd backend

# Create virtual environment
python -m venv venv

# Activate virtual environment
# Windows:
venv\Scripts\activate
# macOS/Linux:
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Create environment file
cp .env.example .env

# Edit .env and add your API key
# OPENAI_API_KEY=sk-proj-your-key-here

# Start server
python main.py
```

**Backend runs at:** http://localhost:8000

### 3. Frontend Setup

```bash
# Navigate to app (from project root)
cd app

# Install dependencies
npm install

# Start development server
npx expo start
```

**Options:**
- Press `w` - Open in web browser
- Press `i` - Open iOS simulator
- Press `a` - Open Android emulator
- Scan QR - Open on physical device with Expo Go

---

## Detailed Setup

### Backend Configuration

#### Environment Variables

Create `backend/.env`:

```env
# Required - OpenAI API Key
OPENAI_API_KEY=sk-proj-xxxxxxxxxxxxx

# Optional - Server configuration
HOST=0.0.0.0
PORT=8000

# Optional - NCBI API key for higher rate limits
NCBI_API_KEY=your_ncbi_key_here
```

#### Dependencies

The `requirements.txt` includes:

```
fastapi>=0.109.0
uvicorn[standard]>=0.27.0
openai>=1.0.0
pydantic>=2.5.3
python-dotenv>=1.0.0
pymed>=0.8.9
pytrials>=0.3.0
httpx>=0.25.0
```

#### Verify Backend

```bash
# Health check
curl http://localhost:8000/health

# Expected response:
# {"status":"healthy","timestamp":"2025-12-23T09:00:00.000000"}

# Test chat API (requires OpenAI key)
curl -X POST http://localhost:8000/api/chat \
  -H "Content-Type: application/json" \
  -d '{"conversation_id":"test","messages":[{"role":"user","content":"I have headaches"}],"extracted_data":{"symptoms":[],"patterns":[],"triggers":[],"timeline":[]}}'

# Expected: AI asks for severity (1-10 scale)

# Test research API (no key required)
curl -X POST http://localhost:8000/api/research \
  -H "Content-Type: application/json" \
  -d '{"symptoms":["headache"],"include_trials":true}'

# Expected: Returns PubMed articles and clinical trials
```

### Frontend Configuration

#### Web Support (Optional)

To run on web browser:

```bash
cd app
npx expo install react-native-web react-dom @expo/metro-runtime
npx expo start --web
```

#### Physical Device Testing

1. Install Expo Go on your device:
   - [iOS App Store](https://apps.apple.com/app/expo-go/id982107779)
   - [Google Play Store](https://play.google.com/store/apps/details?id=host.exp.exponent)

2. Find your computer's local IP:
   ```bash
   # Windows
   ipconfig

   # macOS/Linux
   ifconfig
   ```

3. Update API client (`app/src/api/client.ts`):
   ```typescript
   const API_BASE_URL = __DEV__
     ? 'http://192.168.1.100:8000'  // Your local IP
     : 'https://api.medari.app';
   ```

4. Start Expo and scan QR code with Expo Go

#### iOS Simulator (macOS only)

1. Install Xcode from App Store
2. Open Xcode > Preferences > Locations > Command Line Tools
3. Run `npx expo start` and press `i`

#### Android Emulator

1. Install Android Studio
2. Create AVD (Android Virtual Device)
3. Run `npx expo start` and press `a`

---

## Development Workflow

### Start Development Servers

**Terminal 1 - Backend:**
```bash
cd backend
venv\Scripts\activate  # or source venv/bin/activate
python main.py
```

**Terminal 2 - Frontend:**
```bash
cd app
npx expo start
```

### Hot Reload

- **Frontend**: Auto-reloads on file save
- **Backend**: Restart server after changes (or use `--reload` flag)

```bash
# Backend with auto-reload
uvicorn main:app --reload --host 0.0.0.0 --port 8000
```

### Running Tests

```bash
# Backend tests (when added)
cd backend
pytest

# Frontend tests (when added)
cd app
npm test
```

---

## Troubleshooting

### Common Issues

#### Backend: "OPENAI_API_KEY not found"

```
Solution: Ensure .env file exists with valid API key
1. cp .env.example .env
2. Edit .env and add your OpenAI key
3. Restart server
```

#### Backend: Port 8000 already in use

```bash
# Find process using port
# Windows:
netstat -ano | findstr :8000
taskkill /PID <PID> /F

# macOS/Linux:
lsof -i :8000
kill -9 <PID>
```

#### Frontend: Metro bundler crash

```bash
# Clear cache and restart
cd app
npx expo start --clear
```

#### Frontend: Module not found

```bash
# Reinstall dependencies
cd app
rm -rf node_modules
npm install
```

#### Web: "shadow*" style props deprecated

This is a warning, not an error. The app still works. Future update will fix.

#### Physical device: Can't connect to backend

1. Ensure device is on same WiFi network
2. Check computer's firewall allows port 8000
3. Verify IP address in `client.ts` is correct
4. Try disabling firewall temporarily for testing

### Log Files

- **Backend**: Console output
- **Frontend**: Expo terminal + browser console (web) or device logs

---

## Production Deployment

### Backend Deployment (Example: Railway/Render)

1. Push code to GitHub
2. Connect to Railway/Render
3. Set environment variables:
   - `OPENAI_API_KEY`
   - `PORT` (usually auto-set)
4. Deploy

### Frontend Deployment

#### Web (Vercel/Netlify)

```bash
cd app
npx expo export --platform web
# Deploy dist/ folder
```

#### Mobile (EAS Build)

```bash
# Install EAS CLI
npm install -g eas-cli

# Login to Expo
eas login

# Configure build
eas build:configure

# Build for iOS
eas build --platform ios

# Build for Android
eas build --platform android
```

---

## Project Structure After Setup

```
medari/
+-- app/                    # React Native app
|   +-- node_modules/       # (generated)
|   +-- .expo/              # (generated)
|   +-- app/                # Screens
|   +-- src/                # Source code
|   +-- assets/             # Icons and splash
|   +-- package.json
|   +-- tsconfig.json
|   +-- app.json
|
+-- backend/                # FastAPI server
|   +-- venv/               # (generated)
|   +-- main.py
|   +-- research_service.py
|   +-- requirements.txt
|   +-- .env                # (create from .env.example)
|   +-- .env.example
|
+-- docs/                   # Documentation
|   +-- API.md
|   +-- ARCHITECTURE.md
|   +-- SETUP.md
|
+-- README.md
+-- LICENSE
```

---

## IDE Setup (VS Code Recommended)

### Extensions

- Python (Microsoft)
- Pylance
- ESLint
- Prettier
- TypeScript Vue Plugin (Volar)
- React Native Tools
- Expo Tools

### Settings (`.vscode/settings.json`)

```json
{
  "python.defaultInterpreterPath": "./backend/venv/Scripts/python",
  "python.formatting.provider": "black",
  "editor.formatOnSave": true,
  "typescript.preferences.importModuleSpecifier": "relative",
  "[typescript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[typescriptreact]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  }
}
```

---

## Getting Help

- **Issues**: [GitHub Issues](https://github.com/sreejagatab/Medari/issues)
- **Documentation**: See `docs/` folder
- **API Reference**: See `docs/API.md`

---

*Last Updated: 2025-12-23*

Built by [Jagatab.uk](https://jagatab.uk) with love.
