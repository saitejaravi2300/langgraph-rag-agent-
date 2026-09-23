# 🤖 LangGraph RAG Agent

An **agentic Retrieval-Augmented Generation (RAG) application** developed using **FastAPI** and **LangGraph**. The system combines document retrieval, web search, persistent conversational memory, and streaming LLM responses into a single AI assistant.

The application uses **PostgreSQL with pgvector** for semantic document search, **LangGraph PostgreSQL checkpointers** for conversation persistence, and a **Streamlit** interface for interacting with the agent.

## 🚀 Key Features

* **Agentic RAG with LangGraph** – Uses a ReAct-style workflow to decide when to retrieve documents or perform web searches.
* **Real-time streaming** – Responses are streamed incrementally from the backend to the frontend.
* **Persistent conversations** – Conversations are organized into individual threads with history stored using LangGraph checkpointers.
* **Semantic document search** – Uploaded documents are converted into embeddings and stored in PostgreSQL/pgvector.
* **User authentication** – JWT-based signup, login, refresh-token handling, and user-specific data isolation.
* **Multi-format document ingestion** – Supports PDF, DOCX, and TXT documents.
* **Tool-based reasoning** – The agent can use document retrieval and Tavily web search as external tools.
* **Async backend architecture** – Built around FastAPI and SQLAlchemy 2.0 asynchronous APIs.
* **Health and logging support** – Includes backend health checks and structured application logging.

## 💻 Technology Stack

### Backend

* FastAPI
* LangGraph
* LangChain
* SQLAlchemy 2.0
* Pydantic v2

### Database & Vector Search

* PostgreSQL
* pgvector
* `langchain-postgres`
* LangGraph PostgreSQL Checkpointer

### Frontend

* Streamlit

### AI / LLM

* OpenAI-compatible LLM APIs
* Configurable embedding models
* LangChain tools and retrievers

## 📋 Prerequisites

Before running the project, make sure the following are installed:

* Python 3.12+
* Docker
* Docker Compose
* PostgreSQL with pgvector
* API key for the selected LLM provider
* Tavily API key if web search is enabled

## 📦 Quick Start with Docker

### 1. Configure environment variables

Copy the example environment file:

```bash
cp env.example .env
```

Update the `.env` file with your API keys, database credentials, model configuration, and JWT settings.

### 2. Start the application

Run:

```bash
docker compose up --build
```

After the containers start successfully, the application will be available at:

* **Backend API:** `http://localhost:8000/api/v1`
* **Swagger Documentation:** `http://localhost:8000/api/v1/docs`
* **ReDoc:** `http://localhost:8000/api/v1/redoc`
* **Streamlit Frontend:** `http://localhost:8501`

> **Note:** The recommended PostgreSQL image is `pgvector/pgvector:pg16`.
> If you are using an existing PostgreSQL installation, make sure the pgvector extension is enabled.

```sql
CREATE EXTENSION IF NOT EXISTS vector;
```

## 🧰 Running Locally

### Backend Setup

Navigate to the backend directory:

```bash
cd backend
```

Create and activate a virtual environment:

```bash
python -m venv .venv
```

For Windows:

```bash
.venv\Scripts\activate
```

For Linux/macOS:

```bash
source .venv/bin/activate
```

Install the required dependencies:

```bash
pip install -r requirements.txt
```

### Start PostgreSQL with pgvector

If PostgreSQL is not already running, Docker can be used:

```bash
docker run --name langgraph_postgres \
  -p 5432:5432 \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_PASSWORD=test \
  -e POSTGRES_DB=langgraph_db \
  -d pgvector/pgvector:pg16
```

Start the FastAPI server:

```bash
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000 --reload-dir ./app
```

### Frontend Setup

Open another terminal:

```bash
cd frontend
```

Create the virtual environment:

```bash
python -m venv .venv
```

Activate it and install dependencies:

```bash
pip install -r requirements.txt
```

Launch Streamlit:

```bash
streamlit run gui/main.py
```

## 🔧 Environment Configuration

