---
name: open-notebook
description: "Interact with Open Notebook (lfnovo/open-notebook) via REST API — search knowledge bases, manage sources/notes, chat with notebooks, generate podcasts, and configure models"
trigger: "user asks to search, query, or interact with Open Notebook; mentions sources, notes, notebooks, or knowledge base; asks about AI research using local vector DB"
---

# Open Notebook — AI Agent Guide

Open Notebook is an open-source AI research assistant and knowledge management tool. This skill covers how an AI agent interacts with its REST API programmatically.

## Quick Reference

| Item | Value |
|------|-------|
| API Base | \`http://localhost:5055/api/\` |
| Web UI | \`http://localhost:8502/\` |
| Swagger Docs | \`http://localhost:5055/docs\` |
| Auth | Disabled (no API key needed) |
| Proxy Workaround | **\`--noproxy '*'\`** required for all curl calls |
| MCP Package | \`open-notebook-mcp\` (PyPI, via \`uvx\`) |
| Config | \`GET /api/config\` — check settings |

## ⚠️ Critical Rules

### 1. Always use \`--noproxy '*'\` with curl
The environment uses \`ALL_PROXY=socks5://127.0.0.1:40000\`. Every curl call to localhost **must** include \`--noproxy '*'\` or it hangs/fails.

### 2. API path is \`/api/\` — NOT \`/api/v1/\`
Swagger UI shows paths at \`/api/\`. The \`/api/v1/\` prefix returns 404 on every endpoint.

### 3. Source creation needs a \`type\` field
\`POST /api/sources\` requires \`type\` in body. Valid types: \`website\`, \`file\`, \`text\`, \`youtube\`, \`pdf\`, \`rss\`, \`sitemap\`.

---

## Core Workflows

### Search a Notebook
\`\`\`bash
curl -s --noproxy '*' -X POST http://localhost:5055/api/search \
  -H "Content-Type: application/json" \
  -d '{"query": "검색어", "notebook_id": "notebook:xxx"}' | python3 -m json.tool
\`\`\`

### Ask AI (RAG-based Q&A)
\`\`\`bash
curl -s --noproxy '*' -X POST http://localhost:5055/api/search/ask \
  -H "Content-Type: application/json" \
  -d '{"query": "질문", "notebook_id": "notebook:xxx"}'
\`\`\`

### Simple Ask (no session)
\`\`\`bash
curl -s --noproxy '*' -X POST http://localhost:5055/api/search/ask/simple \
  -H "Content-Type: application/json" \
  -d '{"query": "질문", "notebook_id": "notebook:xxx"}'
\`\`\`

### List Notebooks
\`\`\`bash
curl -s --noproxy '*' http://localhost:5055/api/notebooks | python3 -m json.tool
\`\`\`

### Get Notebook Detail (includes source/note counts)
\`\`\`bash
curl -s --noproxy '*' http://localhost:5055/api/notebooks/NOTEBOOK_ID | python3 -m json.tool
\`\`\`

### List Sources in a Notebook
\`\`\`bash
curl -s --noproxy '*' "http://localhost:5055/api/sources?notebook_id=NOTEBOOK_ID" | python3 -m json.tool
\`\`\`

### Create a Source
\`\`\`bash
curl -s --noproxy '*' -X POST http://localhost:5055/api/sources \
  -H "Content-Type: application/json" \
  -d '{
    "type": "website",
    "url": "https://example.com",
    "notebook_id": "notebook:xxx"
  }'
\`\`\`

### Create a Note
\`\`\`bash
curl -s --noproxy '*' -X POST http://localhost:5055/api/notes \
  -H "Content-Type: application/json" \
  -d '{
    "notebook_id": "notebook:xxx",
    "title": "노트 제목",
    "content": "내용"
  }'
\`\`\`

### Chat with a Notebook (3-step RAG)

**1️⃣ Create Session**
\`\`\`bash
SESSION_ID=$(curl -s --noproxy '*' -X POST http://localhost:5055/api/chat/sessions \
  -H "Content-Type: application/json" \
  -d '{"notebook_id": "notebook:xxx"}' | python3 -c "import sys,json; print(json.load(sys.stdin)['session_id'])")
\`\`\`

**2️⃣ Build Context**
\`\`\`bash
CONTEXT=$(curl -s --noproxy '*' -X POST "http://localhost:5055/api/notebooks/notebook:xxx/context" \
  -H "Content-Type: application/json" \
  -d '{"query": "질문 내용"}' | python3 -c "import sys,json; print(json.load(sys.stdin)['context'])")
\`\`\`

**3️⃣ Execute Chat**
\`\`\`bash
curl -s --noproxy '*' -X POST http://localhost:5055/api/chat/execute \
  -H "Content-Type: application/json" \
  -d "{
    \"session_id\": \"$SESSION_ID\",
    \"message\": \"질문 내용\",
    \"context\": \"$CONTEXT\"
  }" | python3 -m json.tool
\`\`\`

---

## Advanced Features

### Run a Transformation
\`\`\`bash
curl -s --noproxy '*' -X POST http://localhost:5055/api/transformations/execute \
  -H "Content-Type: application/json" \
  -d '{
    "notebook_id": "notebook:xxx",
    "transformation_id": "transformation:xxx"
  }'
\`\`\`

### Generate a Podcast
\`\`\`bash
curl -s --noproxy '*' -X POST http://localhost:5055/api/podcasts/generate \
  -H "Content-Type: application/json" \
  -d '{"notebook_id": "notebook:xxx"}' | python3 -m json.tool
\`\`\`

---

## MCP Integration (Hermes native MCP client)

\`\`\`yaml
# In ~/.hermes/config.yaml
mcp_servers:
  open-notebook:
    command: uvx
    args: ["open-notebook-mcp"]
    env:
      OPEN_NOTEBOOK_URL: http://localhost:5055
\`\`\`

## Pitfalls

1. **Proxy blocks localhost** → Always \`--noproxy '*'\` with curl
2. **Wrong API prefix** → \`/api/v1/*\` returns 404; use \`/api/*\`
3. **Source needs type** → Missing \`type\` field returns validation error
4. **No auth configured** → By default: no password
5. **Embedding rebuild is async** → Poll status endpoint for progress
