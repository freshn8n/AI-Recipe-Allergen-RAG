# AI Recipe Allergen RAG

![Workflow overview: Google Drive to Supabase ingestion pipeline and Telegram AI agent](rag-workflow-thumbnail-v2.png)

An AI assistant that answers allergen and ingredient questions from a restaurant's own recipe documents, delivered through Telegram. Built entirely in n8n with no custom code.

> **Status:** Working prototype, built and tested on sample recipe documents. Not yet deployed in a live kitchen.

<!-- Demo video goes here -->

## The problem

In food service, staff and guests need fast, accurate answers about ingredients and allergens. Looking it up by hand during a busy service is slow, and answers vary depending on who you ask. I spent 10+ years in kitchens, and this is a question that comes up every shift.

## How it works

The system uses Retrieval-Augmented Generation (RAG):

- Recipe documents are converted into embeddings and stored in a vector database
- When someone asks a question, the most relevant recipes are retrieved
- That recipe text is passed to the LLM, which answers using only what was retrieved
- The answer goes back to the user in Telegram

Because answers come from the actual recipe documents instead of the model's general knowledge, the assistant stays grounded in your menu and can say when a dish isn't in its data.

## Architecture

Two separate n8n workflows: one loads the data, one answers questions. Recipes are embedded once and stored, so each question is a fast lookup.

### Workflow 1: Document ingestion

```
Trigger (manual or scheduled)
  → Find parent folder (Google Drive)
  → List subfolders → loop folders
  → List files → loop files
  → Normalize metadata
  → Download file
  → OpenAI Embeddings
  → Supabase Vector Store (embeddings + text)
```

### Workflow 2: Question answering

```
Telegram Trigger (incoming message)
  → AI Agent
      ├─ OpenAI Chat Model
      ├─ Simple Memory (conversation context)
      └─ Vector Store tool → Supabase → OpenAI Embeddings
  → Send Telegram reply
```

Simple Memory lets people ask follow-ups naturally: "Does the salmon have dairy?" then "What about gluten?" The agent knows which dish is still being discussed.

## Tech stack

| Component | Tool |
|---|---|
| Orchestration | n8n |
| LLM | OpenAI chat model |
| Embeddings | OpenAI Embeddings API |
| Vector database | Supabase (pgvector) |
| Data source | Google Drive (recipe documents) |
| Interface | Telegram Bot API |

## Example questions

| Question | What the system does |
|---|---|
| Does the salmon dish contain gluten? | Retrieves the salmon recipe and checks its ingredients |
| What are the vegan options? | Searches all recipes and lists the dishes that fit |
| What's in the house vinaigrette? | Returns the full ingredient list for the vinaigrette |
| Is there shellfish in the seafood pasta? | Retrieves the pasta recipe and confirms or denies shellfish |

## Why RAG instead of a general chatbot?

A general chatbot doesn't know your recipes. Asked about allergens, it will guess based on what the dish usually contains. RAG limits the answer to what's in your recipe documents, which makes answers specific to your menu and easier to audit.

## Safety note

This is a lookup tool, not a replacement for kitchen verification. Answers are only as accurate as the recipe documents behind them, and recipes, substitutions, and cross-contact risks change. For severe allergies, always confirm with the kitchen.

## Results so far

- Two-workflow pipeline built and running end to end in n8n
- Tested on sample recipe documents
- Adding recipes only requires uploading files to Google Drive and re-running ingestion

## Roadmap

- Automatic re-ingestion when Drive documents change
- Allergen tags for faster filtering
- Web chat interface in addition to Telegram
- Separate knowledge bases per location
- Query logging for review

## Workflow and implementation

The n8n workflows, prompts, and configuration are not public. This repository documents the architecture and design for portfolio purposes. If you want a similar system for your business, reach out.

## Author

**Blair "Fresh" Jones**, AI automation builder, Nashville, TN
[LinkedIn](https://www.linkedin.com/in/blair-jones-43906112b) · [GitHub](https://github.com/freshn8n) · blairjmail@gmail.com

## License

© Blair Jones. All rights reserved. Shared for portfolio and educational purposes only. The workflow, configuration, and implementation may not be reproduced, distributed, or used commercially without written permission.
