# Cheat Sheet — Embeddings, Semantic Search & RAG

*Acme Pay Internal Engineering Knowledge Assistant · notebooks: `github.com/karthikapv-kv/ai-upskill-layer-3`*

## The story in one line

**Search by words** → words don't capture meaning → **embeddings** (text as vectors) → compare them with **cosine similarity** → **semantic search** → retrieve the relevant knowledge → give it to an LLM → **RAG**

## Four definitions

| | Definition |
|---|---|
| **Embedding** | A learned numerical representation of an object (word, sentence or document) where useful relationships between objects are reflected in the geometry of the vectors. MiniLM: any text → **384 numbers**. |
| **Cosine similarity** | How closely two vectors point in the same direction: **1** same · **0** unrelated · **−1** opposite. |
| **Semantic search** | Rank documents by the embedding similarity between the query and each document, so results match on meaning, not exact words. |
| **RAG** | Retrieval-Augmented Generation: retrieve relevant chunks, add them to the prompt, and let an LLM answer from them, with sources. |

```text
cosine_similarity(A, B) = (A · B) / (||A|| × ||B||)
A · B  = multiply matching numbers, add them up        (3,4)·(4,3) = 24
||A||  = square each number, add, take the root         √(3² + 4²) = 5
→ 24 / (5 × 5) = 0.96          Length-1 vectors: cosine = dot product
```

## The RAG pipeline in seven steps

| | Step | What it does | Acme |
|---|---|---|---|
| **Indexing** (offline) | 1 Document ingestion | Load docs + metadata | 6 runbooks + manifest |
| | 2 Chunking | Split docs into pieces | 120 words, 20 overlap → 48 chunks |
| | 3 Embedding | Chunk → vector | MiniLM, 384 numbers |
| | 4 Indexing | Store vector + text + metadata | Chroma collection |
| **Answering** (online) | 5 Retrieval | Embed the question, find nearest chunks | top-3 chunks |
| | 6 Augmentation | Instructions + chunks + question → prompt | chunks tagged `[RB-05::chunk-04]` |
| | 7 Generation | LLM answers and cites sources | Qwen3-1.7B (free, local) |

## Key terms

| Term | Meaning |
|---|---|
| Vector | A list of numbers (384 for MiniLM) |
| Embedding space | Where one model's vectors live; only compare vectors from the **same** model |
| Similarity | How close two vectors are; relative, **not** a percentage |
| Vector search | Find the k stored vectors nearest to a query vector (k-NN). Exact = score them all; approximate = indexed, faster at scale |
| Retrieval | Finding the most relevant chunks for a question |
| Chunk | A smaller piece of a long document, embedded and retrieved on its own |
| Vector database | Stores vector + text + metadata; returns the nearest vectors to a query |
| Context | The retrieved text placed in the prompt |
| Grounding | Basing an answer on provided evidence |
| Hallucination | A fluent answer not supported by evidence (an "unsupported answer") |

## Keyword vs semantic vs hybrid

| Use | When | Acme example |
|---|---|---|
| **Keyword** | Exact identifiers: error codes, IDs, config keys | `max.poll.interval.ms` → KB-007 ✓ (semantic's top result ✗) |
| **Semantic** | Paraphrases, symptoms in your own words | "charged twice" → *reversing duplicate debit transactions* ✓ (keyword: 0 shared words) |
| **Hybrid** | Both kinds of queries | 0.7 × semantic + 0.3 × keyword fixes the identifiers, but can push down no-shared-word matches |

## Rules of thumb

- **Scores are relative.** Use them to rank; ranges are model-specific. "The payment succeeded" ↔ "The payment failed" = **0.794**: high ≠ same meaning.
- **One model on both sides.** The same sentence in MiniLM vs BGE scores only ~0.3 against itself. Changing models means re-embedding everything.
- **Choose models by testing.** Write real queries with expected docs, measure hit@k, read the failures (Acme: MiniLM 14/15 vs BGE 12/15, on a small sample). Bigger ≠ better. Consider domain-specific models only if your jargon is missed.
- **Chunk long documents.** MiniLM reads 256 tokens; the rest is silently cut. Whole RB-01 = 0.256 vs best chunk = 0.549 for a refund question.
- **Retrieval always returns something**, even for "How do I configure Kubernetes autoscaling?" (best score 0.314). Instruct the LLM to say when the docs don't contain the answer.
- **Why RAG:** private and fresh knowledge, grounded answers with sources, and better accuracy (Acme's real `acmectl` steps instead of an invented process), so fewer hallucinations.
- **RAG grounds answers but doesn't guarantee them.** Watch for poor retrieval, missing information, ignored context, invented details and wrong citations.

## Core code

```python
import chromadb
import numpy as np
from sentence_transformers import SentenceTransformer

model = SentenceTransformer("sentence-transformers/all-MiniLM-L6-v2")
embed_texts = lambda texts: model.encode(texts, normalize_embeddings=True)   # rows of length 1

# Semantic search (NumPy): one matrix multiplication
scores = doc_embeddings @ embed_texts([query])[0]
top_3 = np.argsort(-scores)[:3]

# Vector database (Chroma): we pass our own embeddings
collection = chromadb.EphemeralClient().create_collection(
    "acme_runbooks", configuration={"hnsw": {"space": "cosine"}}, embedding_function=None)
collection.add(ids=ids, documents=texts, metadatas=metadatas, embeddings=embed_texts(texts).tolist())
result = collection.query(query_embeddings=embed_texts([question]).tolist(), n_results=3)
similarity = 1 - result["distances"][0][0]                                   # Chroma returns distance

# RAG prompt: instructions + labelled chunks + question
messages = [{"role": "system", "content": INSTRUCTIONS},
            {"role": "user", "content": f"Documentation excerpts:\n{context}\n\nQuestion: {question}\nAnswer:"}]
```

## Learn more

[Google ML Crash Course: Embeddings](https://developers.google.com/machine-learning/crash-course/embeddings) · [The Illustrated Word2vec](https://jalammar.github.io/illustrated-word2vec/) · [Sentence Transformers: Semantic Search](https://sbert.net/examples/sentence_transformer/applications/semantic-search/README.html) · [RAG From Scratch (LangChain)](https://github.com/langchain-ai/rag-from-scratch) · [Chroma docs](https://docs.trychroma.com/docs/overview/introduction) · full list: `docs/learning_resources.md`

**Next session:** chunking strategies and vector databases in depth.
