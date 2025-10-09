# Semantic Recall on Memories

Production approach: semantic search retrieves only relevant memories for current conversation.

## Approach

On message:

1. Query rewriter generates keywords + search query from conversation
2. Parallel searches:
   - BM25 (keyword matching)
   - Semantic (embedding matching)
3. Merge w/ Reciprocal Rank Fusion
4. Take top-k (e.g. 10)
5. Include only these in system prompt
6. After response, save new memories w/ embeddings

## Pros

- Scales to large DBs (100s-1000s)
- Efficient context usage (only relevant)
- Lower cost/latency vs all memories
- Combines keyword precision + semantic understanding

## Cons

- More complex
- Requires embedding generation/storage
- May miss memories if retrieval poorly tuned

## When to Use

- Large DBs (100+)
- Production apps w/ many users
- Most memories not relevant to every conversation
- Efficient context required
