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

**Notebook values quoted in the slides, guide, solutions and cheat sheet** (all-MiniLM-L6-v2 unless stated)

| Where | Values |
|---|---|
| NB1 billing set vs "My customer was charged twice." | paid two times 0.558 · card declined 0.386 · Kafka consumer 0.230 |
| NB1 how-to set vs "How do I reverse a duplicate transaction?" | refund someone who paid twice 0.541 · roll back deployment 0.303 · rate limit 0.104 |
| NB1 "key" context set | idempotency key ↔ Idempotency-Key header 0.717 · ↔ API key 0.354 · API key ↔ HTTP 429 0.480 |
| NB1 surprise pairs | deployment succeeded/failed 0.844 · payment succeeded/failed 0.794 |
| NB1 printed vectors | "My customer was charged twice." starts −0.004, 0.021, 0.015, 0.013, −0.013, −0.049, 0.033, −0.061, 0.061, −0.011 · "I bought a new laptop." (guide) starts −0.019, 0.002, 0.024, −0.043, 0.083, −0.067, 0.007, 0.007 |
| NB1 2D cosine examples | 1.000 · 0.960 · 0.000 · −1.000 (exact) |
| NB2 E4–E6 | HTTPS query: KB-019 0.465 after E4 (KB-002 0.273 before) · sluggish → KB-009 0.489 · undo release → KB-015 0.388 · Retry-After semantic top KB-017 0.452 · ledger-consumers semantic top KB-003 0.495 · max.poll.records semantic KB-010 0.205 (KB-007 0.099) · ERR_DB_POOL_TIMEOUT → KB-012 0.542 · DUPLICATE_DEBIT → KB-001 0.614 · TXN-5582-1147 → KB-017 0.212 |
| NB2 canonical (project question) | "My customer was charged twice…": KB-017 0.464 · KB-001 0.403 · KB-003 0.401 |
| NB3 cross-model, same sentence | 0.366 · 0.333 · 0.318 |
| NB4 | whole RB-01 ≈ its first 150 words 0.993 · refund question: whole doc 0.256 vs best chunk 0.549 (RB-01::chunk-06) · Kafka 0.643 (RB-02::chunk-05) · rollback 0.577 / 0.490 · receipts: RB-01::chunk-03 0.461, RB-02::chunk-00 0.421 (rank 2), RB-06::chunk-00 0.396 (rank 5) · partial question RB-01::chunk-07 0.557, chunk-06 0.526 |
| NB4 chunk-size experiment (pair lab) | top scores at 60/10: 0.652, 0.744, 0.490, 0.590, 0.600 · at 240/40: 0.577, 0.529, 0.437, 0.461, 0.620 · rephrasing: "undo last release" 0.346, "revert deploy" 0.485, "The new version is broken, go back" 0.295 |
| NB5 | Wi-Fi question retrieval 0.138 / 0.131 / 0.129 · Friday afternoon best 0.283 |
| GloVe (full vocabulary) | rome − italy + spain → madrid 0.704, seville 0.656, paris 0.655 · tokyo − japan + china → beijing 0.832 · runners-up monarch 0.684, throne 0.676, frankfurt 0.799, vienna 0.768 |
