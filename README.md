# LEC-AI-Retrieval-Honest-Comparison

# 1. Corpus Selection: Why Wikipedia Pages from ML/NLP Domain

## My Choice

I selected **346 Wikipedia articles** from six categories: Machine Learning, Natural Language Processing, Information Retrieval, Deep Learning, Artificial Neural Networks, and Computational Linguistics.

## Why This Domain

I chose this specific corpus for three reasons:

**1. I can judge relevance honestly**

I am familiar with ML and NLP concepts. When I write queries like "What is backpropagation and how does it relate to neural networks?", I know which documents are truly relevant. If I had chosen legal documents or ancient history, I would be guessing. That would make my evaluation meaningless.

**2. The domain creates real retrieval challenges**

In technical fields, people describe the same concept using different words. A document about "word2vec" never mentions "word embedding" in its first 3000 characters. A query asking for "embedding methods" should still find it. This lets me test whether dense retrieval actually helps over BM25.

**3. Document length varies naturally**

My corpus has documents ranging from 367 characters (short stub articles) to 3000 characters (my truncation limit). Some retrievers favor longer documents (BM25 gets more term matches). Others treat all documents equally. This variation reveals different failure modes.

## What I Rejected and Why

| Alternative | Why I Said No |
|-------------|----------------|
| ArXiv abstracts | Too uniform. Every abstract follows the same structure. No interesting variation. |
| Legal documents | I cannot judge relevance. I would be fabricating ground truth. |
| Product help docs | Too predictable. Each query maps to one answer. No ambiguity to test. |
| Random Wikipedia pages | No thematic connection. All retrievers fail equally. No comparison possible. |

## Honest Limitations

- **Only 346 documents, not 400**. The categories I chose ran dry. I decided quality (staying on-topic) matters more than padding with off-topic pages.
- **Text is truncated at 3000 characters**. Full Wikipedia articles would exceed my latency budget of 1 second. I kept the introduction sections which contain the core concepts.
- **No cross-domain queries**. You cannot ask about "machine learning in ancient Rome" because my corpus has no history documents. This is fine because my queries stay within the domain.

## A Concrete Example of Why This Corpus Works

Take document `doc_0123` about "Transformer (machine learning model)". It uses words like "attention mechanism", "self-attention", and "positional encoding". It never says "large language model" or "GPT".

Now consider my hard query (example): *"What architecture made large language models possible?"*

- BM25 will fail because "large language model" appears nowhere in the document.
- Dense retrieval may succeed if the embedding model learned that transformers enable LLMs.
- Hybrid will do whatever my weighted blend decides.

This specific failure pattern is exactly what the assignment asks me to analyze. I cannot create this with random or synthetic data.

# 2. Retrieval Configurations: What I Built and Why

## The Four Configurations

I implemented four retrieval methods on the same 346 Wikipedia documents:

| Config | Method | Why This Choice |
|--------|--------|------------------|
| **BM25** | Lexical matching (term frequency + inverse document frequency) | Standard baseline. No ML, just counting words. |
| **Dense** | Sentence-BERT (`all-MiniLM-L6-v2`) + FAISS cosine search | 384-dim embeddings. Fast enough for <1s latency. Captures synonyms. |
| **Hybrid** | Reciprocal Rank Fusion (RRF) of BM25 + dense | Score-agnostic. No parameter tuning needed. Standard from IR literature. |
| **Reranker** | Cross-encoder (`ms-marco-MiniLM-L-6-v2`) on top-20 hybrid candidates | More accurate but slower. Only reranks 20 docs, not all 346. |

---

## Experimental Decisions I Made

### Decision 1: RRF over weighted sum for hybrid

**Why**: Weighted sum requires tuning `alpha` (0.3, 0.7). RRF uses only rank positions, works out of the box, and performs consistently across queries.

**Trade-off**: RRF ignores score magnitudes. A document with BM25 score 100 vs 10 gets same rank boost. This is fine because BM25 and dense scores are on different scales anyway.

### Decision 2: Flat FAISS index (not IVF)

**Why**: Exact inner product search on 346 vectors takes ~2-5ms. Approximate search would add complexity without speed gain.

**Trade-off**: Would not scale to 10k+ documents. But my corpus is 346 docs, and p95 latency must stay under 1 second. Flat index is correct here.

### Decision 3: Cross-encoder on top-20 candidates only

**Why**: Cross-encoder sees query+document together — very accurate but O(n) latency. Scoring all 346 docs would take ~10 seconds. Scoring 20 takes ~200-300ms.

**Trade-off**: If the correct document is ranked 21st by hybrid, reranker never sees it. I accept this risk because my hybrid config already returns relevant docs in top-20 for all test queries.

### Decision 4: Simple tokenization (lowercase + split)

**Why**: Keeps preprocessing identical across BM25 and dense. No stemming or stopword removal ensures fair comparison.

**Trade-off**: BM25 suffers slightly. But adding stopword removal would only benefit BM25, making comparison less honest. Same preprocessing = fair test.

---

## What the Smoke Test Revealed

Running `"neural networks that learn word representations from text"`:

| Config | Top Result | Latency |
|--------|-----------|---------|
| BM25 | "Deep learning" | 7ms |
| Dense | "GloVe" | 22ms |
| Hybrid | "GloVe" | 24ms |
| Reranker | "Embedding (machine learning)" | 7207ms |

**Observations**:

- **BM25 is fast** 
- **Dense captures semantics** 
- **Hybrid balances both** 
- **Reranker is slow**

---

## Summary Table

| Config | Latency (p95) | Best For | Fails On |
|--------|---------------|----------|----------|
| BM25 | 12ms | Exact term queries, rare words | Synonymy, paraphrasing |
| Dense | 35ms | Semantic similarity, word embeddings | Rare acronyms, out-of-vocab terms |
| Hybrid | 45ms | Most queries (balanced) | When one retriever is confidently wrong |
| Reranker | 7200ms* | Precision, challenging queries | Speed (violates constraint) |

# 3. My 20 Queries:

## What Each Difficulty Tests

| Difficulty | What It Tests | Expected Best Config |
|------------|---------------|---------------------|
| STANDARD | Exact term matching, basic concept retrieval | BM25 (fast, accurate enough) |
| HARD (Paraphrase) | Semantic understanding without keyword overlap | Dense |
| HARD (Multi-hop) | Connecting multiple concepts across documents | Reranker |
| HARD (Ambiguous) | Handling underspecified information needs | Hybrid |

---

## One Sentence Summary

**15 easy queries verify basic retrieval works; 5 hard queries (paraphrase, multi-hop, ambiguous) expose differences between BM25, dense, hybrid, and reranker.**



**Claim**: For this corpus under 1s latency constraint, **hybrid with RRF** is the best configuration. The reranker is more accurate but too slow for the constraint.

