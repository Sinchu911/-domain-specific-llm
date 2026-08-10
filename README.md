# Domain-Specific LLM Assistant

A retrieval-augmented (RAG) question-answering system for a chosen domain (finance, legal, or medical), built entirely on free-tier tools. Optionally extended with LoRA fine-tuning of a small open-source model to adapt its response style and reasoning to the domain.

Unlike a generic chatbot wrapper, this project grounds every answer in real domain documents through retrieval, and (optionally) adapts the underlying model's behavior through fine-tuning — rather than relying on a single system prompt.

## Why this exists

Generic LLMs answer domain questions from general training data, which means they can be outdated, generic, or simply wrong on specifics (exact clauses, figures, guidelines). This project fixes that by:
1. **Retrieval (RAG)** — pulling relevant chunks from real domain documents before the model answers, so responses are grounded in actual source text.
2. **Fine-tuning (optional/stretch)** — adapting a small open model's tone and reasoning style to match how domain experts communicate.

## Architecture

**Indexing (offline, done once)**
```
Domain documents → Chunk & embed → Vector database
```

**Query (online, done per question)**
```
User question → Embed & search vector DB → Top-k chunks
→ Augmented prompt (chunks + question) → LLM → Answer
```

## Tech stack (100% free)

| Component | Tool | Why |
|---|---|---|
| Compute | Kaggle Notebooks (free GPU, 30 hrs/week) | Better free GPU quota than Colab |
| Embeddings | `sentence-transformers` (`all-MiniLM-L6-v2`) | Free, runs on CPU, no API cost |
| Vector DB | FAISS or ChromaDB | Open source, local, no hosting cost |
| Base LLM | Qwen2.5-0.5B-Instruct / Llama-3.2-1B-Instruct | Small enough to fine-tune on a free T4 |
| Fine-tuning | LoRA / QLoRA via `peft` + `bitsandbytes` | Trains a fraction of params, fits free GPU memory |
| Domain data | Public datasets (FiQA, MedQA, case law corpora, SEC filings, MedlinePlus) | No cost, no scraping needed |

## Project status

- [ ] Domain selected (finance / legal / medical)
- [ ] Document corpus collected
- [ ] Chunking + embedding pipeline
- [ ] Vector DB indexing
- [ ] Retrieval + prompt augmentation
- [ ] Baseline eval set (20-30 Q&A pairs with expected facts)
- [ ] RAG vs. no-RAG accuracy comparison
- [ ] LoRA fine-tuning (stretch goal)
- [ ] Simple UI / API

## Repo structure (planned)

```
├── data/
│   ├── raw/              # source documents
│   └── processed/        # chunked text
├── indexing/
│   ├── chunk.py          # splits docs into chunks
│   └── embed_and_store.py  # embeds chunks, builds vector DB
├── retrieval/
│   └── query.py          # embeds question, retrieves top-k, builds prompt
├── finetuning/            # optional
│   ├── prepare_dataset.py
│   └── train_lora.py
├── eval/
│   └── eval_set.json     # domain Q&A pairs for accuracy testing
├── app.py                 # simple interface tying it together
└── README.md
```

## What makes this different from a prompt wrapper

- Answers are grounded in retrieved source chunks, not just model memory — every response can cite what it pulled from.
- Retrieval quality is measured, not assumed: an eval set checks whether RAG actually improves factual accuracy over the base model.
- (Stretch) The model itself is adapted via fine-tuning, not just instructed via a system prompt.

## Setup

```bash
pip install sentence-transformers faiss-cpu chromadb peft bitsandbytes transformers accelerate
```

Full setup and run instructions will be added as each pipeline stage is built.

## Roadmap

1. Pick one domain and commit to it (don't split effort across three).
2. Build and validate the RAG pipeline end-to-end.
3. Build an eval set and measure RAG vs. no-RAG accuracy.
4. If time allows, fine-tune a small model with LoRA on domain Q&A data.
5. Wrap in a minimal UI for demo purposes.

## License

MIT
