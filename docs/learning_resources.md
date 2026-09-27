# Learning Resources — Embeddings, Semantic Search & RAG

> Curated reading and videos for going deeper after the workshop. **Essential** items are the best place to start.

All links were checked on 2026-09-27; titles are as shown on the page. Times are approximate.

### Embeddings

| # | Resource | What to learn from it | Priority | Time |
|---|---|---|---|---|
| R1 | [Google Machine Learning Crash Course: Embeddings](https://developers.google.com/machine-learning/crash-course/embeddings) | Why one-hot vectors are limiting; embeddings as lower-dimensional learned vectors | **Essential** | 45–60 min |
| R2 | [The Illustrated Word2vec (Jay Alammar)](https://jalammar.github.io/illustrated-word2vec/) | Visual intuition for word vectors, king − man + woman ≈ queen, skip-gram and negative sampling | **Essential** | 30–45 min |
| R3 | [3Blue1Brown: Dot products and duality (Essence of Linear Algebra, Chapter 9)](https://www.3blue1brown.com/lessons/dot-products/) · [video](https://www.youtube.com/watch?v=LyGKycYT2v0) | Geometric meaning of the dot product, the heart of cosine similarity | Useful | 15 min |
| R4 | [3Blue1Brown: Transformers, the tech behind LLMs (Deep Learning Chapter 5)](https://www.3blue1brown.com/lessons/gpt/) · [video](https://www.youtube.com/watch?v=wjZofJX0v4M) | Beautiful visuals of embeddings where directions carry meaning. Watch the embedding part; the transformer internals are out of scope | Optional | 25 min |
| R4b | [3Blue1Brown: Essence of linear algebra (playlist)](https://www.youtube.com/playlist?list=PLZHQObOWTQDPD3MizzM2xVFitgF8hE_ab) · [Neural networks (playlist)](https://www.youtube.com/playlist?list=PLZHQObOWTQDNU6R1_67000Dx_ZCJB-3pi) · [channel](https://www.youtube.com/@3blue1brown) | Background, if you enjoy the style | Optional | as desired |
| R13 | [Efficient Estimation of Word Representations in Vector Space (Mikolov et al., 2013)](https://arxiv.org/abs/1301.3781) | The original Word2Vec paper (CBOW and skip-gram). Skim the abstract and figures | Optional | 15 min |
| R14 | [GloVe: Global Vectors for Word Representation (Stanford NLP)](https://nlp.stanford.edu/projects/glove/) | What GloVe is; pre-trained vectors under the PDDL licence | Optional | 10 min |

### Sentence embeddings and models

| # | Resource | What to learn from it | Priority | Time |
|---|---|---|---|---|
| R5 | [Sentence Transformers documentation](https://sbert.net/) | The library we use; quickstart for encoding and similarity | **Essential** | 20 min |
| R7 | [all-MiniLM-L6-v2 model card](https://huggingface.co/sentence-transformers/all-MiniLM-L6-v2) | Our main model: 384 dimensions, 256-token limit, training data, Apache-2.0 | **Essential** | 10 min |
| R8 | [bge-small-en-v1.5 model card](https://huggingface.co/BAAI/bge-small-en-v1.5) | Comparison model: 384 dimensions, 512 tokens, the query instruction, MIT | **Essential** | 10 min |
| R9 | [MTEB Leaderboard](https://huggingface.co/spaces/mteb/leaderboard) | How embedding models are benchmarked publicly. Browse, don't study | Optional | 10 min |
| R15 | [Sentence-BERT: Sentence Embeddings using Siamese BERT-Networks (Reimers & Gurevych, 2019)](https://arxiv.org/abs/1908.10084) | Where modern sentence embeddings came from. Abstract and introduction | Optional | 15 min |

### Semantic search

| # | Resource | What to learn from it | Priority | Time |
|---|---|---|---|---|
| R6 | [Sentence Transformers: Semantic Search](https://sbert.net/examples/sentence_transformer/applications/semantic-search/README.html) | Symmetric vs asymmetric search, a reference implementation, and a note on approximate search at scale | **Essential** | 20 min |
| R20 | [pgvector](https://github.com/pgvector/pgvector) | Vector similarity search inside Postgres, useful for the "why not PostgreSQL?" question | Optional | 10 min |

### RAG

| # | Resource | What to learn from it | Priority | Time |
|---|---|---|---|---|
| R10 | [RAG From Scratch (LangChain), GitHub](https://github.com/langchain-ai/rag-from-scratch) · [video playlist](https://www.youtube.com/playlist?list=PLfaIDFEXuae2LXbO1_PKyVJiQ23ZztA0x) | Indexing, retrieval and generation built up step by step. **Watch the first few videos (overview, indexing, retrieval, generation)**; the later advanced parts are beyond this workshop | **Essential** (first videos) | 30–40 min |
| R12 | [Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks (Lewis et al., 2020)](https://arxiv.org/abs/2005.11401) | The paper that coined "RAG"; parametric vs non-parametric memory. Abstract and introduction | Optional | 15 min |
| R16 | [Lost in the Middle: How Language Models Use Long Contexts (Liu et al., 2023)](https://arxiv.org/abs/2307.03172) | Why "just put everything in the prompt" is unreliable | Optional | 10 min (abstract) |
| R19 | [Introducing Contextual Retrieval (Anthropic, 2024)](https://www.anthropic.com/engineering/contextual-retrieval) | A clear, practical explainer of RAG retrieval failures and fixes. The advanced parts belong to the next session | Optional | 15 min |
| R21 | [Qwen3-1.7B model card](https://huggingface.co/Qwen/Qwen3-1.7B) · [0.5B fallback](https://huggingface.co/Qwen/Qwen2.5-0.5B-Instruct) | Our LLM: Apache-2.0, 1.7 B parameters, 32,768-token context, thinking vs non-thinking mode | **Essential** (for NB5) | 5 min |
| R17 | [Text Embeddings Reveal (Almost) As Much As Text (Morris et al., 2023)](https://arxiv.org/abs/2310.06816) | Embedding inversion, for FAQ 11 | Optional | 10 min (abstract) |
| R18 | [Hugging Face LLM Course](https://huggingface.co/learn/llm-course/chapter1/1) | Broader background on transformers and LLMs | Optional | hours |

### Vector databases

| # | Resource | What to learn from it | Priority | Time |
|---|---|---|---|---|
| R11 | [Chroma documentation](https://docs.trychroma.com/docs/overview/introduction) · [Manage collections](https://docs.trychroma.com/docs/collections/manage-collections) · [Add data](https://docs.trychroma.com/docs/collections/add-data) · [Query and get](https://docs.trychroma.com/docs/querying-collections/query-and-get) | Exactly the three operations we use: create a collection, add vectors with text and metadata, query | Useful | 20 min |
| R22 | [gensim-data](https://github.com/piskvorky/gensim-data) | Where `glove-wiki-gigaword-100` comes from (128 MB, 400K vocabulary, PDDL) | Optional | 5 min |

**Model licences (verified on the model cards, 2026-09-27):** all-MiniLM-L6-v2 is Apache-2.0; bge-small-en-v1.5 is MIT; Qwen3-1.7B and Qwen2.5-0.5B-Instruct are Apache-2.0. None are gated. GloVe vectors are under the ODC Public Domain Dedication and License (PDDL).
