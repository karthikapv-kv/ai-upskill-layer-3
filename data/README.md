# Acme Pay fictional knowledge base

All content is fictional. Acme Pay, its services (`checkout-api`, `payment-service`, `ledger-service`, `refund-worker`), the `acmectl` CLI, incidents and transaction IDs are invented for this workshop.

| File | Used in | Contents |
|---|---|---|
| `kb_documents.json` | NB2, NB3 | 18 short documents (`document_id`, `title`, `category`, `content`) |
| `eval_queries.json` | NB3 | 15 paraphrased queries, each with one `expected_document_id` |
| `runbooks/manifest.json` + `runbooks/RB-0*.md` | NB4, NB5 | 6 longer Markdown runbooks (~700–870 words each) |
| `rag_test_questions.json` | NB5 | RAG test questions: answerable, partial and unanswerable cases |
| `word_vectors_fallback.json` | NB1 (optional) | 120 real GloVe `glove-wiki-gigaword-100` vectors (PDDL licence) for when the gensim download fails |

Validate with `python tools/validate_dataset.py` (structure) or `python tools/validate_dataset.py --embeddings` (structure + retrieval behaviour).

## Measured results (reference run)

These come from one run on 2026-09-27 (Python 3.12, sentence-transformers 6.1.0, transformers 5.17.0, torch 2.14.0, gensim 4.4.0, CPU). Exact scores can vary slightly with library versions and hardware, but rankings should hold.

**Keyword vs semantic, `My payment was charged twice.`** (all-MiniLM-L6-v2, top-3)
- Keyword: KB-002, KB-003, KB-017 (1 shared term each). KB-001 scores 0.
- Semantic: KB-017 (0.487), KB-003 (0.476), KB-001 (0.395).

**Keyword-wins cases:** `max.poll.interval.ms`: keyword → KB-007, semantic top-1 → KB-010. `payment-events.DLT`: keyword → KB-008, semantic top-1 → KB-002.

**Model comparison on `eval_queries.json`** (15 queries)
| Model | hit@1 | hit@3 |
|---|---|---|
| all-MiniLM-L6-v2 | 14/15 | 15/15 |
| BAAI/bge-small-en-v1.5 (with query prefix) | 12/15 | 14/15 |

**Runbook chunking** (120-word chunks, 20-word overlap): 48 chunks. Best chunk similarity: duplicate-payment question 0.689 (RB-01), project questions 0.577–0.643, Kubernetes question 0.314, "Friday afternoon" question 0.283 (answerable, but scores lower than the unanswerable one).

**GloVe analogies** (full vocabulary): king − man + woman → queen (0.770); paris − france + germany → berlin (0.885).

**Sentence similarities quoted in the guide and slides** (all-MiniLM-L6-v2, cosine)
| Pair | Score |
|---|---|
| I bought a new laptop. ↔ I purchased a new computer. | 0.861 |
| I bought a new laptop. ↔ My laptop won't turn on. | 0.491 |
| I bought a new laptop. ↔ I ate an apple. | 0.266 |
| My account was debited twice. ↔ I was charged two times for one purchase. | 0.572 |
| My account was debited twice. ↔ How can I reverse a duplicate transaction? | 0.429 |
| My account was debited twice. ↔ How do I change my debit card PIN? | 0.356 |
| I deposited money in the bank. ↔ I put cash into my savings account. | 0.742 |
| I deposited money in the bank. ↔ We sat beside the bank of the river. | 0.405 |
| We sat beside the bank of the river. ↔ We had a picnic on the riverside. | 0.434 |
| The payment succeeded. ↔ The payment failed. | 0.794 |
| I love this product. ↔ I hate this product. | 0.701 |
| Same sentence embedded by MiniLM vs by BGE (different spaces) | 0.366 |

Model facts: MiniLM 384 dims, 256-token limit, ~22.7M parameters, mean pooling + normalisation built in; BGE-small 384 dims, 512-token limit, ~33.4M parameters. RB-01 = 1,193 MiniLM tokens. Embedding the same sentence twice gave identical vectors.
