# Adding Search Tool to Agent

## Learning Goals

Transform manual search pipeline into agent-controlled tool. Configure multi-step workflows with `stopWhen`. Apply Anthropic prompt template structure. Stream conversational answers based on retrieved email context.

## Steps To Complete

### 1. Create `searchSemanticEmails` Tool and Integrate

Convert lesson 2.2 search algorithm into tool agent can invoke autonomously. Agent generates keywords + search query, calls tool, receives ranked emails with full content.

**Implementation:**

```ts
// src/app/api/chat/route.ts
import { google } from '@ai-sdk/google';
import {
  convertToModelMessages,
  createUIMessageStream,
  createUIMessageStreamResponse,
  stepCountIs,
  streamText,
  tool,
  UIMessage,
} from 'ai';
import { z } from 'zod';
import { loadEmails, searchWithRRF } from '@/app/search';

export const POST = async (req: Request): Promise<Response> => {
  const body: { messages: UIMessage[], id: string } = await req.json();

  const stream = createUIMessageStream({
    execute: async ({ writer }) => {
      const result = streamText({
        model: google('gemini-2.5-flash-lite'),
        system: `...`, // See step 2
        messages: convertToModelMessages(body.messages),
        tools: {
          searchSemanticEmails: tool({
            description: 'Search emails using both keyword and semantic search. Returns most relevant emails ranked by reciprocal rank fusion.',
            inputSchema: z.object({
              keywords: z
                .array(z.string())
                .describe('Exact keywords for BM25 search (names, amounts, specific terms)'),
              searchQuery: z
                .string()
                .describe('Natural language query for semantic search (broader concepts)'),
            }),
            execute: async ({ keywords, searchQuery }) => {
              const emails = await loadEmails();

              // Use search algorithm from lesson 2.2
              const bm25Results = await searchWithBM25(keywords, emails);
              const embeddingResults = await searchWithEmbeddings(searchQuery, emails);
              const rrfResults = reciprocalRankFusion([bm25Results, embeddingResults]);

              // Return top 5-10 full email objects
              const topEmails = rrfResults.slice(0, 10).map(r => ({
                id: r.email.id,
                from: r.email.from,
                to: r.email.to,
                subject: r.email.subject,
                body: r.email.body,
                timestamp: r.email.timestamp,
                score: r.score,
              }));

              return {
                emails: topEmails,
                count: topEmails.length,
              };
            },
          }),
        },
        stopWhen: stepCountIs(5),
      });

      writer.merge(result.toUIMessageStream());
    },
  });

  return createUIMessageStreamResponse({ stream });
};
```

**Notes:**
- Tool gets both `keywords` (BM25) and `searchQuery` (embeddings) separately - agent generates both
- Returns full email objects not just IDs - simpler for this lesson (metadata-first pattern in 4.4)
- `stopWhen: stepCountIs(5)` allows multi-step workflows - agent can call tool, analyze, respond
- Import search functions from `@/app/search.ts` (created in lesson 2.2)
- Tool executes search algorithm: BM25 + embeddings + RRF, returns top 10

### 2. Apply Anthropic Prompt Template Structure

Structure system prompt using Anthropic's recommended format: task context � background data � rules � the ask.

**Implementation:**

```ts
system: `You are a helpful email assistant that can search through a user's personal emails to answer questions.

# Background

Current date: ${new Date().toISOString().split('T')[0]}

You have access to a searchSemanticEmails tool that searches emails using both keyword (BM25) and semantic (embedding) search combined via reciprocal rank fusion.

# Rules

- When user asks about emails, ALWAYS call searchSemanticEmails tool first
- Generate specific keywords (names, amounts, exact terms) for BM25 search
- Generate natural language query (broader concepts) for semantic search
- Base your answer ONLY on returned email content
- Cite sources using email subjects in markdown format
- If search returns no results, say so clearly
- Never fabricate email content

# Task

Answer user questions about their emails based on search results. Be concise, accurate, and always cite sources.`
```

**Notes:**
- Anthropic template improves LLM instruction-following vs unstructured prompts
- Task context: establishes role ("email assistant")
- Background: provides facts (date, available tools, how they work)
- Rules: specific behavioral guidelines (when to search, how to cite)
- The ask: final instruction (answer based on results)
- Explicit "ALWAYS call tool first" prevents agent from guessing answers
- `stopWhen: stepCountIs(5)` critical - without it, agent stops after single tool call

## Additional Notes

**Agent vs Manual Search (lesson 1.3 vs 2.3):**
- Lesson 1.3: Manual keyword generation with `generateObject`, then search, then answer
- Lesson 2.3: Agent autonomously decides when to search, generates params, calls tool
- Agent approach: more flexible, handles multi-turn, can call tool multiple times
- Manual approach: more predictable, explicit control flow, easier to debug

**Tool Parameter Design:**
- Separate `keywords` and `searchQuery` params matches query rewriter pattern (lesson 1.6)
- Agent learns to use keywords for exact terms, searchQuery for concepts
- Example: `keywords: ["$5000", "invoice"]`, `searchQuery: "large payment requests"`
- Single param simpler but less precise - BM25 and embeddings want different inputs

**Multi-Step Agent Behavior:**
- `stopWhen: stepCountIs(5)` allows up to 5 LLM calls
- Typical flow: (1) initial message, (2) tool call, (3) tool result processing, (4) final answer
- Without `stopWhen`, agent stops immediately after tool call - no answer generated
- Can call tool multiple times if needed ("search for X, now search for Y")

**Tool Response Format:**
- Returns full email objects with all fields (from, to, subject, body, timestamp)
- Alternative: return email IDs, add `getEmailById` tool (covered in lesson 4.4)
- Full content simpler for this lesson but uses more tokens
- Score included for debugging - agent usually ignores, uses content

**Testing Strategy:**
- Test queries requiring keywords: "emails from Sarah", "$5000 invoice"
- Test queries requiring semantics: "vacation planning", "angry customers"
- Test queries requiring both: "meeting about budget" (keyword: budget, semantic: meeting)
- Verify agent calls tool before answering (check tool call in UI)
- Verify sources cited in markdown format

**Common Issues:**
- Agent answers without calling tool � strengthen "ALWAYS call tool first" rule
- Agent calls tool but stops � missing `stopWhen: stepCountIs(5)`
- Poor search results � verify lesson 2.2 search algorithm works standalone
- Agent confused by similar emails � reduce top N results to 5 instead of 10
