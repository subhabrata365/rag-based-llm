# RAG-Based MultiPDF Chat App

This guide is designed to help you explain this project confidently in interviews.

## 1) One-Minute Project Pitch (Simple Version)

"The goal of this project is to let users ask questions about their own PDF files and get answers based only on those files. I built a RAG pipeline: extract text from PDFs, split into chunks, embed chunks, store in FAISS, retrieve relevant chunks for each question, then use an LLM to generate the final answer with chat memory."

## 2) Problem Statement

Large language models are powerful but can hallucinate when asked about private or domain-specific documents. The project addresses that by injecting relevant document context at inference time instead of relying on model pretraining alone.

## 3) End-to-End Flow

1. User uploads PDFs in Streamlit sidebar.
2. `get_pdf_text()` extracts page text using `PyPDF2`.
3. `get_text_chunks()` splits text into overlapping chunks (`1000/200`).
4. `get_vectorstore()` embeds chunks with `OpenAIEmbeddings` and creates a FAISS index.
5. `get_conversation_chain()` builds a `ConversationalRetrievalChain` using:
   - `ChatOpenAI`
   - FAISS retriever (`vectorstore.as_retriever()`)
   - `ConversationBufferMemory`
6. `handle_userinput()` sends user question to the chain and renders alternating user/bot messages.

## 4) Languages, Frameworks, and Microservices

- **Primary language:** Python
- **Frontend/UI framework:** Streamlit
- **LLM/RAG framework:** LangChain
- **Vector database/index:** FAISS (local, in-memory)
- **Document parsing library:** PyPDF2
- **Model providers used:** OpenAI (`ChatOpenAI`, `OpenAIEmbeddings`)
- **Environment management:** python-dotenv

### Microservices Note

This project is **not** implemented as microservices.  
It is a **single monolithic Streamlit app** running in one process (`app.py`).

If interviewers ask about microservices, you can say:
- "Current version is monolithic for simplicity and learning speed."
- "A production version could split into services: ingestion service, embedding/index service, and query/chat API service."

## 5) Core Components and Responsibilities

- `get_pdf_text(pdf_docs)`  
  Collects raw text from all uploaded PDFs.

- `get_text_chunks(text)`  
  Breaks long text into retrieval-friendly chunks.

- `get_vectorstore(text_chunks)`  
  Converts text chunks into vectors and builds searchable index.

- `get_conversation_chain(vectorstore)`  
  Connects retrieval + LLM + memory into one queryable object.

- `handle_userinput(user_question)`  
  Executes QA turn and updates UI chat history.

## 6) Why These Design Choices

- **FAISS**: lightweight, fast local similarity search; great for demos/prototypes.
- **Character chunking with overlap**: simple and effective baseline; overlap keeps context continuity.
- **ConversationBufferMemory**: supports follow-up questions without repeating prior context manually.
- **Streamlit**: minimal code to ship an interactive data/AI app.

## 7) Tradeoffs and Limitations

- **Pros**
  - Easy to understand and explain.
  - Demonstrates complete RAG loop.
  - Fast local experimentation.

- **Cons**
  - No persistence for vector index (re-index each run).
  - No citation display (harder to verify answer provenance).
  - No robust handling for scanned PDFs/OCR failures.
  - Uses older pinned LangChain stack.
  - No guardrails for prompt injection in source docs.

## 8) Production Hardening Plan

If asked "how would you productionize this?", discuss:

1. Add document metadata (`file_name`, `page_number`) and return citations.
2. Persist embeddings/index (disk, database, or managed vector DB).
3. Use token-aware chunking and retrieval tuning (`k`, MMR, reranking).
4. Add eval set and retrieval metrics (precision@k, answer faithfulness).
5. Implement observability (latency, token usage, failure tracking).
6. Add security controls (file validation, content filtering, auth).
7. Improve UX (progress indicators, source snippets, session management).

## 9) Common Interview Questions + Clear Answers

### Q1: Why use RAG instead of fine-tuning?

**Clear answer:**  
RAG is better for document QA because documents change often.  
You can update the index quickly without retraining a model.

**Interview answer (30 sec):**  
"RAG is faster, cheaper, and easier to maintain for document question-answering. I can add new PDFs and re-index, instead of fine-tuning every time the data changes. It also improves factual grounding by forcing the model to answer from retrieved context."

### Q2: Why chunk size 1000 and overlap 200?

**Clear answer:**  
This is a balanced default.  
1000 keeps enough context; 200 overlap prevents sentence/context breaks between chunks.

**Interview answer (30 sec):**  
"I used 1000/200 as a baseline tradeoff between retrieval quality and token cost. If chunks are too small, context is lost; too large, retrieval gets noisy and expensive. Overlap helps preserve meaning across chunk boundaries."

### Q3: How does memory help here?

**Clear answer:**  
Memory stores previous messages so follow-up questions make sense.

**Interview answer (20 sec):**  
"`ConversationBufferMemory` keeps chat history, so users can ask follow-up questions like 'explain that section again' and the app still understands the context."

### Q4: What causes hallucinations even in RAG?

**Clear answer:**  
If retrieval returns wrong/weak chunks, the model can still guess.

**Interview answer (30 sec):**  
"RAG reduces hallucination but does not remove it. Hallucinations still happen when retrieval quality is poor, chunks are noisy, or prompts are weak. I would improve this with better chunking, reranking, stricter prompts, and source citations."

### Q5: Biggest bottleneck in this app?

**Clear answer:**  
Rebuilding embeddings/index on every upload is slow.

**Interview answer (20 sec):**  
"The biggest bottleneck is indexing latency. The app recreates embeddings and FAISS index during processing, and because it is not persisted, this work repeats every session."

### Q6: What would you test?

**Clear answer:**  
Test extraction, chunking, retrieval quality, and error handling.

**Interview answer (checklist):**
- PDF extraction correctness for different PDFs
- chunking behavior (size/overlap boundaries)
- retrieval relevance for known test questions
- end-to-end question answering flow
- failure paths (empty file, bad file, missing API key, API timeout)

## 10) Failure Modes You Should Mention

- PDF has scanned images only (no text layer), so extraction returns little/no text.
- Very short or noisy chunks reduce retrieval precision.
- API failures/timeouts from embedding or chat model providers.
- User asks out-of-scope question not covered in uploaded documents.

## 11) Quick Resume Bullet (Copy-Friendly)

"Built a Streamlit RAG application for multi-document PDF QA using LangChain, OpenAI embeddings, FAISS vector retrieval, and conversational memory; implemented ingestion, chunking, semantic search, and context-grounded response generation."

## 12) 30-60-Second Architecture Explanation

"The app has two phases: indexing and querying. In indexing, PDFs are parsed, chunked, embedded, and stored in a FAISS index. In querying, the user question is embedded, top similar chunks are retrieved, and a chat model generates an answer using retrieved context plus conversation history. This architecture improves relevance and factuality compared to plain LLM prompting."

## 13) Stretch Improvements (If You Want to Impress)

- Add hybrid retrieval (BM25 + dense vectors).
- Add reranker model before final context selection.
- Add structured output format with confidence and source links.
- Support OCR pipeline for scanned PDFs.
- Add multi-tenant architecture and persisted sessions.
