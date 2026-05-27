# 🤖 RAG Against the Machine

> A Retrieval-Augmented Generation system for querying the [vLLM](https://github.com/vllm-project/vllm) codebase using hybrid search and a local LLM.

![RAG Architecture Diagram](assets/architec.jpg)

---

## 📖 What is this?

**RAG Against the Machine** is a fully local RAG pipeline that lets you ask natural language questions about the vLLM repository — its source code, documentation, and architecture — and get grounded, evidence-based answers.

It combines **hybrid retrieval** (BM25 + vector embeddings) with the **Qwen3-0.6B** language model to generate accurate answers without relying on any external API.

---

## ✨ Features

- **Adaptive chunking** — context-aware splitting that respects Python class/function boundaries and Markdown headers
- **Hybrid retrieval** — combines BM25 lexical search (`bm25s`) with semantic vector search (ChromaDB + `all-MiniLM-L6-v2`)
- **Local LLM generation** — uses `Qwen/Qwen3-0.6B` via HuggingFace Transformers, fully offline
- **Persistent indexes** — both BM25 and ChromaDB indexes are saved to disk for fast reuse
- **CLI interface** — simple commands for indexing and querying
- **Configurable chunk sizes** — from 150 to 2000 characters, tuned for mixed code + docs

---

## 🏗️ Architecture

The pipeline follows these sequential stages:

```
vLLM Repo Files
      │
      ▼
 [1] Ingestion        ← Reads .py and .md files from the vLLM repo
      │
      ▼
 [2] Chunking         ← Adaptive, file-type-aware splitting (150–2000 chars)
      │
      ▼
 [3] Indexing         ← BM25 index (bm25s) + ChromaDB vector store
      │
      ▼
 [4] Retrieval        ← Hybrid search: lexical + semantic, deduplicated & ranked
      │
      ▼
 [5] Generation       ← Qwen3-0.6B generates answer from retrieved context
      │
      ▼
    Answer
```

---

## 🚀 Quick Start

### 1. Install dependencies

```bash
make install
```

This uses [`uv`](https://github.com/astral-sh/uv) to install all project dependencies defined in `pyproject.toml`.

### 2. Index the vLLM repository

```bash
make run
# or equivalently:
uv run -m src index --max_chunk_size 2000
```

This will ingest, chunk, and index the entire vLLM repository. ChromaDB indexing takes up to ~5 minutes. The indexes are persisted to `data/processed/`.

### 3. Ask a question

```bash
uv run python -m src answer "What are the key capabilities of Ray Serve LLM for vLLM deployment?"
```

---

## ⚙️ Configuration

| Parameter | Default | Description |
|-----------|---------|-------------|
| `--max_chunk_size` | `2000` | Maximum characters per chunk (min: `150`) |

Chunk size affects the trade-off between retrieval precision and context richness. Smaller chunks improve recall; larger chunks provide more context to the LLM.

---

## 🔍 Retrieval Strategy

The system uses **Hybrid Retrieval** combining two complementary approaches:

**Lexical Search (BM25)**
Uses the `bm25s` library for term-frequency-based retrieval. Effective for exact keyword matches and technical identifiers.

**Semantic Search (Embeddings)**
Uses ChromaDB with the `all-MiniLM-L6-v2` embedding model to find semantically similar chunks even when exact keywords don't match.

**Ranking & Deduplication**
The `find_top_k` method merges results from both engines, removes duplicates, and returns the top-k most relevant snippets to use as LLM context.

---

## ✂️ Chunking Strategy

Chunking is file-type aware and uses a hierarchical approach:

- **Python files** → splits prioritize `class` and `def` boundaries
- **Markdown files** → splits prioritize headers (`#`, `##`, `###`)
- **Threshold-based breaks** → uses 80–90% of `max_chunk_size` to find the best structural boundary
- **Fallback** → when no logical separator is found, falls back to newlines or punctuation

This preserves semantic coherence and avoids cutting mid-function or mid-section.

---

## 📊 Performance

| Metric | Value |
|--------|-------|
| Recall@5 | > threshold (consistent across test queries) |
| Indexing time (ChromaDB) | ≤ 5 minutes for the full vLLM repo |
| Query throughput | ~1 query/minute |

---

## 📁 Project Structure

```
IA_LLM_RAG/
├── src/                    # Main source code
├── assets/                 # Architecture diagram and other media
├── filetype_scanner/       # File-type detection utilities
├── __init__.py
├── pyproject.toml          # Project metadata and dependencies
├── Makefile                # Convenience commands
└── uv.lock                 # Locked dependency versions
```

---

## 🛠️ Tech Stack

| Component | Library |
|-----------|---------|
| Lexical search | `bm25s` |
| Vector store | `chromadb` |
| Embeddings | `sentence-transformers` (`all-MiniLM-L6-v2`) |
| LLM | `Qwen/Qwen3-0.6B` via `transformers` |
| Inference backend | `torch` + `accelerate` |
| Data validation | `pydantic` |
| CLI | `fire` |
| Package manager | `uv` |

---

## 📋 Requirements

- Python ≥ 3.10
- [`uv`](https://github.com/astral-sh/uv) installed
- Sufficient disk space for ChromaDB index and model weights (~1–2 GB)
- GPU optional but recommended for faster generation

---

## 📄 License

This project is open source. See [LICENSE](LICENSE) for details.
