# KnowledgeVault

KnowledgeVault is a full-stack AI knowledge assistant that helps teams turn private documents into a searchable, source-backed question-answering system.

Upload company files, index them into a vector database, and ask natural-language questions against your own knowledge base. KnowledgeVault retrieves the most relevant document chunks, sends grounded context to an LLM, and returns answers with source references so users can trust where the response came from.

## What KnowledgeVault Solves

Most teams store important knowledge across PDFs, Word documents, text files, wikis, shared drives, onboarding docs, policies, reports, and internal notes. Finding the right answer often means opening multiple files, scanning pages manually, and hoping the newest document is the one being used.

KnowledgeVault gives teams a private AI assistant for that scattered knowledge.

Instead of searching file names or scrolling through folders, users can ask questions directly:

```text id="mpk51c"
What is our refund policy for enterprise customers?
Which onboarding steps are required before production access?
Summarize the compliance requirements mentioned in the uploaded documents.
What does the architecture document say about database backups?
```

The system answers from uploaded documents, not from vague memory.

## Key Features

### Secure User Accounts

KnowledgeVault includes authentication with password hashing and JWT-based access control. Each user has their own document space, so uploaded files and retrieved chunks are scoped to the authenticated user.

### Private Document Library

Users can upload files, list their documents, retrieve document metadata, process documents into searchable chunks, and delete documents they own.

### RAG Question Answering

The chat flow embeds the user question, searches indexed document chunks using vector similarity, builds a context-aware prompt, and generates an answer through an OpenAI chat model.

### Source-Backed Responses

Retrieved chunks are returned with answer payloads so users can inspect the source material behind each response.

### Vector Search with pgvector

KnowledgeVault stores generated embeddings in PostgreSQL using pgvector. This keeps document metadata, users, analytics, and vector records inside a relational data layer.

### Document Processing Pipeline

The ingestion flow downloads uploaded files from storage, extracts text, chunks content, generates embeddings, stores vectors, updates document status, and tracks indexing cost.

### Usage Analytics

The backend records analytics events for RAG queries, including retrieval time, LLM response time, cost estimates, answer quality signals, and no-answer tracking.

### Modern Full-Stack UI

The frontend is built with Next.js, React, TypeScript, Tailwind CSS, shadcn-style components, and a clean chat-oriented interface.

### Docker-Based Local Development

The project includes Docker Compose services for the frontend, backend, and PostgreSQL with pgvector.

## Tech Stack

### Frontend

* Next.js
* React
* TypeScript
* Tailwind CSS
* shadcn/ui-style components
* Axios
* React Hook Form
* Zod
* Lucide icons
* Sonner notifications

### Backend

* Python 3.12
* FastAPI
* SQLAlchemy
* Pydantic
* Pydantic Settings
* LangChain
* OpenAI chat models
* VoyageAI embeddings
* PyMuPDF
* python-docx
* JWT authentication
* Argon2 password hashing
* Supabase client
* Uvicorn

### Database and Storage

* PostgreSQL
* pgvector
* Supabase storage

### DevOps

* Docker Compose
* Multi-service local environment
* Environment-based configuration
* Health and observability middleware

## Architecture Overview

KnowledgeVault follows a modular backend architecture. The FastAPI application initializes separate modules for authentication, chat, documents, health checks, and analytics.

```text id="sogjmt"
Frontend
   ↓
Next.js UI
   ↓
FastAPI Backend
   ↓
Auth Module       Documents Module       Chat Module       Analytics Module
   ↓                    ↓                    ↓                    ↓
JWT Access       Upload and Indexing     RAG Retrieval       Usage Events
   ↓                    ↓                    ↓                    ↓
PostgreSQL       Supabase Storage        pgvector Search     PostgreSQL
```

## RAG Flow

```text id="7bo1g5"
1. User uploads a document
2. File is saved to storage
3. Document metadata is stored in PostgreSQL
4. User triggers indexing
5. Pipeline downloads the file into a temporary workspace
6. Text is extracted from the document
7. Content is split into chunks
8. Embeddings are generated with VoyageAI
9. Chunks and vectors are stored in PostgreSQL with pgvector
10. User asks a question
11. Question is embedded
12. Relevant chunks are retrieved by vector similarity
13. Context is sent to the LLM
14. Answer and source chunks are returned
15. Analytics event is recorded
```

## Repository Structure

```text id="bshc50"
.
├── backend/
│   ├── app/
│   │   ├── core/
│   │   │   ├── database.py
│   │   │   ├── environment.py
│   │   │   ├── exceptions.py
│   │   │   └── observability.py
│   │   ├── modules/
│   │   │   ├── analytics/
│   │   │   ├── auth/
│   │   │   ├── chat/
│   │   │   ├── documents/
│   │   │   ├── health/
│   │   │   └── users/
│   │   └── main.py
│   ├── pyproject.toml
│   └── Dockerfile
├── frontend/
│   ├── app/
│   ├── components/
│   ├── package.json
│   └── pnpm-lock.yaml
├── docs/
│   ├── c4/
│   └── data-model/
├── docker-compose.yml
├── .env.example
└── readme.md
```

## Data Model

KnowledgeVault uses a practical enterprise knowledge schema:

### users

Stores account and authentication details such as email, hashed password, and display name.

### documents

Stores uploaded document metadata, owner relationship, file type, processing status, storage path, and file information.

### chunk_records / embeddings

Stores extracted text fragments and their vector embeddings for similarity search.

### analytics_events

Stores system and usage events such as chat queries, ingestion activity, latency, costs, and pipeline errors.

## Local Setup

### Prerequisites

Make sure you have:

