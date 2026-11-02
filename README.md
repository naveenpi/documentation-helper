# National Parks Guide

A small retrieval-augmented generation (RAG) app for asking questions about U.S. national parks. It crawls public National Park Service (NPS) pages, stores searchable text in Pinecone, and uses a Streamlit chat interface to answer questions with source links.

This project is adapted from the [LangChain Documentation Helper](https://github.com/emarco177/documentation-helper) example. The data source and prompts here are for national parks, and the project uses `uv` for Python dependencies.

## What it does

1. `ingestion.py` crawls pages starting at `https://www.nps.gov/index.htm` with Tavily.
2. Each page is split into text chunks. OpenAI's `text-embedding-3-small` turns the chunks into vectors, which are stored in Pinecone with their source URLs.
3. When you ask a question, `backend/core.py` retrieves relevant chunks and sends them to an OpenAI chat model to draft an answer.
4. `main.py` displays the answer and the retrieved source URLs in Streamlit.

The app can help explore park information, but its index is a snapshot from the most recent ingestion run. For closures, alerts, fees, and hours, check the linked NPS page before a visit.

## Quick start

### Prerequisites

- `uv` and Python 3.11 or newer. This repository currently pins Python 3.11 in `.python-version`.
- An OpenAI API key for embeddings and answers.
- A Tavily API key for crawling NPS pages.
- A Pinecone API key and an existing dense-vector index compatible with the embedding model.

### 1. Install dependencies

From the project directory:

```bash
uv sync
```

### 2. Configure credentials

Create a `.env` file in the project directory:

```dotenv
OPENAI_API_KEY=your_openai_api_key
PINECONE_API_KEY=your_pinecone_api_key
TAVILY_API_KEY=your_tavily_api_key
INDEX_NAME=your_pinecone_index_name
```

The Pinecone index must exist before ingestion. `INDEX_NAME` is used by both the ingestion script and the backend. Keep `.env` private; the repository's `.gitignore` excludes it.

### 3. Ingest NPS pages

```bash
uv run python ingestion.py
```

This crawls NPS pages and writes their chunks to Pinecone. It calls external APIs and can incur usage charges. Run it when you first set up the app or when you intentionally want to refresh the index. Re-running it currently adds new records rather than replacing prior records, so repeated runs can create duplicates.

### 4. Start the chat app

```bash
uv run streamlit run main.py
```

Open the local URL printed by Streamlit, usually `http://localhost:8501`. Press **Ctrl+C** in the terminal to stop the server.

## Using the app

Ask a question such as:

> What should I consider before visiting a national park in winter?

The response includes an expandable **Sources** section with URLs from the retrieved NPS pages. **Clear chat** resets the displayed conversation. Each question is currently sent to the backend on its own; earlier messages are displayed but are not passed to the model as conversation history.

## Project structure

```text
documentation-helper/
├── backend/
│   ├── __init__.py
│   └── core.py          # Retrieval tool and answer generation
├── ingestion.py         # Crawl, split, embed, and index NPS content
├── logger.py            # Ingestion log formatting
├── main.py              # Streamlit chat interface
├── pyproject.toml       # Python dependencies
└── .python-version      # Project Python version
```

## Current scope

- The crawl starts at the NPS home page with a maximum depth of five. It may index general site pages as well as park-specific pages.
- Retrieved text can contain navigation, image markup, or other page boilerplate from the crawl.
- The app retrieves from the stored index; it does not fetch live NPS updates for each question.
- Source URLs identify retrieved pages, but the app does not yet provide passage-level citations or a timestamp for each indexed page.

## Reference

The structure and RAG workflow were inspired by [emarco177/documentation-helper](https://github.com/emarco177/documentation-helper). NPS content comes from the [National Park Service](https://www.nps.gov/).
