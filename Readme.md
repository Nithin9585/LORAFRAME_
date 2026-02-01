# LORAFRAME (IDLock Engine)

**Persistent Character Memory & Video Generation System**

LORAFRAME is an advanced AI pipeline designed to generate consistent, evolving digital characters. It combines episodic memory, LLM reasoning, and identity-preservation technology to create "permanent digital actors" that remember their past scenes and maintain visual consistency across thousands of generated images and videos.

##  Key Features

*   **Identity Retention (IDLock)**: Leverages InsightFace and semantic vector embeddings to ensure character faces remain consistent across generations.
*   **Episodic Memory**: Characters "remember" previous scenes (context, injuries, clothing changes) via RAG (Retrieval Augmented Generation) and Vector DBs.
*   **Prompt Engine**: Uses **Groq (Llama 3)** to convert simple user prompts into rich, context-aware scene descriptions that respect the character's history.
*   **Multi-Modal Output**: Generates high-quality Images and Videos using **Google Gemini** and **Imagen/Veo** models.
*   **Self-Healing**: Automated "Refiner" loop detects identity drift and repairs faces if they deviate from the character's canonical look.

---
**Eye Level**
<img width="486" height="687" alt="image" src="https://github.com/user-attachments/assets/5bc45e90-8ff3-41f1-8702-8f2d552efc80" />

**high angle**
<img width="505" height="687" alt="image" src="https://github.com/user-attachments/assets/db6908d2-9eee-4825-9d2e-65e3fa9d4ca4" />


**low angle**
<img width="468" height="690" alt="image" src="https://github.com/user-attachments/assets/728c0182-c073-4f5a-b677-ad4aaec03412" />


**low angle**
<img width="290" height="408" alt="image" src="https://github.com/user-attachments/assets/c094914a-3dcf-402c-89a0-927c304401de" />

**BirdEyes**
<img width="501" height="688" alt="image" src="https://github.com/user-attachments/assets/292daff9-33dd-468b-a253-2ca62ca494d1" />


**dutch angle**
<img width="501" height="678" alt="image" src="https://github.com/user-attachments/assets/31cd3a9d-c1db-40c3-9028-b2bccb31159d" />

**Worm's Eye View**
<img width="466" height="689" alt="image" src="https://github.com/user-attachments/assets/f127d692-0b50-4b8a-b3d0-100ccdbdc144" />

**Over The Shoulder**
<img width="487" height="677" alt="image" src="https://github.com/user-attachments/assets/39fcbcda-27fc-4f36-8656-613b4b8ed9ae" />

**Oblique Angle**
<img width="464" height="684" alt="image" src="https://github.com/user-attachments/assets/9d6d082a-4c7c-4543-81c6-4dd6d09af7b9" />

**Point Of View**
<img width="454" height="675" alt="image" src="https://github.com/user-attachments/assets/15f51df9-d690-44fe-8257-fc76e993ec82" />

<img width="449" height="672" alt="image" src="https://github.com/user-attachments/assets/4af42b01-b8ed-43b5-a472-a3efc462ed54" />




##  Architecture

```mermaid
graph TD

  %% ===== CLIENT LAYER =====
  U["User / Client App"] --> API["API Gateway (FastAPI)"]

  %% ===== JOB QUEUE =====
  API --> Q["Redis Queue (RQ)"]

  %% ===== WORKERS =====
  Q --> GEN["Generator Worker"]
  Q --> REF["Refiner Worker"]
  Q --> STA["State Analyzer Worker"]
  Q --> COL["LoRA Collector Worker"]
  Q --> TRN["LoRA Trainer Worker"]

  %% ===== DATA LAYER =====
  PG["Postgres Metadata DB"]
  VDB["Vector DB (Embeddings)"]
  OBJ["Object Storage (Images / Models)"]

  %% ===== MEMORY =====
  GEN --> PG
  GEN --> VDB
  STA --> PG
  STA --> VDB

  %% ===== PROMPT ENGINE =====
  GEN --> LLM["LLM Prompt Engine"]

  %% ===== LORA SYSTEM =====
  GEN --> LR["LoRA Registry"]
  LR --> GEN
  COL --> TRN
  TRN --> OBJ
  TRN --> LR
  LR --> PG

  %% ===== GENERATION =====
  GEN --> AI["Image / Video Generator"]
  AI --> OBJ

  %% ===== VALIDATION =====
  AI --> VAL["Vision Validator (IDR)"]
  VAL -->|Pass| STA
  VAL -->|Fail| REF

  %% ===== REFINEMENT LOOP =====
  REF --> AI

  %% ===== LORA DATA PIPELINE =====
  STA --> COL

  %% ===== ADMIN UI =====
  API --> UI["Admin / Dashboard"]

```

---

##  Technology Stack

### Frontend
*   **Framework**: React.js
*   **Language**: JavaScript

### Backend
*   **Framework**: Python 3.10+, FastAPI
*   **Database**: PostgreSQL (SQLAlchemy), Redis (Caching/Queues)
*   **Vector Database**: Pinecone / Milvus / FAISS
*   **LLM Inference**: Groq (Llama 3 70B/8B)
*   **Image/Video Gen**: Google Gemini Pro Vision, Imagen 2, Veo
*   **Computer Vision**: InsightFace, ONNX Runtime (Identity extraction & IDR validation)
*   **Storage**: Google Cloud Storage (GCS) or AWS S3

---

##  Getting Started

### Prerequisites

*   Python 3.10+
*   PostgreSQL & Redis (or Docker)
*   API Keys:
    *   `GOOGLE_API_KEY` (Gemini/PaLM)
    *   `GROQ_API_KEY`
    *   `PINECONE_API_KEY` (Optional, defaults to local FAISS if not set)

    pip install -r requirements.txt
    



---

##  Deployment URLs

**Frontend (Live App):** [https://lore-frame-in.vercel.app](https://lore-frame-in.vercel.app)

**Backend API (GCP):** [Online API Docs (Swagger UI)](https://cineai-api-4sjsy6xola-uc.a.run.app/docs)

---

## API Documentation

Detailed endpoint documentation is available in [API_DOCUMENTATION.md](API_DOCUMENTATION.md).

### Core Endpoints

*   `POST /api/v1/characters`: Create a new character from reference images.
*   `POST /api/v1/generate`: Generate a consistent image for a character.
*   `POST /api/v1/video/generate`: Generate a video scene.
*   `GET /api/v1/jobs/{job_id}`: Check generation status.

---

##  Project Structure

```
cineAI/
├── app/
│   ├── api/            # API Routes (characters, generate, video)
│   ├── core/           # Config, Database, Redis setup
│   ├── models/         # SQLAlchemy Database Models
│   ├── schemas/        # Pydantic Request/Response Models
│   ├── services/       # Core Logic (Groq, Gemini, MemoryEngine)
│   └── workers/        # Async Task Workers
├── scripts/            # Utility scripts
├── tests/              # Pytest suite
├── uploads/            # Local storage for dev
├── .env.example        # Environment variable template
├── requirements.txt    # Python dependencies
└── README.md           # This file
```
