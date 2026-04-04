# AI-Recipe-Allergen-RAG
An AI-powered assistant that retrieves live recipe data to accurately answer allergen questions and look up dishes in real time — built entirely in n8n with no custom code.

📌 Overview

This project solves a real problem in food service: staff and customers need fast, accurate answers about ingredients and allergens — and traditional lookup methods are slow, inconsistent, and error-prone.
This system uses Retrieval-Augmented Generation (RAG) to:
	∙	Embed recipe documents into a vector database
	∙	Retrieve the most relevant recipe data when a question is asked
	∙	Pass that context to an LLM to generate an accurate, grounded response
	∙	Return the answer instantly via Telegram
Because the AI is grounded in real retrieved data — not just its training — it doesn’t hallucinate ingredients or guess at allergens.

🏗️ Architecture

This system is built as two separate n8n workflows — one for ingestion, one for querying. This is the correct production approach: data is embedded once and stored, then retrieved instantly on every query.
Workflow 1 — Document Ingestion Pipeline
Processes recipe documents from Google Drive and loads them into the vector database.
Trigger (Manual / Scheduled)
    ↓
Find Parent Folder (Google Drive)
    ↓
List Subfolders
    ↓
Loop Folders
    ↓
List Files Inside Current Folder
    ↓
Loop Files
    ↓
Edit Fields (normalize metadata)
    ↓
Download File
    ↓
Embeddings OpenAI (convert to vectors)
    ↓
Supabase Vector Store (store embeddings + content)
    ↓
Default Data Loader
Workflow 2 — AI Agent Query Pipeline
Handles live user queries via Telegram and returns grounded, accurate responses.
Telegram Trigger (incoming message)
    ↓
AI Agent
    ├── OpenAI Chat Model (LLM)
    ├── Simple Memory (conversation context)
    └── Answer Questions with Vector Store (RAG tool)
            ↓
        Supabase Vector Store
            ↓
        Embeddings OpenAI (embed the query)
            ↓
        OpenAI Chat Model (generate grounded response)
    ↓
Send Text Message (Telegram — response to user)

🛠️ Tech Stack

Component             Tool
Orchestraion.         n8n
LLM.                  Open AI (GPT-                                4/compatible API
Embeddings.           Open AI Embeddings API
Vector Database.      Supabase (pgvector)
Data Source.          Google Drive (recipe                         documents)
User Interface.       Telegram Bot API
Logging (Optional).   Notion/Google Sheets

⚙️ How It Works

Workflow 1 — Ingestion (Run Once Per Update)
	1.	n8n locates the parent folder in Google Drive and recursively lists all subfolders and files
	2.	Each recipe file is downloaded and passed through OpenAI Embeddings to generate a vector representation
	3.	The vector and original text are stored together in Supabase, making the data semantically searchable
Workflow 2 — Query (Every User Request)
	1.	A user sends a message via Telegram (e.g. “Does the Caesar salad contain dairy?”)
	2.	The Telegram Trigger fires and passes the message to the AI Agent node
	3.	The AI Agent uses three tools simultaneously:
	∙	OpenAI Chat Model — the core LLM for generating responses
	∙	Simple Memory — retains conversation context for follow-up questions
	∙	Answer Questions with Vector Store — searches Supabase for the most relevant recipe data, using OpenAI Embeddings to match the query semantically
	4.	The retrieved recipe context is passed to the LLM, which generates a grounded, accurate answer
	5.	The response is sent back to the user via Telegram
	
💡 Simple Memory means the agent remembers the conversation — users can ask follow-up questions naturally without repeating themselves. For example: “Does it have dairy?” → “What about gluten?” — the agent knows what dish is still being discussed.

💡 Why RAG Instead of Just ChatGPT?
A standard LLM (like ChatGPT) would try to answer allergen questions from its training data — which knows nothing about your specific recipes. It would guess, hallucinate ingredients, or give generic answers.
RAG fixes this by grounding every answer in your actual recipe data. The AI can only answer from what’s in your database — making it accurate, auditable, and safe for allergen use cases.

📋 Example Queries.

User Question.               System Behavior
Does the salmon dish
contain gluten?              Retrieves                                   salmon                                      recipe,                                     checks                                      ingreidents,                                answer                                      accurately.      
What are the vegan options?  Searches all                                recipes,                                    filters by                                  vegan tags
                             /ingredients, 
                             list options. 
What's in the house 
vinaigrette?                 Retrives                                    vinaigrette                                 recipe, return                              full ingred
                             ients list.
Is there shell fish in the 
seafood pasta?               Retrives pasta                              recipe,                                     confirms or                                 denies                                      shellfish                                   presence.

🔒 Workflow & Implementation

The internal n8n workflow, prompt structure, and configuration files for this project are proprietary and not publicly available.
This repository documents the architecture, design decisions, and outcomes for portfolio purposes only.
If you’re interested in a similar system for your business, feel free to reach out directly.

📈 Results & Impact

	∙	Eliminated manual recipe lookups during service
	∙	Reduced allergen query response time from minutes to seconds
	∙	Grounded AI responses in real data — no hallucinations
	∙	Scalable: adding new recipes only requires uploading to Google Drive

🗺️ Roadmap

	∙	Multi-restaurant support (separate knowledge bases per location)
	∙	Web chat interface (in addition to Telegram)
	∙	Automatic re-ingestion when Google Drive documents are updated
	∙	Allergen tagging system for faster filtering
	∙	Admin dashboard for query logging and review

	👤 Author
	
Blair Jones — IT & Automation Builder
📩 blairjmail@gmail.com
🔗 LinkedIn | GitHub

📄 License

© Blair Jones. All rights reserved. This project is shared for portfolio and educational purposes only. The workflow, configuration, and implementation are proprietary and may not be reproduced, distributed, or used commercially without explicit written permission.