Create a `.env` file in the project root.

### LLM Configuration

```env
OPENAI_API_KEY=your_api_key
MODEL_PROVIDER=openai
MODEL_NAMES=["gpt-4o","gpt-4o-mini"]
MODEL_BASE_URL=
```

### Embedding Configuration

```env
EMBEDDINGS_MODEL_NAME=text-embedding-3-large
EMBEDDINGS_BASE_URL=
```

### Web Search

```env
TAVILY_API_KEY=your_tavily_api_key
```

### Authentication

```env
TOKEN_BEARER_URL=/api/v1/auth/login
JWT_SECRET=your_secure_secret
JWT_ALGORITHM=HS256
ACCESS_TOKEN_EXPIRY_MINS=1440
REFRESH_TOKEN_EXPIRY_DAYS=1
```

### PostgreSQL

```env
POSTGRES_HOST=127.0.0.1
POSTGRES_PORT=5432
POSTGRES_USER=postgres
POSTGRES_PASSWORD=test
POSTGRES_DATABASE=langgraph_db
PGVECTOR_COLLECTION_NAME=my_collection
```

### Frontend

```env
BACKEND_BASE_URL=http://127.0.0.1:8000/api/v1
```

> Refer to `env.example` for the complete configuration.

## 🧩 API Endpoints

The API uses the following base path:

```text
/api/v1
```

### Authentication

| Method | Endpoint              | Description                 |
| ------ | --------------------- | --------------------------- |
| POST   | `/auth/signup`        | Create a new account        |
| POST   | `/auth/login`         | Authenticate a user         |
| GET    | `/auth/logout`        | Logout                      |
| GET    | `/auth/refresh-token` | Generate a new access token |

### User Management

| Method | Endpoint                        | Description                  |
| ------ | ------------------------------- | ---------------------------- |
| GET    | `/users/me`                     | Get current user information |
| PUT    | `/users/user-profile/{user_id}` | Update user profile          |
| DELETE | `/users/user-profile/{user_id}` | Delete user profile          |

### Threads

| Method | Endpoint               | Description                       |
| ------ | ---------------------- | --------------------------------- |
| POST   | `/threads/`            | Create a conversation thread      |
| GET    | `/threads/`            | List user threads                 |
| GET    | `/threads/{thread_id}` | Retrieve a specific thread        |
| PATCH  | `/threads/{thread_id}` | Update thread title               |
| DELETE | `/threads/{thread_id}` | Delete thread and associated data |

### Documents

| Method | Endpoint                        | Description                 |
| ------ | ------------------------------- | --------------------------- |
| GET    | `/documents/{thread_id}`        | List documents              |
| POST   | `/documents/upload/{thread_id}` | Upload and index a document |
| DELETE | `/documents/{document_id}`      | Delete a document           |

### Chat

| Method | Endpoint            | Description                   |
| ------ | ------------------- | ----------------------------- |
| POST   | `/chat/`            | Public streaming chat         |
| POST   | `/chat/{thread_id}` | Authenticated agent chat      |
| GET    | `/chat/{thread_id}` | Retrieve conversation history |

## 📡 Streaming Response Format

The chat endpoints support **newline-delimited JSON (NDJSON)** streaming.

The backend can send different event types depending on what the agent is doing.

### Event Types

* `llm_chunk` – Partial LLM response
* `tool_call` – Indicates that the agent has selected a tool
* `tool_result` – Contains the result returned by the selected tool

Example:

```json
{"type":"tool_call","name":"retrieve_user_documents","args":{"query":"policy overview"}}

{"type":"tool_result","name":"retrieve_user_documents","content":"Retrieved document content..."}

{"type":"llm_chunk","content":"Based on the retrieved information..."}
```

This allows the frontend to display the assistant's response while the agent is still processing the request.

## 🔄 System Architecture

The application is divided into four primary stages.

### 1. Document Ingestion

Users can upload:

* PDF
* DOCX
* TXT

The documents are processed using document loaders and divided into smaller chunks with:

```text
RecursiveCharacterTextSplitter
```

