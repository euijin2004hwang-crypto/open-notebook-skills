# Open Notebook Skills 🧠📚

**Open Notebook** ([lfnovo/open-notebook](https://github.com/lfnovo/open-notebook))는 오픈소스 AI 연구 어시스턴트 겸 지식 관리 도구입니다.
NotebookLM을 대체하는 로컬 first RAG 시스템으로, 나만의 지식 베이스를 구축하고 AI와 대화하듯 검색할 수 있습니다.

이 레포지토리는 **Hermes Agent**가 Open Notebook의 REST API를 완벽히 활용할 수 있도록 설계된 **AI Agent Skill 컬렉션**입니다.

---

## 📦 포함된 스킬

### 1. `open-notebook` (English)
Hermes Agent가 Open Notebook REST API와 프로그래매틱하게 상호작용하기 위한 영문 가이드입니다.

| 기능 | 설명 |
|------|------|
| 🔍 **노트북 검색** | 키워드 기반 knowledge base 검색 |
| 💬 **RAG Q&A** | 3-step chat workflow (세션 생성 → 컨텍스트 빌드 → 질문 실행) |
| 📄 **소스 관리** | 웹페이지, 파일, YouTube, PDF 등 다양한 소스 추가/조회 |
| 📝 **노트 관리** | 노트 생성 및 조회 |
| 🎙️ **팟캐스트 생성** | NotebookLM 스타일 오디오 팟캐스트 자동 생성 |
| 🔄 **변환 실행** | 임베딩 리빌드 등 변환 워크플로우 |
| ⚙️ **모델 설정** | 공급자/모델 조회, 자동 할당, 설정 변경 |
| 🔗 **MCP 통합** | Hermes native MCP 클라이언트 설정 (`uvx open-notebook-mcp`) |

### 2. `open-notebook-kr-guide` (한국어)
Open Notebook REST API에 대한 상세한 한국어 가이드입니다.

| 기능 | 설명 |
|------|------|
| 🔍 **노트북 검색** | `/api/search` 엔드포인트로 키워드 검색 |
| 💬 **3단계 RAG 채팅** | 세션 생성 → 컨텍스트 구성 → AI 질의 (진짜 RAG 워크플로우) |
| 📄 **소스 CRUD** | 소스 추가 (`type`, `url` 필드 필수), 목록 조회 |
| 📝 **노트 CRUD** | 노트 생성 (`notebook_id`, `title`, `content`) |
| ⚙️ **설정 관리** | config 조회, 모델 목록, 공급자 확인, 설정 변경 |
| 🎙️ **팟캐스트 생성** | 노트북 콘텐츠 기반 오디오 팟캐스트 자동 생성 |
| 🔗 **MCP 연동** | Hermes config.yaml에 Open Notebook MCP 등록 방법 |

---

## 🚀 설치 방법

### Hermes Agent에 스킬 추가

```bash
# 1. ~/.hermes/skills/ 아래에 스킬 파일 복사
cp -r skills/* ~/.hermes/skills/

# 2. MCP 설정 (~/.hermes/config.yaml)
mcp_servers:
  open-notebook:
    command: uvx
    args: ["open-notebook-mcp"]
    env:
      OPEN_NOTEBOOK_URL: http://localhost:****/api/
```

### Open Notebook 실행 (Docker)

```bash
git clone https://github.com/lfnovo/open-notebook.git
cd open-notebook
docker compose up -d
```

> Web UI: `http://localhost:****`  
> API: `http://localhost:****/api/`  
> Swagger: `http://localhost:****/docs`

---

## ⚠️ 중요 주의사항

### 프록시 환경
SOCKS5 프록시(`ALL_PROXY=socks5://127.0.0.1:****`) 환경에서 localhost 호출 시 **반드시 `--noproxy '*'`** 를 curl에 포함해야 합니다. 포함하지 않으면 요청이 프록시를 통해 나가서 hang/fail이 발생합니다.

```bash
# ✅ 올바른 예시
curl -s --noproxy '*' -X POST http://localhost:****/api/search ...

# ❌ 잘못된 예시 (프록시를 타서 실패)
curl -s -X POST http://localhost:****/api/search ...
```

### API 경로
Swagger UI에는 `/api/v1/`로 표시되지만 **실제 경로는 `/api/`** 입니다.  
`/api/v1/` prefix는 모든 엔드포인트에서 404를 반환합니다.

### 인증
기본 설정으로 인증이 비활성화되어 있어 API 키 없이도 모든 요청이 가능합니다.

---

## 💡 사용 예시

### 노트북 검색
```bash
curl -s --noproxy '*' -X POST http://localhost:****/api/search \
  -H "Content-Type: application/json" \
  -d '{"query": "검색어", "notebook_id": "notebook:xxx"}'
```

### RAG 질문 (3단계)
```bash
# 1. 세션 생성
SESSION_ID=$(curl -s --noproxy '*' -X POST http://localhost:****/api/chat/sessions \
  -H "Content-Type: application/json" \
  -d '{"notebook_id": "notebook:xxx"}' | python3 -c "import sys,json; print(json.load(sys.stdin)['session_id'])")

# 2. 컨텍스트 빌드
CONTEXT=$(curl -s --noproxy '*' -X POST "http://localhost:****/api/notebooks/notebook:xxx/context" \
  -H "Content-Type: application/json" \
  -d '{"query": "질문 내용"}' | python3 -c "import sys,json; print(json.load(sys.stdin)['context'])")

# 3. AI 질문 실행
curl -s --noproxy '*' -X POST http://localhost:****/api/chat/execute \
  -H "Content-Type: application/json" \
  -d '{
    "session_id": "'"$SESSION_ID"'",
    "message": "질문 내용",
    "context": "'"$CONTEXT"'"
  }'
```

---

## 📁 파일 구조

```
open-notebook-skills/
├── README.md                    # 이 파일
├── LICENSE                      # Apache 2.0
└── skills/
    ├── open-notebook/
    │   └── SKILL.md             # 영문 AI Agent 가이드
    └── open-notebook-kr-guide/
        └── SKILL.md             # 한국어 API 활용 가이드
```

---

## 📜 라이선스

Apache 2.0 License

---

## 🔗 관련 링크

- [Open Notebook GitHub](https://github.com/lfnovo/open-notebook)
- [Hermes Agent](https://hermes-agent.nousresearch.com/)
