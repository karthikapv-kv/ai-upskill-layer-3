# Participant Setup — Embeddings, Semantic Search & RAG

**~10 minutes, any day before the session.** Everything is free and runs in Google Colab; no installs, accounts or API keys beyond a Google account.

> Colab resets its machines when they're idle, so this check doesn't keep anything for the session day. It makes sure everything **works** on your account. On the day we re-run the setup cells, which is quick.

## 1. Open each notebook

| Notebook | Open in Colab |
|---|---|
| 1 — Embeddings Playground | [Open](https://colab.research.google.com/github/karthikapv-kv/ai-upskill-layer-3/blob/main/notebooks/01_embeddings_playground.ipynb) |
| 2 — Semantic Search | [Open](https://colab.research.google.com/github/karthikapv-kv/ai-upskill-layer-3/blob/main/notebooks/02_semantic_search.ipynb) |
| 3 — Choosing an Embedding Model | [Open](https://colab.research.google.com/github/karthikapv-kv/ai-upskill-layer-3/blob/main/notebooks/03_embedding_model_comparison.ipynb) |
| 4 — Basic RAG Retrieval | [Open](https://colab.research.google.com/github/karthikapv-kv/ai-upskill-layer-3/blob/main/notebooks/04_basic_rag_retrieval.ipynb) |
| 5 — The Acme Knowledge Assistant | [Open](https://colab.research.google.com/github/karthikapv-kv/ai-upskill-layer-3/blob/main/notebooks/05_complete_rag_assistant.ipynb) |

## 2. Check Notebooks 1–4 (CPU is fine)

In each one, run the cells under **"0. Setup"** (Shift + Enter).

- ✅ **Expected:** NB1 prints "Ready: sentence-transformers/all-MiniLM-L6-v2"; NB2 shows a table of the 18 Acme documents; NB3 prints "18 documents, 15 evaluation queries"; NB4's setup cells print nothing (that's fine), and its section 1 shows a table of 6 runbooks.
- A warning about `HF_TOKEN` or "unauthenticated requests" is **safe to ignore**.
- If Colab asks to "restart the session" after installing, click **Restart**, then run the setup cells again.

## 3. Check Notebook 5 (needs a GPU)

1. **Runtime → Change runtime type → T4 GPU → Save.** It's free, but availability isn't guaranteed.
2. Run the cells down to and including **"2. Load the LLM"**. This downloads a ~4 GB model, usually in a few minutes.
3. ✅ **Expected:** `Loaded Qwen/Qwen3-1.7B on cuda`.
   - `on cpu` means you didn't get a GPU. It still works, but answers are slow.
   - `prompt-only mode` means the model couldn't load. Tell the instructor.

## 4. On the day

- Sit with your pair; one laptop per pair is fine.
- At the start, open Notebooks 1 and 2 and run their setup cells.
- **At the break**, open Notebook 5 on a **T4 GPU** and run it down to "2. Load the LLM", so it's ready for the last hands-on.

## If something fails

| Problem | Try |
|---|---|
| Install errors / "restart session" | Restart the session, then run the setup cells again |
| Download fails or times out | Run the cell again; downloads resume |
| "Cannot connect to GPU backend" | Use CPU for now; answers are slower but work |
| "Session crashed: out of RAM" | Runtime → Restart session; run each cell once from the top |
| Anything else | Note the error message and bring it to the session |
