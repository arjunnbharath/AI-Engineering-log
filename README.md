# RAG for Beginners

> **RAG** = **R**etrieval-**A**ugmented **G**eneration

A simple, beginner-friendly guide to understanding RAG. No prior AI experience needed — just basic curiosity!

---

## Table of Contents

1. [What is RAG? (Simple Explanation)](#what-is-rag-simple-explanation)
2. [Real-World Analogy](#real-world-analogy)
3. [Key Terms (Glossary)](#key-terms-glossary)
4. [How RAG Works — Step by Step](#how-rag-works--step-by-step)
5. [Chunking vs Embedding — What's the Difference?](#chunking-vs-embedding--whats-the-difference)
6. [The Full Picture](#the-full-picture)
7. [Simple Code Example](#simple-code-example)
8. [Tools You'll Use](#tools-youll-use)
9. [Common Beginner Questions](#common-beginner-questions)
10. [Learning Roadmap](#learning-roadmap)
11. [Resources](#resources)

---

## What is RAG? (Simple Explanation)

Imagine you have a **smart assistant** (like ChatGPT) and a pile of **your own documents** (PDFs, notes, company files).

**The problem:** ChatGPT doesn't know what's inside *your* documents. It only knows what it was trained on.

**The solution — RAG:** Before answering your question, the system:
1. **Searches** your documents for relevant information
2. **Reads** that information
3. **Answers** your question using what it found

That's it. RAG = **Search first, then answer.**

---

## Real-World Analogy

### 🎓 Open-Book Exam

| Without RAG | With RAG |
|-------------|----------|
| Student takes exam from memory only | Student can look up answers in a textbook |
| Might guess wrong answers | Answers come from the actual book |
| Can't use your private notes | Can use *your* documents |

**RAG turns the AI from a student who memorized everything into a student who can open your textbook and find the right page.**

### 📚 Library Analogy

1. You ask: *"What is our refund policy?"*
2. The librarian (retrieval) searches the shelves and finds 3 relevant pages
3. The assistant (LLM) reads those pages and writes you a clear answer

---

## Key Terms (Glossary)

Read this first if any word below confuses you.

| Term | Simple Meaning |
|------|----------------|
| **LLM** | Large Language Model — the AI that writes answers (e.g. ChatGPT, Claude) |
| **Document** | Any text source: PDF, Word file, webpage, notes |
| **Chunk** | A small piece of text cut from a bigger document (like one paragraph) |
| **Chunking** | The act of splitting a big document into smaller chunks |
| **Embedding** | Turning text into a list of numbers so a computer can compare meaning |
| **Vector** | Just another word for that list of numbers from embedding |
| **Vector Database** | A special storage that finds similar meaning fast (like Chroma, FAISS) |
| **Retrieval** | Finding the most relevant chunks for a user's question |
| **Prompt** | The instructions + context you send to the LLM |
| **Hallucination** | When the AI makes up facts that sound real but are wrong |

---

## How RAG Works — Step by Step

RAG has two phases: **Setup** (do once) and **Question time** (every time someone asks).

---

### Phase 1: Setup (Do This Once)

You prepare your documents so the system can search them later.

#### Step 1 — Load Documents

Read your files and get the text out.

```
📄 company_handbook.pdf  →  "Our refund policy is..."
📄 faq.docx              →  "Shipping takes 3-5 days..."
```

#### Step 2 — Chunking (Split into Pieces)

Big documents are cut into smaller pieces. **Output is still plain text.**

```
Full document:
"Our refund policy allows returns within 30 days.
 Shipping costs are non-refundable.
 Contact support@company.com for help."

        ↓ split into chunks

Chunk 1: "Our refund policy allows returns within 30 days."
Chunk 2: "Shipping costs are non-refundable."
Chunk 3: "Contact support@company.com for help."
```

**Why split?** So when someone asks about refunds, you find only the refund paragraph — not the whole 100-page handbook.

#### Step 3 — Embedding (Text → Numbers)

Each chunk is passed through an **embedding model**. It returns a **vector** (a list of numbers).

```
Chunk: "Our refund policy allows returns within 30 days."
                    ↓ embedding
Vector: [0.12, -0.45, 0.88, 0.03, 0.67, ...]
```

**Why numbers?** Computers compare numbers easily. Similar meaning → similar numbers.

Example:
```
"refund policy"     →  [0.12, -0.45, 0.88, ...]
"money back rules"  →  [0.11, -0.43, 0.90, ...]   ← very similar numbers!
```

#### Step 4 — Store in Vector Database

Save each chunk + its vector in a **vector database**. Think of it as a smart filing cabinet that finds similar meaning.

```
Vector DB:
  Chunk 1 + [0.12, -0.45, ...]
  Chunk 2 + [0.34, 0.21, ...]
  Chunk 3 + [0.55, -0.10, ...]
```

✅ Setup done! Your documents are now searchable.

---

### Phase 2: Question Time (Every Time a User Asks)

#### Step 5 — User Asks a Question

```
User: "Can I return my order after 2 weeks?"
```

#### Step 6 — Embed the Question

Turn the question into numbers (same way you embedded the chunks).

```
"Can I return my order after 2 weeks?"
        ↓ embedding
[0.13, -0.44, 0.87, ...]
```

#### Step 7 — Retrieval (Search)

Compare the question's vector to all stored chunk vectors. Pick the **most similar** chunks (usually top 3–5).

```
Found:
  ✅ Chunk 1: "Our refund policy allows returns within 30 days."
  ✅ Chunk 3: "Contact support@company.com for help."
```

This is the **R** in RAG — **Retrieval**.

#### Step 8 — Augmented Generation (Answer)

Put the found chunks into a prompt and send it to the LLM.

```
You are a helpful assistant. Answer using ONLY the context below.
If the answer is not in the context, say "I don't know."

Context:
- Our refund policy allows returns within 30 days.
- Contact support@company.com for help.

Question: Can I return my order after 2 weeks?

Answer:
```

The LLM reads the context and writes an answer:

```
Yes! You can return your order within 30 days. 
2 weeks is within that window. 
For help, contact support@company.com.
```

This is the **AG** in RAG — **Augmented Generation**.

---

## Chunking vs Embedding — What's the Difference?

**They are NOT the same thing.** Beginners often mix these up.

| | Chunking | Embedding |
|---|----------|-----------|
| **What happens** | Big text → small text pieces | Text → list of numbers |
| **Input** | Full document | One chunk (or one question) |
| **Output** | `"Our refund policy..."` (readable text) | `[0.12, -0.45, 0.88, ...]` (numbers) |
| **Analogy** | Cutting a pizza into slices | Giving each slice a barcode |

```
📄 Full PDF
     │
     ▼  CHUNKING (still text)
["chunk 1", "chunk 2", "chunk 3"]
     │
     ▼  EMBEDDING (text → numbers)
[[0.1, 0.4, ...], [0.2, 0.1, ...], [0.5, -0.3, ...]]
     │
     ▼
💾 Saved in Vector Database
```

**Remember:** Chunk first → then embed each chunk separately.

---

## The Full Picture

```
═══════════════════════════════════════════════════
  PHASE 1: SETUP (once)
═══════════════════════════════════════════════════

  Your PDFs / Docs
        │
        ▼
   1. Load text
        │
        ▼
   2. Chunking (split into pieces)
        │
        ▼
   3. Embedding (text → numbers)
        │
        ▼
   4. Store in Vector DB


═══════════════════════════════════════════════════
  PHASE 2: ANSWER QUESTIONS (every time)
═══════════════════════════════════════════════════

  User asks a question
        │
        ▼
   5. Embed the question
        │
        ▼
   6. Search Vector DB (find similar chunks)
        │
        ▼
   7. Send chunks + question to LLM
        │
        ▼
   8. LLM writes the answer
```

---

## Simple Code Example

Don't worry if you don't understand every line yet. Read the comments.

```python
# ─── PHASE 1: SETUP (run once) ───────────────────

# 1. Load your documents
documents = load_pdfs("my_folder/")   # reads PDF files

# 2. Split into chunks
chunks = split_into_chunks(documents)   # still text

# 3. Convert each chunk to a vector (numbers)
vectors = embed(chunks)                 # text → numbers

# 4. Save in vector database
vector_db.save(chunks, vectors)


# ─── PHASE 2: ANSWER A QUESTION ──────────────────

# User's question
question = "What is the refund policy?"

# 5. Turn question into numbers
question_vector = embed(question)

# 6. Find the 3 most similar chunks
relevant_chunks = vector_db.search(question_vector, top_k=3)

# 7. Build a prompt with context + question
prompt = f"""
Answer using ONLY this context:
{relevant_chunks}

Question: {question}
"""

# 8. LLM generates the answer
answer = llm.generate(prompt)
print(answer)
```

---

## Tools You'll Use

You don't need to build everything from scratch. These tools help:

| What You Need | Beginner-Friendly Tools |
|---------------|-------------------------|
| Read PDFs | `PyPDF`, `pypdf` |
| Split text into chunks | LangChain `RecursiveCharacterTextSplitter` |
| Create embeddings | OpenAI API, `sentence-transformers` (free, local) |
| Store & search vectors | **Chroma** (easiest to start), FAISS |
| Generate answers | OpenAI GPT, Claude, Ollama (free, local) |
| Tie it all together | LangChain, LlamaIndex |

**Recommended starter stack:** Python + Chroma + sentence-transformers + Ollama (all free, runs on your computer).

---

## Common Beginner Questions

### "Do I need to train my own AI model?"
**No.** You use a pre-trained LLM (like GPT or Llama) and a pre-trained embedding model. RAG doesn't require training.

### "Is chunking the same as embedding?"
**No.** Chunking = split text. Embedding = turn text into numbers. See [Chunking vs Embedding](#chunking-vs-embedding--whats-the-difference).

### "What is a vector?"
A **vector** is just a list of numbers, e.g. `[0.12, -0.45, 0.88]`. The embedding model creates it. Similar meanings get similar numbers.

### "What is a vector database?"
A database optimized for finding "which numbers are closest to these numbers?" — i.e. which text chunks mean something similar to your question.

### "Why does the AI still give wrong answers sometimes?"
Usually because:
- The right chunk wasn't found (bad retrieval)
- Chunks were too big or too small
- The answer wasn't actually in your documents

Fix: better chunking, better search, or tell the LLM to say "I don't know" when unsure.

### "What's the difference between RAG and ChatGPT?"
ChatGPT answers from memory. RAG answers from **your documents** that you provide.

---

## RAG vs Other Approaches (Simple)

| Approach | When to Use | Beginner Note |
|----------|-------------|---------------|
| **RAG** | Chat with your own docs | ✅ Start here for "chat with PDF" apps |
| **Fine-tuning** | Change how the AI talks/behaves | Advanced — don't need this to learn RAG |
| **Long context** | Paste entire doc into one prompt | Works for small docs only; gets expensive |
| **Agents** | AI that takes actions (calls APIs, etc.) | Learn RAG first, then explore agents |

---

## Learning Roadmap

Follow this order. Check off each step as you complete it.

- [ ] **Step 1** — Read this README and understand the 8 steps
- [ ] **Step 2** — Build "chat with one PDF" using Python + Chroma
- [ ] **Step 3** — See which chunks were retrieved for each answer
- [ ] **Step 4** — Try different chunk sizes and compare results
- [ ] **Step 5** — Add "I don't know" when context doesn't have the answer
- [ ] **Step 6** — Build a simple chat UI (Streamlit)
- [ ] **Step 7** — Test with 10 questions and check if answers are correct

---

## One-Sentence Summary

> **RAG = search your documents for relevant info, then let the AI answer using only what it found.**

---

## Resources

### For Beginners
- [LangChain RAG Tutorial](https://python.langchain.com/docs/tutorials/rag/) — hands-on walkthrough
- [Chroma Getting Started](https://docs.trychroma.com/getting-started) — easiest vector DB to try
- [Ollama](https://ollama.com/) — run LLMs locally for free

### Go Deeper (When You're Ready)
- [Original RAG Paper (2020)](https://arxiv.org/abs/2005.11401)
- [LlamaIndex Docs](https://docs.llamaindex.ai/)

---

## About This Repo

This is a **learning journal**. Concepts are explained simply so any beginner can follow along. The repo will grow with code examples and projects as learning continues.

**Found something confusing?** Open an issue or improve this README — learning together is the goal!

---

*Last updated: learning in progress 🚀*
