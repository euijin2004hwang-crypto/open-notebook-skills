---
name: open-notebook-kr-guide
description: "Open Notebook (lfnovo/open-notebook) REST API 한국어 가이드 — 노트북 검색, 소스/노트 관리, RAG 채팅, 모델 설정, Docker 운영까지 상세 설명"
trigger: "user asks about Open Notebook in Korean; mentions RAG, knowledge base search, source management in Korean context"
---

# Open Notebook 한국어 가이드

Open Notebook은 오픈소스 AI 연구 보조 및 지식 관리 도구입니다.

## 기본 정보

| 항목 | 값 |
|------|-----|
| API Base | \`http://localhost:5055/api/\` |
| Web UI | \`http://localhost:8502/\` |
| Swagger Docs | \`http://localhost:5055/docs\` |
| 인증 | 기본 비활성화 |
| 필수 옵션 | **\`--noproxy '*'\`** |

## ⚠️ 중요 규칙

### 1. curl에 \`--noproxy '*'\` 필수
SOCKS5 프록시(\`ALL_PROXY=socks5://127.0.0.1:40000\`) 환경에서 localhost 호출 시 반드시 필요.

### 2. API 경로는 \`/api/\`
\`/api/v1/\`은 404 반환.

---

## 3단계 RAG 챗 워크플로우

### 1️⃣ 노트북 검색
\`\`\`bash
curl -s --noproxy '*' -X POST http://localhost:5055/api/search \
  -H "Content-Type: application/json" \
  -d '{"query": "검색어", "notebook_id": "notebook:xxx"}'
\`\`\`

### 2️⃣ RAG 질문
\`\`\`bash
curl -s --noproxy '*' -X POST http://localhost:5055/api/search/ask \
  -H "Content-Type: application/json" \
  -d '{"query": "질문", "notebook_id": "notebook:xxx"}'
\`\`\`

### 3️⃣ 세션 기반 챗 (진짜 RAG)

**세션 생성 → 컨텍스트 빌드 → 질문 전송**

\`\`\`bash
# 1. 세션 생성
SESSION_ID=$(curl -s --noproxy '*' -X POST http://localhost:5055/api/chat/sessions \
  -H "Content-Type: application/json" \
  -d '{"notebook_id": "notebook:xxx"}' | python3 -c "import sys,json; print(json.load(sys.stdin)['session_id'])")

# 2. 컨텍스트 빌드
CONTEXT=$(curl -s --noproxy '*' -X POST "http://localhost:5055/api/notebooks/notebook:xxx/context" \
  -H "Content-Type: application/json" \
  -d '{"query": "질문 내용"}' | python3 -c "import sys,json; print(json.load(sys.stdin)['context'])")

# 3. 질문 전송 (컨텍스트 포함)
curl -s --noproxy '*' -X POST http://localhost:5055/api/chat/execute \
  -H "Content-Type: application/json" \
  -d "{
    \"session_id\": \"$SESSION_ID\",
    \"message\": \"질문 내용\",
    \"context\": \"$CONTEXT\"
  }"
\`\`\`

---

## 소스 관리

### 소스 생성
\`\`\`bash
curl -s --noproxy '*' -X POST http://localhost:5055/api/sources \
  -H "Content-Type: application/json" \
  -d '{"type": "website", "url": "https://example.com", "notebook_id": "notebook:xxx"}'
\`\`\`

\`type\` 필드 필수. 지원 타입: \`website\`, \`file\`, \`text\`, \`youtube\`, \`pdf\`, \`rss\`, \`sitemap\`

### 소스 목록 조회
\`\`\`bash
curl -s --noproxy '*' "http://localhost:5055/api/sources?notebook_id=NOTEBOOK_ID"
\`\`\`

### 노트 생성
\`\`\`bash
curl -s --noproxy '*' -X POST http://localhost:5055/api/notes \
  -H "Content-Type: application/json" \
  -d '{"notebook_id": "notebook:xxx", "title": "제목", "content": "내용"}'
\`\`\`

---

## 노트북 컨벤션

- **\`-H\` 접미사**: 하린 전용 노트북 (자유로운 검색/읽기/쓰기)
- **\`-H\` 없음**: 주인님 전용 (읽기 전용, 명령 기반)

---

## 모델 설정

\`\`\`bash
# 설정 확인
curl -s --noproxy '*' http://localhost:5055/api/config

# 모델 목록
curl -s --noproxy '*' http://localhost:5055/api/models | python3 -m json.tool

# 기본 모델 자동 할당
curl -s --noproxy '*' -X POST http://localhost:5055/api/models/auto-assign

# 공급자 상태 확인
curl -s --noproxy '*' http://localhost:5055/api/models/providers

# 설정 업데이트
curl -s --noproxy '*' -X PUT http://localhost:5055/api/settings \
  -H "Content-Type: application/json" \
  -d '{"chat_model": "gpt-4"}'
\`\`\`

---

## 팟캐스트 생성

\`\`\`bash
curl -s --noproxy '*' -X POST http://localhost:5055/api/podcasts/generate \
  -H "Content-Type: application/json" \
  -d '{"notebook_id": "notebook:xxx"}'
\`\`\`

---

## MCP 연동 (Hermes)

\`\`\`yaml
# ~/.hermes/config.yaml
mcp_servers:
  open-notebook:
    command: uvx
    args: ["open-notebook-mcp"]
    env:
      OPEN_NOTEBOOK_URL: http://localhost:5055
\`\`\`

---

## 주의사항

1. **프록시**: 모든 curl에 \`--noproxy '*'\` 필수
2. **API 경로**: \`/api/\`만 사용 (\`/api/v1/\` → 404)
3. **소스 타입**: \`type\` 필드 누락 시 오류
4. **임베딩 재구축**: 비동기 작업, 진행률 폴링 필요
