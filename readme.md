# MultiPDF Chat App (RAG-Based LLM)

A Streamlit application that lets you upload multiple PDFs and ask questions grounded in their contents using Retrieval-Augmented Generation (RAG).

This project is intentionally small and interview-friendly: it demonstrates an end-to-end RAG pipeline using `LangChain`, `FAISS`, and `OpenAI`.

## What This Project Does

- Ingests multiple PDF files through a Streamlit UI.
- Extracts text from each page.
- Splits text into overlapping chunks.
- Creates embeddings for each chunk.
- Stores embeddings in a FAISS vector index.
- Retrieves relevant chunks for each user query.
- Generates contextual answers with an LLM while keeping chat history.

## System Overview

The app is implemented in `app.py` and follows this sequence:

1. **Document ingestion**  
   Uploaded PDFs are read using `PyPDF2.PdfReader`.
2. **Chunking**  
   Text is split with `CharacterTextSplitter` using:
   - `chunk_size=1000`
   - `chunk_overlap=200`
3. **Embedding + indexing**  
   Chunks are embedded with `OpenAIEmbeddings` and indexed using `FAISS`.
4. **Conversational retrieval**  
   `ConversationalRetrievalChain` combines:
   - `ChatOpenAI` for answer generation
   - FAISS retriever for context selection
   - `ConversationBufferMemory` for chat continuity
5. **Response rendering**  
   Chat history is rendered in Streamlit using HTML templates from `htmlTemplates.py`.

## Architecture Notes (Interview View)

- **RAG pattern**: retrieval narrows context, improving factual grounding vs pure generation.
- **Vector store**: FAISS gives fast local similarity search.
- **Memory strategy**: conversation buffer preserves prior turns, helpful for follow-up questions.
- **Tradeoff**: this implementation optimizes for simplicity and learning, not production scale.

## Repository Structure

- `app.py` - main app logic (ingestion, chunking, vector store, chain, UI flow)
- `htmlTemplates.py` - CSS and HTML templates for chat bubbles
- `requirements.txt` - Python dependencies
- `.env.example` - required environment variable names
- `docs/INTERVIEW_GUIDE.md` - deep interview prep guide for this project

## Installation

### 1) Clone and enter the project

```bash
git clone <your-repo-url>
cd rag-based-llm
```

### 2) Create and activate a virtual environment

```bash
python -m venv .venv
```

Windows (PowerShell):

```bash
.venv\Scripts\Activate.ps1
```

macOS/Linux:

```bash
source .venv/bin/activate
```

### 3) Install dependencies

```bash
pip install -r requirements.txt
```

### 4) Configure environment variables

Create a `.env` file (or copy from `.env.example`) and set:

```bash
OPENAI_API_KEY=your_openai_api_key
HUGGINGFACEHUB_API_TOKEN=optional_if_using_huggingface
```

## Running the App

```bash
streamlit run app.py
```

Then in the browser:

1. Upload one or more PDFs in the sidebar.
2. Click **Process**.
3. Ask questions in the text input.

## Hosting

This project is deployment-ready.

- Fastest option: Streamlit Community Cloud
- Alternative: Docker-based deployment on Render/Railway

Step-by-step instructions are in:

- `docs/HOSTING.md`

## Key Dependencies

- `streamlit` - web UI
- `PyPDF2` - PDF text extraction
- `langchain` - chain orchestration
- `openai` - OpenAI model API
- `faiss-cpu` - vector similarity search
- `python-dotenv` - environment variable loading
- `tiktoken` - token utilities (used by LangChain/OpenAI stack)

## Limitations

- No persistent storage for embeddings (recomputed per session).
- No source citations in responses.
- Minimal error handling for empty/bad PDFs.
- Uses older LangChain imports/API style pinned in `requirements.txt`.
- No authentication, rate limiting, or monitoring.

## Suggested Improvements

- Add per-chunk metadata and return citations (file/page).
- Persist vector index on disk for faster reloads.
- Replace `CharacterTextSplitter` with token-aware splitter.
- Add better prompt engineering and answer constraints.
- Add tests (unit tests for extraction/chunking and integration tests for QA flow).
- Upgrade dependencies and migrate to current LangChain package layout.

## Interview Preparation

For a full interview-oriented walkthrough (design choices, tradeoffs, probable questions, and strong answer templates), see:

- `docs/INTERVIEW_GUIDE.md`

## Credits

- Original concept/tutorial inspiration: [YouTube - Build a Multi PDF Chat App with LangChain](https://youtu.be/dXxQ0LR-3Hg)

## License

This project is released under the [MIT License](https://opensource.org/licenses/MIT).
