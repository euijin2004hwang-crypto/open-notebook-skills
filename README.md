# Open Notebook Skills 🧠📚

A collection of **Hermes Agent skills** for [Open Notebook](https://github.com/lfnovo/open-notebook), an open-source AI research assistant and knowledge management tool that serves as a local-first alternative to NotebookLM.

These skills enable AI agents to programmatically interact with the Open Notebook REST API — search knowledge bases, run RAG-powered Q&A, manage sources and notes, generate podcasts, and configure models.

---

## 📦 Included Skills

### 1. `open-notebook` (English)
An English-language Hermes Agent skill for interacting with the Open Notebook REST API.

| Feature | Description |
|---------|-------------|
| 🔍 **Notebook Search** | Vector/text search across knowledge bases |
| 💬 **RAG Q&A** | 3-step chat workflow (session → context → execute) |
| 📄 **Source Management** | Add web pages, files, YouTube, PDF, and RSS as sources |
| 📝 **Note Management** | Create and retrieve notes |
| 🎙️ **Podcast Generation** | Auto-generate NotebookLM-style audio podcasts |
| 🔄 **Transformation Workflows** | Embedding rebuilds and batch transformations |
| ⚙️ **Model Configuration** | Provider discovery, model sync, auto-assignment |
| 🔗 **MCP Integration** | Native Hermes MCP client setup via `uvx open-notebook-mcp` |

### 2. `open-notebook-kr-guide` (한국어)
A detailed Korean-language Hermes Agent skill for the Open Notebook REST API.

| Feature | Description |
|---------|-------------|
| 🔍 **Notebook Search** | Keyword search via `/api/search` |
| 💬 **3-Step RAG Chat** | Session creation → context building → AI query execution |
| 📄 **Source CRUD** | Add, list, and manage sources with type field |
| 📝 **Note CRUD** | Create and update notes |
| ⚙️ **Configuration** | Model/provider queries and config changes |
| 🎙️ **Podcast Generation** | Auto-generate audio podcasts from notebook content |
| 🔗 **MCP Integration** | Hermes MCP server registration |

### 3. `hermes-tweet` (English)
A companion Hermes Agent skill for using Hermes Tweet before or alongside Open
Notebook research workflows.

| Feature | Description |
|---------|-------------|
| X/Twitter Research | Search tweets, profiles, replies, and public context |
| Source Collection | Gather social links and summaries before adding notebook sources |
| Monitoring | Track accounts or tweets that may become research material |
| Guarded Actions | Keep posting, replies, DMs, and account changes behind explicit approval |

---

## 🗂️ Notebook Naming Convention

Open Notebook uses `notebook:<id>` as its notebook identifier format. You can organize notebooks using a **naming convention with descriptive suffixes** to categorize them by purpose:

| Convention | Example | Purpose |
|------------|---------|---------|
| `-research` | `notebook:abc123-research` | Research / academic knowledge base |
| `-work` | `notebook:def456-work` | Work-related documents and notes |
| `-personal` | `notebook:ghi789-personal` | Personal notes, media, and creative work |
| No suffix | `notebook:jkl012` | Shared or default workspace |

This convention is entirely **user-defined** — Open Notebook itself does not enforce any naming pattern. Using tags helps agents:
- Determine which notebooks are available for querying
- Apply appropriate permission rules (read-only vs. read-write)
- Organize knowledge by domain or ownership

To discover available notebooks and their IDs:
```bash
curl -s --noproxy '*' http://localhost:****/api/notebooks | python3 -m json.tool
```

---

## 🚀 Installation

### Hermes Agent

```bash
# 1. Copy skill files to your Hermes skills directory
cp -r skills/* ~/.hermes/skills/

# 2. Configure MCP server (in ~/.hermes/config.yaml)
mcp_servers:
  open-notebook:
    command: uvx
    args: ["open-notebook-mcp"]
    env:
      OPEN_NOTEBOOK_URL: http://localhost:****/api/
```

### Run Open Notebook (Docker)

```bash
git clone https://github.com/lfnovo/open-notebook.git
cd open-notebook
docker compose up -d
```

> Web UI: `http://localhost:****`  
> API: `http://localhost:****/api/`  
> Swagger: `http://localhost:****/docs`

---

## ⚠️ Important Notes

### Proxy Environment
In environments using a SOCKS5 proxy (`ALL_PROXY=socks5://127.0.0.1:****`), **always include `--noproxy '*'`** in curl calls to localhost. Without it, requests route through the proxy and hang or fail.

```bash
# ✅ Correct
curl -s --noproxy '*' -X POST http://localhost:****/api/search ...

# ❌ Wrong (routes through proxy, hangs)
curl -s -X POST http://localhost:****/api/search ...
```

### API Path
Swagger UI may show paths as `/api/v1/`, but **the actual prefix is `/api/`**. Using `/api/v1/` returns 404 on every endpoint.

### Authentication
Auth is disabled by default — no API key needed for local instances.

---

## 💡 Usage Examples

### Search a Notebook
```bash
curl -s --noproxy '*' -X POST http://localhost:****/api/search \
  -H "Content-Type: application/json" \
  -d '{"query": "your search query", "notebook_id": "notebook:xxx"}'
```

### RAG Q&A (3-Step)
```bash
# 1. Create session
SESSION_ID=$(curl -s --noproxy '*' -X POST http://localhost:****/api/chat/sessions \
  -H "Content-Type: application/json" \
  -d '{"notebook_id": "notebook:xxx"}' | python3 -c "import sys,json; print(json.load(sys.stdin)['session_id'])")

# 2. Build context
CONTEXT=$(curl -s --noproxy '*' -X POST "http://localhost:****/api/notebooks/notebook:xxx/context" \
  -H "Content-Type: application/json" \
  -d '{"query": "your question"}' | python3 -c "import sys,json; print(json.load(sys.stdin)['context'])")

# 3. Send question with context
curl -s --noproxy '*' -X POST http://localhost:****/api/chat/execute \
  -H "Content-Type: application/json" \
  -d '{
    "session_id": "'"$SESSION_ID"'",
    "message": "your question",
    "context": "'"$CONTEXT"'"
  }'
```

### List All Notebooks
```bash
curl -s --noproxy '*' http://localhost:****/api/notebooks | python3 -m json.tool
```

---

## 📁 Repository Structure

```
open-notebook-skills/
├── README.md
├── LICENSE
└── skills/
    ├── hermes-tweet/
    │   └── SKILL.md
    ├── open-notebook/
    │   └── SKILL.md
    └── open-notebook-kr-guide/
        └── SKILL.md
```

---

## 📜 License

Apache 2.0

---

## 🔗 Related

- [Open Notebook on GitHub](https://github.com/lfnovo/open-notebook)
- [Hermes Agent](https://hermes-agent.nousresearch.com/)