* Docker
* Docker Compose
* Node.js if running frontend outside Docker
* Python 3.12 if running backend outside Docker
* OpenAI API key
* VoyageAI API key
* Supabase project credentials if using Supabase storage

## Environment Variables

Create a `.env` file in the project root.

You can start from `.env.example`:

```env id="nuf2u7"
POSTGRES_USER=your_postgres_user
POSTGRES_PASSWORD=your_postgres_password
POSTGRES_DB=your_database_name

POSTGRES_PORT=15432
BACKEND_PORT=8000

DATABASE_URL=postgresql+psycopg://your_postgres_user:your_postgres_password@db:5432/your_database_name

secret_key=your_secret_key_here
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30

VOYAGE_API_KEY=your_voyage_api_key_here
OPENAI_API_KEY=your_openai_api_key_here
```

Depending on your storage implementation, also configure the required Supabase values used by the backend.

## Running with Docker Compose

From the project root:

```bash id="lasrcw"
docker compose up --build
```

This starts:

* Frontend on port `3000`
* Backend on the configured backend port, usually `8000`
* PostgreSQL with pgvector on the configured database port

Open the frontend:

```text id="26u5qc"
http://localhost:3000
```

Open the backend API:

```text id="umc3mv"
http://localhost:8000
```

## Running the Backend Manually

Go into the backend directory:

```bash id="j6dexz"
cd backend
```

Create a virtual environment:

```bash id="gxrdu7"
python -m venv .venv
```

Activate it on macOS or Linux:

```bash id="c0g3wh"
source .venv/bin/activate
```

Activate it on Windows CMD:

```bat id="f2faiu"
.venv\Scripts\activate
```

Install dependencies with `uv`:

```bash id="uxm7tr"
uv sync
```

Or install from the project metadata with pip:

```bash id="o4c0mp"
pip install -e .
```

Run the backend:

```bash id="9w3cxv"
uvicorn app.main:app --reload --host 127.0.0.1 --port 8000
```

## Running the Frontend Manually

Go into the frontend directory:

```bash id="g28xzi"
cd frontend
```

Install dependencies:

```bash id="zmbxae"
pnpm install
```

Run the development server:

```bash id="ypvmsb"
pnpm dev
```

Open:

```text id="j1y6d9"
http://localhost:3000
```

## Main Backend Modules

### Auth Module

Handles user registration, login, password hashing, JWT creation, and authenticated user context.

### Documents Module

Handles upload, listing, deletion, document ownership checks, storage interaction, and indexing triggers.

### Chat Module

Handles user questions, vector retrieval, LLM response generation, returned sources, and RAG analytics.

### Analytics Module

Stores operational events such as query completion, retrieval timing, response quality, and cost metadata.

### Health Module

Provides simple service health endpoints for deployment and local checks.

## Document Lifecycle

```text id="i8umkm"
uploaded → processing → processed
                ↓
              failed
```

When a document is uploaded, it is stored first and marked as uploaded. When indexing starts, the system marks it as processing. If extraction, chunking, and embedding succeed, the document becomes processed. If any step fails, the status becomes failed.

## Chat Behavior

When a user submits a query:

1. KnowledgeVault generates an embedding for the question.
2. It searches only the authenticated user’s documents.
3. It retrieves the top matching chunks.
4. If nothing relevant is found, it returns a safe fallback answer.
5. If chunks are found, it asks the LLM to answer using only the provided context.
6. It returns the final answer plus source chunks.
7. It records timing, cost, and quality metadata.

The prompt instructs the model to say it does not have the information when the context does not contain the answer.

## API Capabilities

KnowledgeVault exposes API flows for:

* User authentication
* Document upload
* Document listing
* Document deletion
* Document indexing
* Chat-based question answering
* Analytics tracking
* Health checks

Exact routes may vary by module router configuration.

## Cost Tracking

The system includes pricing utilities for estimating:

* Embedding/indexing cost
* Vector retrieval cost
* LLM response cost

These values are stored in analytics metadata, which makes the project useful for production-style monitoring and future billing dashboards.

## Security Notes

KnowledgeVault includes several important security foundations:

* Password hashing
* JWT-based authentication
* Per-user document access checks
* User-scoped vector retrieval
* Forbidden access errors for documents owned by another user

Before production deployment, consider adding:

* Rate limiting
* Organization-level tenancy
* Refresh tokens
* Audit logs
* File scanning
* Upload size limits
* Background jobs for indexing
* Role-based permissions
* Secrets management
* Production CORS restrictions

## Product Use Cases

KnowledgeVault can be adapted for:

* Internal company knowledge bases
* HR and onboarding assistants
* Technical documentation search
* Customer support knowledge retrieval
* Legal and compliance document Q&A
* Sales enablement libraries
* Research document exploration
* Policy and procedure lookup
* Engineering architecture documentation
* Private team wiki search

## Roadmap Ideas

* Multi-tenant organizations
* Team workspaces
* Role-based access control
* Redis caching
* Background indexing workers
* Streaming answer UI
* Admin analytics dashboard
* Hybrid keyword plus vector search
* Document versioning
* Support for spreadsheets and HTML pages
* Citation previews in the chat UI
* Evaluation suite for retrieval quality
* SSO support
* Slack or Teams integration

## Why KnowledgeVault

The name reflects the project’s purpose: a secure place where company knowledge is stored, indexed, searched, and transformed into reliable AI answers.

It is not just a chatbot. It is a vault for internal knowledge with retrieval, ownership, citations, analytics, and a clean production-oriented backend.

## License

This project is licensed under the MIT License.

## Author

Built by Alex Ariza Herrera.
