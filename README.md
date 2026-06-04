# LEC-AI-Retrieval-Honest-Comparison

# Corpus Selection: Why Wikipedia Pages from ML/NLP Domain

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

My corpus is not perfect. Here is what is missing:

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

