# 🎓 University AI Chatbot — Optimized & Production-Ready

> **Flutter (Web/Mobile) ↔ Django REST API ↔ RAG Engine ↔ Ollama (LLaMA3)**  
> 100% local, no API costs, runs on your own machine.

---

## 🐛 All Bugs Fixed

| # | File | Bug | Fix |
|---|------|-----|-----|
| 1 | `api_service.dart` | URL was `10.0.2.2:8000` (Android emulator) — broke Flutter Web | Auto-detects platform via `kIsWeb`; correct URL per platform |
| 2 | `rag_engine.py` | All LangChain v0.2.x imports were broken | Updated all imports: `langchain_ollama`, `langchain_core`, `langchain_text_splitters` |
| 3 | `rag_engine.py` | `langchain_community.llms.Ollama` is deprecated | Replaced with `langchain_ollama.OllamaLLM` |
| 4 | `rag_engine.py` | No timeout → silent hang on cold-start | Added `timeout=180` to `OllamaLLM` |
| 5 | `rag_engine.py` | Used deprecated `RetrievalQA.from_chain_type` | Replaced with LCEL chain (modern LangChain) |
| 6 | `settings.py` | `ADMIN_API_KEY` missing → `AttributeError` in views | Added `ADMIN_API_KEY` to settings |
| 7 | `settings.py` | CORS blocked Flutter Web requests | `CORS_ALLOW_ALL_ORIGINS=True` in DEBUG |
| 8 | `settings.py` | `APPEND_SLASH=True` caused POST 301 redirect → body lost | Set `APPEND_SLASH=False` |
| 9 | `models.py` | `client_ip` field missing (views tried to save it) | Added `client_ip` field + migration |
| 10 | `requirements.txt` | Missing `langchain-ollama`, `langchain-core`, `langchain-text-splitters` | All packages and versions pinned correctly |
| 11 | `urls.py` | Root path `/` returned 404 | Added proper API info handler |

---

## ⚡ Optimizations Added

| # | File | Optimization | Benefit |
|---|------|-------------|---------|
| O1 | `rag_engine.py` | **MMR retrieval** (Maximal Marginal Relevance) | Diverse, non-redundant chunks → better answers |
| O2 | `rag_engine.py` | Q&A-tuned chunk splitter (split at `\n\nQ:` first) | Keeps Q&A pairs together → better retrieval |
| O3 | `rag_engine.py` | Source deduplication before response | Cleaner metadata, smaller response |
| O4 | `rag_engine.py` | Query preprocessing (unicode normalise, strip punct) | Better matching for Urdu/Roman Urdu queries |
| O5 | `rag_engine.py` | Thread-safe rebuild (holds lock during index wipe) | No race conditions on concurrent requests |
| O6 | `rag_engine.py` | Context length guard (8000 char limit) | Prevents LLM context window overflow |
| O7 | `settings.py` | `CHUNK_SIZE=600/OVERLAP=80` (was 400/50) | Better fits full Q&A answers in one chunk |
| O8 | `views.py` | `sources_count` in chat response | Flutter UI can show how many sources used |
| O9 | `api_service.dart` | Singleton HTTP client reuse | Avoids port exhaustion on rapid requests |
| O10 | `api_service.dart` | Auto-retry on timeout (once, after 5s) | Handles the "model just finished loading" case |

---

## 📁 Project Structure

```
university_chatbot_optimized/
├── backend/                           # Django REST API
│   ├── chatbot/
│   │   ├── migrations/
│   │   │   └── 0001_initial.py        # DB schema (includes client_ip field)
│   │   ├── rag_engine.py              # ⭐ Core RAG (optimized MMR + LCEL)
│   │   ├── views.py                   # API endpoints
│   │   ├── models.py                  # ChatLog model
│   │   ├── serializers.py             # Input validation
│   │   ├── urls.py                    # Route definitions
│   │   ├── utils.py                   # Custom exception handler
│   │   ├── admin.py                   # Django Admin setup
│   │   └── apps.py                    # App config (loads RAG on startup)
│   ├── data/
│   │   └── university_data.json       # ⭐ Knowledge base (edit this)
│   ├── university_chatbot/
│   │   ├── settings.py                # ⭐ All configuration (tuned)
│   │   ├── urls.py                    # Root URL routing
│   │   └── wsgi.py
│   ├── vector_store/                  # FAISS index (auto-created, do not edit)
│   ├── logs/                          # Server logs (auto-created)
│   ├── build_index.py                 # ⭐ Run this ONCE before first start
│   ├── gunicorn.conf.py               # Production server config
│   ├── requirements.txt               # Python dependencies
│   └── .env.example                   # Environment variable template
│
└── flutter_app/
    ├── lib/
    │   ├── main.dart                  # App entry + Material3 theme
    │   ├── models/
    │   │   └── chat_message.dart      # Message data model
    │   ├── services/
    │   │   ├── api_service.dart       # ⭐ HTTP client (platform auto-detect)
    │   │   └── storage_service.dart   # Local persistence (SharedPreferences)
    │   ├── screens/
    │   │   └── chat_screen.dart       # Main chat UI
    │   └── widgets/
    │       ├── chat_bubble.dart       # Message bubbles (Markdown support)
    │       ├── message_input_bar.dart # Text input + send button
    │       ├── quick_questions.dart   # Suggestion chips
    │       └── typing_indicator.dart  # Animated dots while waiting
    └── pubspec.yaml
```

