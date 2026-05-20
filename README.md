# n8n-projects

A collection of production-ready n8n workflows built for real-world automation — covering AI agents, WhatsApp bots, ticketing integrations, RAG pipelines, and more.

---

## Projects

| Project | Description |
|---------|-------------|
| [Ticket Opening and Closing via WhatsApp](./Ticket%20Opening%20and%20Closing%20via%20WhatsApp) | Automates IT ticket lifecycle (open, track, close) via WhatsApp groups, integrated with Aranda ASMS and Supabase |
| [WhatsApp RAG Bot - Evolution API + Pinecone](./WhatsApp%20RAG%20Bot%20-%20Evolution%20API%20%2B%20Pinecone) | AI agent on WhatsApp with RAG (Pinecone + Gemini), debounce buffer, conversation memory, and Google Drive auto-indexing |

---

## Stack

Most workflows in this repo are built on top of:

- **[n8n](https://n8n.io)** — workflow automation platform
- **[Evolution API](https://github.com/EvolutionAPI/evolution-api)** — WhatsApp gateway
- **[Supabase](https://supabase.com)** — open-source PostgreSQL database
- **[Pinecone](https://www.pinecone.io)** — vector store for RAG
- **[OpenAI](https://openai.com)** / **[Google Gemini](https://deepmind.google/technologies/gemini)** / **[OpenRouter](https://openrouter.ai)** — LLMs and embeddings
- **[Aranda ASMS](https://www.arandasoft.com)** — IT service management

---

## Getting Started

Each project folder contains its own `README.md` with:

- Architecture overview
- Required credentials
- Configuration instructions
- Webhook endpoints
- Notes and caveats

To use any workflow, import the `.json` file into your n8n instance via **Settings → Import workflow**.

---

## Contributing

This is a personal/team repository. New projects will be added over time. Each workflow should include:

- A descriptive folder name
- The exported `.json` workflow file
- A `README.md` explaining the project

---
