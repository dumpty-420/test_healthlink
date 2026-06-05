# 🏥 HealthLink — Smart Health Management System

An AI-powered health management system that analyzes symptoms, recommends doctors, and schedules appointments using Google Gemini AI and Pinecone RAG technology. Deployed on **Google Cloud Run** with a full CI/CD pipeline.

> ⚠️ **Disclaimer:** This is an educational project and NOT a substitute for professional medical advice, diagnosis, or treatment. Always seek the advice of qualified health providers with questions about medical conditions.

---

## 🚀 Live Demo

- **Backend API:** [https://healthlink-dl7kvw447a-el.a.run.app/docs](https://healthlink-dl7kvw447a-el.a.run.app/docs)
- **Health Check:** [https://healthlink-dl7kvw447a-el.a.run.app/api/v1/health](https://healthlink-dl7kvw447a-el.a.run.app/api/v1/health)

---

## 📋 Overview

HealthLink is a production-ready educational project demonstrating modern GenAI application architecture with:

- **FastAPI** backend with function-based multi-agent architecture (Monolithic Function-Based Design — NO classes in agents)
- **Google Gemini 2.0** for intelligent symptom analysis and recommendations
- **Pinecone** vector database for RAG (Retrieval-Augmented Generation)
- **LangChain 1.x** for LLM orchestration
- **SQLite** database for doctors and appointments
- **Docker & Cloud deployment** ready (Google Cloud Run, Hugging Face Spaces)

---

## 🏗️ Architecture

### 4-Agent Pipeline

```
User Input
    │
    ▼
┌─────────────────────┐
│  Symptom Agent      │  → Extracts symptoms, severity, urgency level
└──────────┬──────────┘
           │
    ▼
┌─────────────────────┐
│  Doctor Agent       │  → Matches symptoms to specialists via RAG + Pinecone
└──────────┬──────────┘
           │
    ▼
┌─────────────────────┐
│  Scheduling Agent   │  → Generates appointment slots based on urgency
└──────────┬──────────┘
           │
    ▼
┌─────────────────────┐
│  Summary Agent      │  → Generates health summary and next steps
└─────────────────────┘
```

### Tech Stack

| Layer | Technology |
|---|---|
| LLM | Google Gemini 2.0 Flash |
| Orchestration | LangChain 1.x |
| Vector Store | Pinecone (RAG) |
| Backend | FastAPI + Uvicorn |
| Database | SQLite (SQLAlchemy) |
| Frontend | React/Vite + Streamlit |
| Containerization | Docker |
| Cloud | GCP Cloud Run |
| CI/CD | GitHub Actions + Workload Identity Federation |

---

## 📁 Project Structure

```
Smart-Health-Management-System/
├── agents/                    # Pure function agents (NO classes)
│   ├── symptom_agent.py      # Symptom extraction & urgency assessment
│   ├── doctor_agent.py       # Doctor matching & recommendations
│   ├── scheduling_agent.py   # Appointment slot generation
│   └── summary_agent.py      # Health summary generation
├── api/
│   └── routes.py             # FastAPI endpoints
├── config/
│   ├── settings.py           # Pydantic settings
│   └── logging.py            # Simple Python logging
├── core/
│   ├── llm.py               # LangChain + Gemini client
│   ├── rag.py               # Pinecone vector store
│   ├── database.py          # SQLAlchemy models
│   ├── schemas.py           # Pydantic schemas
│   └── orchestrator.py      # Agent orchestration
├── data/
│   ├── doctors.csv          # 100 doctors across 30 specialties
│   ├── symptoms_kb.json     # 200+ medical conditions KB
│   └── healthlink.db        # SQLite database (auto-generated)
├── ui/
│   └── streamlit_app.py     # Streamlit chat interface
├── utils/
│   ├── helpers.py           # Helper functions
│   └── validators.py        # Input validation
├── tests/
│   ├── test_api.py
│   ├── test_agents.py
│   └── test_e2e.py
├── .github/workflows/
│   └── deploy.yaml          # CI/CD pipeline (GitHub Actions)
├── main.py                  # FastAPI entry point
├── test_e2e.py              # End-to-end test script
├── expand_datasets.py       # Dataset expansion utility
├── Dockerfile               # Docker for Cloud Run
├── Dockerfile.huggingface   # Docker for Hugging Face Spaces
├── docker-compose.yml       # Local Docker setup
├── requirements.txt         # Python dependencies
└── pyproject.toml
```

---

## ⚙️ Setup & Installation

### Prerequisites

- Python 3.12+ (Required — LangChain 1.x needs Python 3.10+)
- Docker (optional)
- Gemini API Key → [aistudio.google.com](https://aistudio.google.com)
- Pinecone API Key → [pinecone.io](https://www.pinecone.io)

### Local Setup

```bash
# Clone the repo
git clone https://github.com/dumpty-420/test_healthlink.git
cd test_healthlink

# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Configure environment
cp .env.example .env
# Add your API keys to .env

# Run the backend
python main.py

# Run Streamlit UI (separate terminal)
streamlit run ui/streamlit_app.py
```

Access:
- API docs: `http://localhost:8000/docs`
- Streamlit UI: `http://localhost:8501`

### Docker Setup

```bash
# Using Docker Compose (recommended)
docker-compose up --build

# Or individual build
docker build -t healthlink:latest .

docker run -d \
  --name healthlink-app \
  -p 8000:8000 \
  --env-file .env \
  healthlink:latest
```

---

## 🔑 Environment Variables

### Required

| Variable | Description |
|---|---|
| `GEMINI_API_KEY` | Google Gemini API key |
| `PINECONE_API_KEY` | Pinecone vector DB key |

### Optional

| Variable | Description | Default |
|---|---|---|
| `LLM_MODEL_NAME` | Gemini model name | `gemini-2.0-flash` |
| `LLM_TEMPERATURE` | LLM temperature | `0.2` |
| `LLM_MAX_TOKENS` | Max output tokens | `2048` |
| `EMBEDDING_MODEL_NAME` | Embedding model | `models/gemini-embedding-001` |
| `PINECONE_ENVIRONMENT` | Pinecone environment | `us-east-1` |
| `PINECONE_INDEX_NAME` | Pinecone index name | `healthlink` |
| `RAG_TOP_K` | RAG retrieval count | `5` |
| `CHUNK_SIZE` | Text chunk size | `500` |
| `CHUNK_OVERLAP` | Chunk overlap | `50` |
| `DATABASE_URL` | Database URL | `sqlite:///./data/healthlink.db` |
| `LOG_LEVEL` | Logging level | `INFO` |

---

## 📡 API Endpoints

### POST `/api/v1/assess`
Run a health assessment.

```json
{
  "user_input": "I have severe headache and fever for 3 days",
  "user_id": "user123",
  "session_id": "session456",
  "preferred_date": "2026-06-10"
}
```

**Response:**
```json
{
  "request_id": "uuid",
  "timestamp": "2026-01-01T00:00:00",
  "symptom_analysis": {
    "symptoms": [{"name": "headache", "severity": "severe", "duration": "3 days"}],
    "urgency_level": "high",
    "confidence_score": 0.95
  },
  "doctor_recommendations": {
    "recommended_doctors": [{"name": "Dr. Sarah Johnson", "specialty": "Neurology", "rating": 4.8}]
  },
  "scheduling_options": {
    "recommended_slot": {"doctor_name": "Dr. Sarah Johnson", "date": "2026-06-08", "time": "09:00"},
    "available_slots": [...]
  },
  "health_summary": {
    "summary": "...",
    "recommended_actions": [...]
  }
}
```

### GET `/api/v1/health`
Health check.

### GET `/api/v1/doctors?specialty=Cardiology&limit=10`
List doctors with optional specialty filter.

### GET `/api/v1/specialties`
List all available specialties.

---

## 🚀 CI/CD Pipeline

Automated pipeline on every push to `main`:

```
Push to main
    │
    ├── Run Tests (pytest)
    ├── Build Docker Image
    ├── Push to Artifact Registry
    ├── Deploy to Cloud Run
    └── Health Check ✅
```

Auth via **Workload Identity Federation** — no service account keys needed.

### GitHub Secrets Required

| Secret | Description |
|---|---|
| `GEMINI_API_KEY` | Gemini API key |
| `PINECONE_API_KEY` | Pinecone API key |

---

## ☁️ Cloud Deployments

### Google Cloud Run (Primary)

```bash
gcloud run deploy healthlink \
  --image asia-south1-docker.pkg.dev/project-bd2948f0-db04-4da3-833/healthlink-repo/healthlink:latest \
  --platform managed \
  --region asia-south1 \
  --allow-unauthenticated \
  --memory 2Gi \
  --cpu 2 \
  --port 8000
```

### Hugging Face Spaces (Alternative)

1. Create a new Docker Space on [huggingface.co](https://huggingface.co)
2. Copy all files including `Dockerfile.huggingface`
3. Add secrets: `GEMINI_API_KEY`, `PINECONE_API_KEY`
4. Push and deploy

---

## 🧪 Testing

```bash
# Run all tests
pytest tests/ -v

# Run with coverage
pytest tests/ -v --cov=. --cov-report=html

# Run end-to-end test script
python test_e2e.py
```

End-to-end tests cover:
- Health check endpoint
- Low urgency scenarios (common cold)
- Medium urgency scenarios (persistent pain)
- High urgency scenarios (chest pain)
- Various medical specialties
- Doctor and specialty endpoints

---

## 🎯 Features

- ✅ Multi-agent symptom analysis with urgency assessment
- ✅ RAG-enhanced medical knowledge using Pinecone
- ✅ Doctor recommendation engine (100 doctors, 30 specialties)
- ✅ Smart appointment scheduling based on urgency
- ✅ Comprehensive health summaries
- ✅ Google Gemini 2.0 integration
- ✅ LangChain 1.x with structured outputs
- ✅ Simple Python logging (no external dependencies)
- ✅ Input validation and error handling
- ✅ Streamlit UI (optional)
- ✅ Docker support (Cloud Run + Hugging Face)
- ✅ CI/CD pipeline (GitHub Actions + Workload Identity Federation)
- ✅ End-to-end testing
- ✅ Expanded datasets (100 doctors, 200+ conditions)

---

## 👤 Author

**Seerat Chugh**
- GitHub: [@dumpty-420](https://github.com/dumpty-420)
- LinkedIn: [seerat-chugh](https://linkedin.com/in/seerat-chugh)
- Email: seeratac@gmail.com

---

## 📄 License

MIT License — free to use for educational purposes.