The generated chunks are embedded and stored in PostgreSQL using pgvector.

### 2. Retrieval

When the user asks a question related to their uploaded documents, the retrieval tool performs semantic similarity search.

The search can be restricted using metadata such as:

```text
user_id
thread_id
```

This ensures that users only retrieve content belonging to their own conversations and documents.

The main retrieval tool is:

```text
retrieve_user_documents
```

### 3. Agent Reasoning

LangGraph manages the agent workflow.

The agent can determine whether it should:

1. Answer directly using the LLM.
2. Search the user's uploaded documents.
3. Perform a web search using Tavily.
4. Combine retrieved information before generating the final response.

The overall workflow follows a ReAct-style agent architecture.

### 4. Persistent Memory

Conversation state is stored using the **LangGraph PostgreSQL checkpointer**.

Each conversation is associated with a unique thread, allowing the agent to maintain context across multiple messages.

When a thread is removed, its associated conversation state and document-related data can also be cleaned up.

## 🧠 Agent Workflow

A simplified flow of the application is:

```text
                    ┌───────────────┐
                    │     User      │
                    └───────┬───────┘
                            │
                            ▼
                    ┌───────────────┐
                    │   Streamlit   │
                    │      UI       │
                    └───────┬───────┘
                            │
                            ▼
                    ┌───────────────┐
                    │   FastAPI     │
                    │     API       │
                    └───────┬───────┘
                            │
                            ▼
                    ┌───────────────┐
                    │   LangGraph   │
                    │     Agent     │
                    └───────┬───────┘
                            │
                 ┌──────────┴──────────┐
                 ▼                     ▼
        ┌─────────────────┐   ┌─────────────────┐
        │ Document Search │   │   Web Search    │
        │    pgvector     │   │     Tavily      │
        └────────┬────────┘   └────────┬────────┘
                 │                     │
                 └──────────┬──────────┘
                            ▼
                    ┌───────────────┐
                    │      LLM      │
                    └───────┬───────┘
                            │
                            ▼
                    ┌───────────────┐
                    │ Streaming     │
                    │   Response    │
                    └───────────────┘
```

## 🔐 Security & Data Isolation

The application uses JWT authentication to identify users and protect private resources.

User-specific filtering is applied to:

* Conversation threads
* Uploaded documents
* Vector embeddings
* Chat history

This prevents one user's documents and conversations from being exposed to another user.

JWT configuration is controlled through environment variables, allowing secrets and token lifetimes to be changed without modifying application code.

## 📚 Document Processing Pipeline

The document workflow can be summarized as:

```text
Document Upload
      │
      ▼
Document Loader
      │
      ▼
Text Extraction
      │
      ▼
Text Chunking
      │
      ▼
Embedding Generation
      │
      ▼
PostgreSQL + pgvector
      │
      ▼
Semantic Retrieval
      │
      ▼
LangGraph Agent
      │
      ▼
LLM Response
```

## 📖 API Documentation

Once the backend is running, interactive API documentation is available through Swagger:

```text
http://localhost:8000/api/v1/docs
```

ReDoc is also available at:

```text
http://localhost:8000/api/v1/redoc
```

These interfaces can be used to inspect available endpoints and test API requests.

## 🖼️ Screenshots

### Unauthenticated Home Page

![Home Page](./screenshots/home.png)

### Authenticated Home Page

![Authenticated Home Page](./screenshots/home-authenticated.png)

## 📁 Project Structure

A typical project structure is:

```text
langgraph-rag-agent/
│
├── backend/
│   ├── app/
│   │   ├── api/
│   │   ├── agents/
│   │   ├── core/
│   │   ├── models/
│   │   ├── services/
│   │   ├── tools/
│   │   └── main.py
│   │
│   └── requirements.txt
│
├── frontend/
│   ├── gui/
│   │   └── main.py
│   └── requirements.txt
│
├── screenshots/
│   ├── home.png
│   └── home-authenticated.png
│
├── docker-compose.yml
├── env.example
├── .gitignore
└── README.md
```

