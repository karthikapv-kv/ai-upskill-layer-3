# Embeddings, Semantic Search & RAG — Hands-on Notebooks

Build the **Acme Pay Internal Engineering Knowledge Assistant**: search a fictional company's engineering docs by meaning, then let a free, local LLM answer developers' questions with sources.

> Search by words → words don't capture meaning → embeddings → cosine similarity → semantic search → retrieve knowledge → give it to an LLM → RAG

Everything runs in **Google Colab**, for free: no installs, accounts or API keys beyond a Google account.

## Notebooks

| # | Notebook | You'll learn | Open |
|---|---|---|---|
| 1 | Embeddings Playground | Turn text into vectors; compare them with cosine similarity | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/karthikapv-kv/ai-upskill-layer-3/blob/main/notebooks/01_embeddings_playground.ipynb) |
| 2 | Build Semantic Search | Keyword vs semantic search, and the idea of hybrid search | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/karthikapv-kv/ai-upskill-layer-3/blob/main/notebooks/02_semantic_search.ipynb) |
| 3 | Choosing an Embedding Model | Compare two models on Acme questions | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/karthikapv-kv/ai-upskill-layer-3/blob/main/notebooks/03_embedding_model_comparison.ipynb) |
| 4 | Basic RAG Retrieval | Document ingestion → chunking → embedding → indexing → retrieval | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/karthikapv-kv/ai-upskill-layer-3/blob/main/notebooks/04_basic_rag_retrieval.ipynb) |
| 5 | The Acme Knowledge Assistant | Augmentation → generation: answers with sources, and where RAG fails · **use a T4 GPU** | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/karthikapv-kv/ai-upskill-layer-3/blob/main/notebooks/05_complete_rag_assistant.ipynb) |

Each notebook runs on its own, top to bottom. Exercises marked ⏱️ are done in the session and 🏠 are take-home; solutions are in click-to-reveal cells at the end of each notebook.

## Before the session

Follow **[docs/participant_setup.md](docs/participant_setup.md)** (about 10 minutes): open each notebook and run its setup cells, and check Notebook 5 on a free **T4 GPU** (*Runtime → Change runtime type → T4 GPU*).

## Reference

- **[Cheat sheet](docs/participant_cheat_sheet.md):** definitions, the seven-step RAG pipeline, rules of thumb and core code on one page.
- **[Learning resources](docs/learning_resources.md):** where to go deeper.

## The data

`data/` holds the fictional Acme Pay knowledge base. The notebooks download these files automatically.

| File | Used in | Contents |
|---|---|---|
| `kb_documents.json` | NB1–NB3 | 18 short engineering docs (payments, Kafka, API, database, deployment, incidents) |
| `eval_queries.json` | NB3 | 15 questions, each with the doc that should answer it |
| `runbooks/` | NB4–NB5 | 6 longer Markdown runbooks + `manifest.json` |
| `rag_test_questions.json` | NB5 | Test questions: answerable, partial and unanswerable |
| `word_vectors_fallback.json` | — | Small set of GloVe word vectors (backup for an optional demo) |

Acme Pay, its services, people and incidents are all fictional.

## Models and tools (all free and open-source)

| Role | Model / library | Licence |
|---|---|---|
| Embeddings | `sentence-transformers/all-MiniLM-L6-v2` | Apache-2.0 |
| Comparison model | `BAAI/bge-small-en-v1.5` | MIT |
| LLM (runs inside Colab) | `Qwen/Qwen3-1.7B` (fallback `Qwen/Qwen2.5-0.5B-Instruct`) | Apache-2.0 |
| Vector database | ChromaDB (in memory) | Apache-2.0 |
| Chat UI (optional) | Gradio | Apache-2.0 |