---

## 🚀 Setup Guide (Step by Step)

### Prerequisites

| Tool | Min Version | Download |
|------|-------------|---------|
| Python | 3.10+ | https://python.org |
| Flutter | 3.19+ | https://flutter.dev |
| Ollama | latest | https://ollama.com |

---

### STEP 1 — Install and Start Ollama

```bash
# 1. Download Ollama from https://ollama.com and install

# 2. Pull the LLaMA3 model (~4.7 GB):
ollama pull llama3

# 3. Verify it works:
ollama run llama3
# Type "hello" — if it responds, Ollama is working. Press Ctrl+D to exit.

# 4. Ollama runs as a background service automatically.
# Verify: http://localhost:11434 should show "Ollama is running"
```

---

### STEP 2 — Backend Setup

```bash
# 1. Navigate to the backend folder:
cd university_chatbot_optimized/backend

# 2. Create a virtual environment:
python -m venv venv

# Activate it:
# Windows:   venv\Scripts\activate
# Mac/Linux: source venv/bin/activate

# 3. Install dependencies:
pip install -r requirements.txt
# ⚠️  First install takes 5–15 min (downloads PyTorch ~700MB + models)

# 4. Set up environment:
cp .env.example .env
# No changes needed for local development

# 5. Create database:
python manage.py migrate

# 6. Build the FAISS index (do this ONCE before first run):
python build_index.py
# Downloads embedding model (~90MB) and builds the index.
# Takes 2–5 min. Look for: "✅ FAISS index built and saved"

# 7. Start the Django server:
python manage.py runserver
```

✅ Server running at: **http://localhost:8000**  
✅ API info: **http://localhost:8000/**  
✅ Admin panel: **http://localhost:8000/admin/**

---

### STEP 3 — Flutter App Setup

```bash
# 1. Navigate to Flutter app:
cd university_chatbot_optimized/flutter_app

# 2. Install Flutter packages:
flutter pub get

# 3. The API URL is already configured for Flutter Web.
#    If using a different platform, edit lib/services/api_service.dart:
#
#    const _RunTarget _target = _RunTarget.web;          ← Flutter Web (default)
#    const _RunTarget _target = _RunTarget.androidEmu;   ← Android Emulator
#    const _RunTarget _target = _RunTarget.androidDevice; ← Physical phone

# 4. Run:
flutter run -d chrome          # Flutter Web (recommended)
flutter run                    # Connected device or emulator
```

---

### STEP 4 — Test the API

```bash
# Health check
curl http://localhost:8000/health

# Engine status
curl http://localhost:8000/api/chatbot/status

# Send a question
curl -X POST http://localhost:8000/api/chatbot \
  -H "Content-Type: application/json" \
  -d '{"message": "What programs does the university offer?", "session_id": "test-1"}'

# Expected response:
# {
#   "reply": "The university offers BS Computer Science...",
#   "response_time_ms": 2800,
#   "sources_count": 3
# }
```

---

## 📡 API Reference

| Method | URL | Description | Auth |
|--------|-----|-------------|------|
| `GET` | `/` | API info & endpoints | None |
| `GET` | `/health` | Health check | None |
| `POST` | `/api/chatbot` | **Send message, get AI reply** | None |
| `GET` | `/api/chatbot/status` | RAG engine status + doc count | None |
| `GET` | `/api/chatbot/history?session_id=X&limit=20` | Session history | None |
| `POST` | `/api/chatbot/rebuild` | Rebuild FAISS index | `X-Admin-Key` header |
| `GET` | `/admin/` | Django admin panel | Admin login |

