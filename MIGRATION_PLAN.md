# Migration Plan: TypeScript Docs → Emails for Retrieval Exercises

## Context

Exercise 01.01-retrieval-with-bm25 has been successfully migrated from using TypeScript documentation (`datasets/ts-docs`) to using emails (`datasets/emails.json`). This document provides a plan to migrate the remaining retrieval exercises.

## Completed: 01.01-retrieval-with-bm25

**What was changed:**
- `bm25.ts` - Switched from `loadTsDocs()`/`searchTypeScriptDocs()` to `loadEmails()`/`searchEmails()`
- `chat.ts` - Updated system prompts and result formatting for email context
- `readme.md` - Updated exercise description from TypeScript docs to emails
- `client/root.tsx` - Changed default query from "Explain TypeScript generics" to email query

**Key patterns:**
- Emails loaded from `datasets/emails.json` as JSON array
- Search combines `${email.subject} ${email.body}` for BM25
- Email type includes: id, from, to, subject, body, timestamp, threadId, etc.
- Result display shows from/to/subject metadata + body content
- System prompt: "helpful email assistant" instead of "TypeScript docs assistant"

## Remaining Exercises to Migrate

### 01.02-embeddings (problem + solution)

**Files to modify:**
- `problem/api/create-embeddings.ts`
- `solution/api/create-embeddings.ts`
- `problem/api/chat.ts`
- `solution/api/chat.ts`
- `problem/readme.md`
- `problem/client/root.tsx` (if exists)
- `solution/client/root.tsx` (if exists)

**Key changes:**
- Rename `loadTsDocs()` → `loadEmails()`
- Rename `embedTsDocs()` → `embedEmails()`
- Rename `searchTypeScriptDocs()` → `searchEmails()`
- Change cache key from `'ts-docs-google'` to `'emails-google'`
- Update embeddings to work on email content (combine subject + body)
- Change return type from `{filename, content, score}` to `{email, score}`
- Update system prompts in `chat.ts` to email assistant
- Update result formatting in `chat.ts` to show email metadata
- Update `readme.md` documentation
- Update default queries in client

**Special considerations:**
- Embeddings cache path stays in `data/` directory
- Record key should be email `id` instead of filename
- `loadEmails()` returns array, may need to create Map by id for embeddings lookup

### 01.03-rank-fusion (explainer only)

**Files to modify:**
- `explainer/api/bm25.ts`
- `explainer/api/embeddings.ts`
- `explainer/api/search.ts`
- `explainer/api/chat.ts`
- `explainer/api/utils.ts` (if it has `loadTsDocs`)
- `explainer/readme.md`
- `explainer/client/root.tsx` (if exists)

**Key changes:**
- Update `bm25.ts` to use emails (rename `searchTypeScriptDocsViaBM25` → `searchEmailsViaBM25`)
- Update `embeddings.ts` for emails
- Update `search.ts` rank fusion to combine email results
- Update `chat.ts` system prompts and formatting
- Update `utils.ts` if it contains shared loading functions
- Update `readme.md` examples
- Update default queries in client

**Special considerations:**
- Rank fusion combines BM25 + embeddings results
- Need consistent email result format across both methods

### 01.04-query-rewriting (problem + solution)

**Files to modify:**
- `problem/api/bm25.ts`
- `solution/api/bm25.ts`
- `problem/api/embeddings.ts`
- `solution/api/embeddings.ts`
- `problem/api/search.ts`
- `solution/api/search.ts`
- `problem/api/chat.ts`
- `solution/api/chat.ts`
- `problem/api/utils.ts` (if exists)
- `solution/api/utils.ts` (if exists)
- `problem/readme.md`
- `problem/client/root.tsx` (if exists)
- `solution/client/root.tsx` (if exists)

**Key changes:**
- Same as rank-fusion for BM25/embeddings/search files
- Update query rewriting prompts to generate email-appropriate queries
- Update system prompts in `chat.ts`
- Update result formatting
- Update `readme.md` examples
- Update default queries in client

### 01.05-reranking (problem + solution)

**Files to modify:**
- `problem/api/bm25.ts`
- `solution/api/bm25.ts`
- `problem/api/embeddings.ts`
- `solution/api/embeddings.ts`
- `problem/api/search.ts`
- `solution/api/search.ts`
- `problem/api/chat.ts`
- `solution/api/chat.ts`
- `problem/api/utils.ts` (if exists)
- `solution/api/utils.ts` (if exists)
- `problem/readme.md`
- `problem/client/root.tsx` (if exists)
- `solution/client/root.tsx` (if exists)

**Key changes:**
- Same as previous exercises for base files
- Update reranker system prompt to work with emails
- Change from "documentation pages" to "emails" in reranker instructions
- Update ID assignment to work with email results
- Update result display to show email metadata
- Update `readme.md` examples
- Update default queries in client

## Consistent Email Type Definition

Use this type across all exercises:

```typescript
export type Email = {
  id: string;
  from: string;
  to: string;
  subject: string;
  body: string;
  timestamp: string;
  threadId?: string;
  inReplyTo?: string;
  references?: string[];
  labels?: string[];
  arcId?: string;
  phaseId?: number;
};
```

## Consistent Loading Pattern

```typescript
import { readFile } from 'fs/promises';
import path from 'path';

export const loadEmails = async () => {
  const EMAILS_LOCATION = path.resolve(
    import.meta.dirname,
    '../../../../../datasets/emails.json',
  );

  const content = await readFile(EMAILS_LOCATION, 'utf8');
  const emails: Email[] = JSON.parse(content);

  return emails;
};
```

## Consistent Search Text Pattern

For BM25 and embeddings, combine subject + body:
```typescript
emails.map((email) => `${email.subject} ${email.body}`)
```

## Consistent Result Display Pattern

```typescript
const from = result.email?.from || 'unknown';
const to = result.email?.to || 'unknown';
const subject = result.email?.subject || `email-${i + 1}`;
const body = result.email?.body || '';
const score = result.score;

return [
  `### 📧 Email ${i + 1}: [${subject}](#${subject.replace(/[^a-zA-Z0-9]/g, '-')})`,
  `**From:** ${from}`,
  `**To:** ${to}`,
  `**Relevance Score:** ${score.toFixed(3)}`,
  body,
  '---',
].join('\n\n');
```

## Consistent System Prompt Pattern

```typescript
system: `You are a helpful email assistant that answers questions based on email content.
  You should use the provided emails to answer questions accurately.
  ALWAYS cite sources using markdown formatting with the email subject as the source.
  Be concise but thorough in your explanations.
`
```

## Example Default Queries

Replace TypeScript-related queries with email queries like:
- "What did David say about the mortgage application?"
- "Find emails about the house purchase"
- "What's the status of my booking?"
- "Who sent me information about consultations?"

## Execution Order

1. **01.02-embeddings** - Foundation for later exercises
2. **01.03-rank-fusion** - Combines BM25 + embeddings
3. **01.04-query-rewriting** - Builds on rank fusion
4. **01.05-reranking** - Builds on query rewriting

## Important Notes

- **DO NOT** change any exercise logic or learning objectives
- **ONLY** swap the data source from TypeScript docs to emails
- Preserve all TODOs in problem files
- Maintain exercise structure (problem/solution/explainer)
- Test that cache paths for embeddings still work
- Ensure email IDs are used consistently for lookups
- Check that all file imports are updated (e.g., `searchTypeScriptDocs` → `searchEmails`)
