# RAG Production App

A Retrieval-Augmented Generation (RAG) application that lets you upload PDF documents, index their text in Qdrant, and ask questions about their contents. The app uses OpenAI embeddings and an OpenAI chat model, with Inngest managing the ingestion and query workflows.

## Features

- Upload PDF files through a Streamlit interface.
- Extract PDF text and split it into overlapping chunks.
- Generate embeddings with OpenAI's `text-embedding-3-large` model.
- Store and retrieve document chunks using Qdrant vector search.
- Process ingestion and question-answering workflows with Inngest.
- Ask questions and view answers with their source document names.

## Tech Stack

- **Python** — application code
- **FastAPI** — Inngest function endpoint
- **Inngest** — event-driven workflow orchestration and local development server
- **Streamlit** — user interface
- **Qdrant** — vector database
- **OpenAI API** — embeddings and answer generation
- **LlamaIndex** — PDF reading and text chunking
- **uv** — Python dependency and environment management

## Prerequisites

Install the following before running the project:

- Python version compatible with `pyproject.toml` (Python 3.13 or newer)
- [`uv`](https://docs.astral.sh/uv/)
- [Docker Desktop](https://www.docker.com/products/docker-desktop/) running on your Mac
- Node.js and `npx`
- An OpenAI API key with API billing/credits enabled

## Configuration

Create a `.env` file in the project root:

```env
OPENAI_API_KEY=your_openai_api_key
```

The application loads this file with `python-dotenv`. Keep your API key private and do not commit `.env` to version control.

## Setup

Clone the repository and change into the project directory:

```bash
git clone <repository-url>
cd RagProductionApp
```

Install Python dependencies:

```bash
uv sync
```

## Run Locally (macOS)

Start each service in a separate terminal, from the project root.

### 1. Start Qdrant

Start Docker Desktop, then run:

```bash
docker run -d \
  --name drantRagDb \
  -p 6333:6333 \
  -v "$(pwd)/qdrant_storage:/qdrant/storage" \
  qdrant/qdrant
```

Qdrant's local API is available at `http://localhost:6333`. The dashboard is available at `http://localhost:6333/dashboard`.

The `qdrant_storage` directory persists the database data between container restarts. If the container already exists, start it with:

```bash
docker start drantRagDb
```

### 2. Start the FastAPI application

In a new terminal:

```bash
uv run uvicorn main:app --reload
```

The FastAPI app serves the Inngest endpoint at:

```text
http://127.0.0.1:8000/api/inngest
```

### 3. Start the Inngest Dev Server

In another terminal:

```bash
npx inngest-cli@latest dev -u http://127.0.0.1:8000/api/inngest --no-discovery
```

The Inngest Dev Server UI is typically available at `http://localhost:8288`.

### 4. Start the Streamlit UI

In another terminal:

```bash
uv run streamlit run streamlit_app.py
```

Open the local URL printed by Streamlit (usually `http://localhost:8501`).

## How It Works

### PDF ingestion

1. Upload a PDF in the Streamlit UI.
2. The UI saves the PDF under `uploads/` and sends an Inngest event named `rag/ingest_pdf`.
3. Inngest invokes the FastAPI function.
4. LlamaIndex extracts PDF text and splits it into chunks.
5. OpenAI generates embeddings for the chunks.
6. The application upserts vectors and payloads (`source` and `text`) into the Qdrant `docs` collection.

### Question answering

1. Enter a question and choose how many chunks to retrieve.
2. The UI sends an Inngest event named `rag/query_pdf_ai`.
3. The application embeds the question and searches Qdrant for similar chunks.
4. The retrieved text is added to the prompt context.
5. OpenAI's `gpt-4o-mini` generates an answer based on the supplied context.
6. The UI displays the answer and source document names.

## Useful URLs

| Service | Local URL |
|---|---|
| FastAPI / Inngest endpoint | `http://127.0.0.1:8000/api/inngest` |
| Inngest Dev Server UI | `http://localhost:8288` |
| Qdrant API | `http://localhost:6333` |
| Qdrant Dashboard | `http://localhost:6333/dashboard` |
| Streamlit UI | `http://localhost:8501` |

## Notes

- Run the commands from the project root unless stated otherwise.
- Keep Qdrant, FastAPI, Inngest, and Streamlit running while using the application.
- The PDF upload flow currently stores uploaded files in the local `uploads/` directory.
