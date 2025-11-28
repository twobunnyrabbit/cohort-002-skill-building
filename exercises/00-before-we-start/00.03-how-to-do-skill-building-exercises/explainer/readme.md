Welcome to the [skill building repository](git@github.com:ai-hero-dev/cohort-002-skill-building.git)! This is where you're going to do most of your skill building work throughout the course.

## The Exercises Directory

The most important directory here is `exercises`, where you'll find each section organized by topic and day.

The structure looks something like this:

```
exercises/01-retrieval-skill-building/
├── 01.01-retrieval-intro
├── 01.02-bm25
├── 01.03-retrieval-with-bm25
├── 01.04-embeddings
├── 01.05-rank-fusion
└── 01.06-query-rewriting
```

You have retrieval skill building, then retrieval project work, retrieval day two skill building, retrieval day two project work, and so on.

## Two Types of Exercises

Inside each section, you'll find a set of exercises. Some are **explainers** with just a `readme.md` and code to explore. Others are **problem and solution pairs** where you do the work yourself.

### Explainer Exercises

Explainer exercises are designed to teach you concepts without requiring you to write code from scratch.

Here's what an explainer looks like:

```
01.02-bm25/
└── explainer
    ├── api
    ├── client
    ├── how-does-bm25-work.png
    ├── main.ts
    └── readme.md
```

The `01.01-retrieval-intro` exercise, for example, is just a `readme.md` with some diagrams about the importance of retrieval. The `01.02-bm25` explainer includes a `readme.md` and a `main.ts` file that runs a local dev server.

### Problem and Solution Exercises

Problem and solution exercises require you to complete tasks, with a solution provided for reference.

Here's what a problem and solution pair looks like:

```
01.03-retrieval-with-bm25/
├── problem
│   ├── api
│   ├── client
│   ├── main.ts
│   └── readme.md
└── solution
    ├── api
    ├── client
    └── main.ts
```

The `problem` folder contains the code where you'll do the work, while the `solution` folder shows the completed implementation.

## Running the Exercises

To run any exercise, open the terminal and execute:

```bash
pnpm dev
```

This will allow you to choose which exercise you want to run. You can use autocomplete to find exercises by their number or name.

For example, you can type `1.2` or `BM25` to find the BM25 exercise:

```txt
? Which exercise do you want to run?
01.02-bm25
```

## Completing Exercises

For **explainers**, your job is to:

- Review the instructions in the `readme.md`
- Understand the concepts presented
- Explore the playground or code examples

For instance, with the BM25 explainer, you'll explore the BM25 playground, review the dataset, test some basic keyword searches, and test different search queries to understand BM25 better.

For **problem and solution exercises**, your job is to:

- Follow the steps in the `readme.md`
- Complete the code tasks in the `problem` folder
- Check off steps as you complete them
- Reference the `solution` folder if you need help

To switch to a different exercise or view the solution, cancel out and run `pnpm dev` again. Select the same exercise number, then choose between `problem` and `solution`.

## Video Support

Every single exercise has an accompanying video from me explaining everything in depth. For problem and solution exercises, there's a separate video for both the problem walkthrough and the solution.

## The Feedback Loop

By doing these exercises, you're going to get a really tight feedback loop. This approach has proven to work again and again in my courses because it lets you practice immediately and see results right away.

You're all set! Let's get started.