### Chat Request / Response

```json
// POST /api/chatbot
// Request:
{ "message": "What is the BS CS fee?", "session_id": "optional-uuid" }

// Response 200:
{
  "reply": "The BS CS fee per semester is PKR 53,000...",
  "response_time_ms": 2341,
  "sources_count": 3
}

// Response 400 (invalid input):
{ "error": "message: This field is required." }

// Response 503 (engine initializing):
{ "error": "The AI engine is still initializing.", "detail": "..." }
```

---

## 🔄 Updating University Data

1. Edit `backend/data/university_data.json`
2. Rebuild the index:

```bash
# Option A — server not running:
python build_index.py

# Option B — server running (live rebuild via API):
curl -X POST http://localhost:8000/api/chatbot/rebuild \
  -H "X-Admin-Key: university-admin-secret-key-2024"

# Response: {"message": "Index rebuilt successfully.", "doc_count": 92}
```

---

## 📱 Platform Configuration

Edit `flutter_app/lib/services/api_service.dart`:

```dart
// Flutter Web (browser) — DEFAULT, auto-detected:
const _RunTarget _target = _RunTarget.web;
// → URL: http://127.0.0.1:8000/api

// Android Emulator:
const _RunTarget _target = _RunTarget.androidEmu;
// → URL: http://10.0.2.2:8000/api

// Physical Android/iOS (same WiFi as your PC):
const _RunTarget _target = _RunTarget.androidDevice;
const String _deviceLanIp = '192.168.1.YOUR_IP';  // ← change this
// Find your IP: ipconfig (Windows) | ifconfig (Mac/Linux)

// Production server:
const _RunTarget _target = _RunTarget.production;
const String _productionUrl = 'https://your-server.com/api';
```

---

## 🐛 Troubleshooting

| Symptom | Cause | Fix |
|---------|-------|-----|
| Request timed out | llama3 still loading (first run) | Wait 60s and retry — auto-retry is built in |
| Cannot connect to server | Django not running | `python manage.py runserver` |
| Cannot connect on mobile | Wrong `_target` in api_service.dart | Set to `androidDevice`, update `_deviceLanIp` |
| FAISS index not found | `build_index.py` not run | `python build_index.py` |
| Import errors on startup | Packages not installed | `pip install -r requirements.txt` |
| Admin 500 error | DB not migrated | `python manage.py migrate` |
| Blank or wrong answers | Ollama not running | `ollama run llama3` |
| "Engine not ready" on status | First-time embedding model download | Wait 2-5 min, then check status again |
| Rebuild returns 401 | Wrong admin key | Check `ADMIN_API_KEY` in `.env` |

---

## 🚀 Production Deployment

```bash
# On your Linux server:

# 1. Install and pull the model
ollama pull llama3

# 2. Deploy backend
cd backend
pip install -r requirements.txt
python manage.py migrate
python build_index.py

# 3. Set production environment variables
export DEBUG=False
export SECRET_KEY=$(python -c "import secrets; print(secrets.token_urlsafe(50))")
export ALLOWED_HOSTS=your-domain.com
export ADMIN_API_KEY=$(python -c "import secrets; print(secrets.token_urlsafe(32))")
export CORS_ALLOWED_ORIGINS=https://your-domain.com

# 4. Start with Gunicorn
gunicorn -c gunicorn.conf.py university_chatbot.wsgi:application

# 5. Build Flutter APK (for mobile distribution)
cd ../flutter_app
flutter build apk --release
# APK: build/app/outputs/flutter-apk/app-release.apk
```

---

## ⚙️ Tech Stack

| Layer | Technology | Notes |
|-------|-----------|-------|
| Backend | Django 4.2 + DRF 3.15 | REST API, admin panel, SQLite logging |
| RAG Pipeline | LangChain 0.2.x (LCEL) | Document loading, chunking, MMR retrieval |
| Vector DB | FAISS CPU | Fast local similarity search, disk-persistent |
| Embeddings | all-MiniLM-L6-v2 | ~90MB, CPU-friendly, 384-dim vectors |
| LLM | Ollama + LLaMA3 | 100% local, no API key or internet needed |
| Frontend | Flutter 3 + Material3 | Android, iOS, and Web from one codebase |

**Running cost: $0/month** ✅ Everything runs locally.
