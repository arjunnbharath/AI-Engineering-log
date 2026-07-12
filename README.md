# Retrieval-Augmented Generation (RAG)

> A learning journal documenting my journey understanding and building RAG systems.

---

## What is RAG?

**Retrieval-Augmented Generation (RAG)** is a technique that gives a language model access to **external knowledge** at question time, instead of relying only on what it learned during training.

### The Problem It Solves

Large Language Models (LLMs) have two major limitations:

1. **Knowledge cutoff** — they don't know about recent events or your private data.
2. **Hallucination** — they can sound confident while making things up.

RAG fixes this by: **retrieve relevant documents → feed them to the model → generate an answer grounded in those documents.**

```
User Question
     │
     ▼
┌─────────────┐     ┌──────────────┐     ┌─────────────┐
│  Embed the  │────▶│  Search for  │────▶│  LLM reads  │
│   question  │     │  similar docs│     │  + answers  │
└─────────────┘     └──────────────┘     └─────────────┘
```

---

## The 5 Stages of RAG

### 1. Ingestion (Offline — One-Time Setup)

Take your source material (PDFs, docs, wiki pages, code, etc.) and prepare it for search.

```
Raw Documents → Clean Text → Split into Chunks → Embed → Store in Vector DB
```

### 2. Chunking

Documents are split into smaller pieces ("chunks"), typically 200–1000 tokens with some overlap.

**Why chunk?**
- Embeddings work better on focused passages
- You retrieve only what's relevant, not entire books
- Models have context window limits

```
"The company was founded in 2019. Revenue grew 40% in 2023..."
                    ↓ chunk
["The company was founded in 2019.", "Revenue grew 40% in 2023..."]
```

### 3. Embedding

Each chunk is converted into a **vector** (a list of numbers) that captures semantic meaning.

Similar meaning → vectors are close together in vector space.

```
"What's our refund policy?"  →  [0.12, -0.45, 0.88, ...]
"How do I get my money back?" →  [0.11, -0.43, 0.90, ...]  ← very close!
```

### 4. Retrieval (At Query Time)

When a user asks a question:

1. Embed the question
2. Find the **k** most similar chunks (cosine similarity)
3. Optionally re-rank results for better precision

This is the **Retrieval** in RAG.

### 5. Augmented Generation

Retrieved chunks are inserted into the prompt:

```
You are a helpful assistant. Answer using ONLY the context below.

Context:
[chunk 1]
[chunk 2]
[chunk 3]

Question: What is our refund policy?

Answer:
```

The LLM generates an answer **grounded in that context**. This is the **Augmented Generation**.

---

## Mental Model

Think of RAG as an **open-book exam** for an LLM:

| Without RAG | With RAG |
|-------------|----------|
| Closed-book: model relies on memory | Open-book: model reads your docs first |
| Can't access your private data | Can answer from your PDFs, DB, wiki |
| May hallucinate facts | Much more likely to cite real content |

---

## Key Components

| Component | Role | Examples |
|-----------|------|----------|
| **Document Loader** | Read files | PyPDF, Unstructured, web scrapers |
| **Chunker / Splitter** | Split text into pieces | RecursiveCharacterTextSplitter |
| **Embedding Model** | Convert text → vectors | OpenAI `text-embedding-3-small`, `sentence-transformers` |
| **Vector Store** | Store & search vectors | Chroma, Pinecone, FAISS, pgvector |
| **LLM** | Generate the answer | GPT-4, Claude, Llama, etc. |
| **Orchestrator** | Wire everything together | LangChain, LlamaIndex, custom code |

---

## Minimal Code Example (Conceptual)

```python
# OFFLINE: Build the index
chunks = split_documents(load_pdfs("company_docs/"))
vectors = embed(chunks)
vector_db.save(chunks, vectors)

# ONLINE: Answer a question
query = "What is our refund policy?"
relevant_chunks = vector_db.search(embed(query), top_k=3)

prompt = f"""
Context:
{relevant_chunks}

Question: {query}
Answer based only on the context above.
"""

answer = llm.generate(prompt)
```

---

## RAG vs Alternatives

| Approach | When to Use |
|----------|-------------|
| **RAG** | Dynamic/private knowledge, need citations, data changes often |
| **Fine-tuning** | Change model behavior/style, not add new facts |
| **Long Context** | Small corpus that fits in one prompt (expensive, slower) |
| **Tool Use / Agents** | Multi-step reasoning, APIs, actions beyond Q&A |

RAG is often the **first choice** for "chat with my documents" applications.

---

## What Makes RAG Good or Bad

### Chunking
- Too small → loses context
- Too large → retrieves irrelevant noise

### Retrieval Quality
- Bad retrieval = good LLM still gives wrong answers ("garbage in, garbage out")
- Try **hybrid search**: semantic + keyword (BM25)

### Prompt Design
- Tell the model to say "I don't know" if context doesn't contain the answer
- Include source metadata for citations

### Evaluation
- Measure: Did we retrieve the right chunks? Is the answer faithful to them?
- Tools: RAGAS, custom test sets with known Q&A pairs

---

## Production Architecture

```
                    ┌─────────────────┐
                    │  User Interface │
                    └────────┬────────┘
                             │
                    ┌────────▼────────┐
                    │   API / Backend │
                    └────────┬────────┘
              ┌──────────────┼──────────────┐
              │              │              │
     ┌────────▼────┐  ┌──────▼──────┐  ┌───▼───┐
     │ Vector DB   │  │  Embedding  │  │  LLM  │
     │ (Chroma...) │  │   Model     │  │ (API) │
     └─────────────┘  └─────────────┘  └───────┘
```

### Advanced Techniques
- **Re-ranking** — cross-encoder model after initial retrieval
- **Query rewriting** — rephrase vague questions before search
- **Metadata filters** — e.g., only search `department=HR`
- **Agentic RAG** — retrieve → evaluate → retrieve again if needed

---

## Learning Roadmap

- [ ] **Hello RAG** — one PDF, Chroma + embeddings, ask questions in a script
- [ ] **Add citations** — return which chunk each answer came from
- [ ] **Improve retrieval** — better chunking, hybrid search, re-ranking
- [ ] **Evaluate** — 10 test questions with expected answers
- [ ] **UI** — Streamlit or FastAPI chat interface

---

## One-Sentence Summary

> **RAG = search your knowledge base for relevant passages, then let the LLM answer using only what it found.**

---

## Resources

- [Original RAG Paper (Lewis et al., 2020)](https://arxiv.org/abs/2005.11401)
- [LangChain RAG Tutorial](https://python.langchain.com/docs/tutorials/rag/)
- [LlamaIndex Documentation](https://docs.llamaindex.ai/)
- [Chroma Vector DB](https://www.trychroma.com/)

---

## Notes

_This repo is a living document. I'll update it as I build projects and learn new concepts._
