## Steps To Complete

### Understanding Rank Fusion

- [ ] Learn why BM25 and semantic search scores can't be directly combined
  - BM25 returns larger numbers like 4.5 or 6
  - Semantic search returns numbers between 0 and 1
  - If combined directly, BM25 will dominate the results

- [ ] Review the `reciprocalRankFusion` function in `api/utils.ts`

```ts
const RRF_K = 60;

export function reciprocalRankFusion(
  rankings: { email: Email; score: number }[][],
): { email: Email; score: number }[] {
  const rrfScores = new Map<string, number>();
  const documentMap = new Map<
    string,
    { email: Email; score: number }
  >();

  rankings.forEach((ranking) => {
    ranking.forEach((doc, rank) => {
      const currentScore = rrfScores.get(doc.email.id) || 0;
      const contribution = 1 / (RRF_K + rank);
      rrfScores.set(doc.email.id, currentScore + contribution);
      documentMap.set(doc.email.id, doc);
    });
  });

  return Array.from(rrfScores.entries())
    .sort(([, scoreA], [, scoreB]) => scoreB - scoreA)
    .map(([emailId, score]) => ({
      email: documentMap.get(emailId)!.email,
      score,
    }));
}
```

- [ ] Understand the `RRF_K` constant
  - Higher values (like 60) are more forgiving—each ranking system contributes less aggressively
  - Lower values (like 30) make the algorithm more aggressive
  - 60 is the commonly used default

### Testing the Playground

- [ ] Start the dev server

```bash
pnpm run dev
```

- [ ] Open `localhost:3000` and search for "mortgage application"
  - You'll see emails with BM25, Semantic, and RRF scores displayed

- [ ] Compare results by clicking different ordering buttons
  - Click "BM25" to see exact keyword matching
  - Click "Semantic" to see meaning-based matching
  - Click "RRF" to see the combined ranking

- [ ] Search for "encouraging message from mum"
  - Notice how semantic search ranks emotionally related emails higher
  - Notice how BM25 struggles with this type of query
  - See how RRF creates a balanced result

- [ ] Experiment with different queries to discover patterns
  - Test keyword-heavy searches (BM25 excels)
  - Test meaning-based searches (semantic search excels)
  - Observe how RRF balances both approaches

### Verify the Implementation

- [ ] Check how `searchEmails` combines both algorithms in `api/search.ts`

```ts
export const searchEmails = async (opts: {
  keywordsForBM25: string[];
  embeddingsQuery: string;
}): Promise<EmailWithScores[]> => {
  const bm25SearchResults = await searchEmailsViaBM25(
    opts.keywordsForBM25,
  );

  const embeddingsSearchResults =
    await searchEmailsViaEmbeddings(opts.embeddingsQuery);

  const rrfResults = reciprocalRankFusion([
    bm25SearchResults,
    embeddingsSearchResults,
  ]);

  // ... rest of function
};
```

- [ ] Verify RRF scoring by examining email cards
  - Each card displays all three scores
  - The RRF score should balance both algorithms
  - Try different queries and verify the results make sense
