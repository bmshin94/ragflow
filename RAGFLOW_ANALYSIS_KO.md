# RAGFlow 전수조사 분석 리포트 (한국어)

> 작성: Claude Code 세션 / 정리일: 2026-10-03
> 분석 대상 포크: **https://github.com/bmshin94/ragflow**
> 원본 업스트림: **https://github.com/infiniflow/ragflow**
> 공식 문서: https://ragflow.io/docs/dev/ · 클라우드: https://cloud.ragflow.io
> 공식 Skill(OpenClaw): https://clawhub.ai/yingfeng/ragflow-skill

---

## 목차

1. [프로젝트 정체 확인](#1-프로젝트-정체-확인)
2. [한 줄 요약 / 쉬운 설명](#2-한-줄-요약--쉬운-설명)
3. [폴더별 전수조사 결과](#3-폴더별-전수조사-결과)
4. [데이터 처리 흐름 (5단계 공장 라인)](#4-데이터-처리-흐름-5단계-공장-라인)
5. [쓸 수 있는 상황 / 나에게 주는 도움](#5-쓸-수-있는-상황--나에게-주는-도움)
6. [설치 및 사용법](#6-설치-및-사용법)
7. [플러그인? 스킬? MCP? — 정체 정리](#7-플러그인-스킬-mcp--정체-정리)
8. [API 토큰 사용 여부](#8-api-토큰-사용-여부)
9. [AI 에이전트 구축에 도움이 되는가](#9-ai-에이전트-구축에-도움이-되는가)
10. [React / PHP로 만들 수 있는가](#10-react--php로-만들-수-있는가)
11. [유튜브 강의 영상 제작 가능성](#11-유튜브-강의-영상-제작-가능성)
12. [수익화 아이디어 7가지 (상세)](#12-수익화-아이디어-7가지-상세)
13. [수익화 로드맵 4단계](#13-수익화-로드맵-4단계)
14. [리스크 체크리스트](#14-리스크-체크리스트)
15. [부록: 빠른 참조표](#15-부록-빠른-참조표)

---

## 1. 프로젝트 정체 확인

| 항목 | 내용 |
|---|---|
| 이름 | **RAGFlow** |
| 원본 레포 | `infiniflow/ragflow` |
| 분석한 포크 | `bmshin94/ragflow` |
| 버전 | `v0.27.2` (`pyproject.toml`) |
| 라이선스 | **Apache License 2.0** (상업적 이용 가능) |
| 전체 파일 수 | 6,214개 |
| Python 파일 | 1,341개 |
| Go 파일 | 2,315개 |
| TS/TSX 파일 | 1,452개 (`web/src`) |
| 포크에서 추가된 변경 | `CLAUDE.md`, `GEMINI.md` 2개 문서만 (코드는 업스트림 원본 유지) |

### 포크 커밋 이력 확인 결과

```
f053b23 Update CLAUDE.md                              ← 포크 작업
88529c1 Add files via upload (CLAUDE.md, GEMINI.md)   ← 포크 작업
7438150 Delete CLAUDE.md                              ← 포크 작업
8f73b64 Merge PR #1: docs: add CLAUDE.md project guide ← 포크 작업
563a108 fix: resolve provider aliases for extractor models (#19824)  ← 업스트림
```

→ 코드 변경 없음. **업스트림 추적이 쉬운 깨끗한 상태.**

---

## 2. 한 줄 요약 / 쉬운 설명

### 한 줄 요약

> **"내 문서(PDF·엑셀·워드·스캔본)를 AI가 진짜로 이해하게 만들고, 출처까지 콕 찍어서 답해주는 엔터프라이즈급 RAG + AI 에이전트 플랫폼"**

### 비유: "AI 전용 도서관 사서 로봇"

회사에 종이 박스 500개(계약서·보고서·엑셀·스캔 영수증)가 있고,
사장님이 "작년 3분기 A사 계약서 위반 조항이 뭐였지?" 라고 물었을 때:

| 방법 | 결과 |
|---|---|
| 사람이 직접 찾기 | 3일 소요 |
| ChatGPT에 질문 | "모릅니다" |
| PDF 복붙해서 질문 | 토큰 초과 + 표 깨짐 + 환각 |
| **RAGFlow** | **3초 → "제2조 3항" + 해당 페이지 하이라이트** |

### RAG란?

**R**etrieval(검색) **A**ugmented(증강) **G**eneration(생성)

- AI만 사용 → "시험 범위를 안 배운 학생" → 아는 척 지어냄(환각)
- RAG 사용 → "오픈북 시험 보는 학생" → 교재에서 찾아 답하고 페이지도 알려줌

**AI를 똑똑하게 만드는 게 아니라, AI에게 교재를 펼쳐주는 기술.**

---

## 3. 폴더별 전수조사 결과

### 3.1 핵심 엔진 4대장

#### `deepdoc/` — RAGFlow의 진짜 무기 (문서를 "보고" 이해)

```
deepdoc/parser/
  pdf_parser.py, docx_parser.py, excel_parser.py, ppt_parser.py,
  epub_parser.py, html_parser.py, markdown_parser.py, json_parser.py,
  txt_parser.py, figure_parser.py
  + 특수 OCR 파서: mineru_parser.py, docling_parser.py, paddleocr_parser.py,
    mistral_parser.py, monkeyocrv2_parser.py, somark_parser.py,
    tcadp_parser.py, opendataloader_parser.py
  + resume/ (이력서 전용 파서)

deepdoc/vision/
  ocr.py                       → 글자 인식
  layout_recognizer.py         → 제목/본문/표/그림 레이아웃 영역 판단
  table_structure_recognizer.py→ 표 구조(병합셀 포함) 복원
  recognizer.py, operators.py, postprocess.py, seeit.py

deepdoc/server/               → 파싱 서비스
```

**왜 중요한가:** 일반 PDF 추출 툴은 표를 평문으로 뭉개버린다.

```
원본 표:              일반 툴 결과:
┌──────┬──────┐       "제품 가격 사과 1000 바나나 2000"
│ 제품 │ 가격 │   →   (표 구조 소실 → 무엇이 어떤 가격인지 알 수 없음)
├──────┼──────┤
│ 사과 │ 1000 │       DeepDoc 결과:
│바나나│ 2000 │   →   사과 = 1000, 바나나 = 2000 (정확)
└──────┴──────┘
```

#### `rag/` — 검색·청킹·LLM 통합

```
rag/app/       → 문서 타입별 청킹 템플릿 14종
                 naive, book, laws, manual, paper, presentation, qa,
                 resume, table, picture, audio, email, one, tag
rag/llm/       → chat_model, embedding_model, rerank_model, ocr_model,
                 cv_model, tts_model, sequence2txt_model,
                 tool_decorator, model_meta, key_utils
rag/graphrag/  → GraphRAG (지식 그래프 검색)
                 general/, light/, ner/, entity_resolution.py,
                 search.py, checkpoints.py, query_analyze_prompt.py
rag/advanced_rag/ → 고급 검색 전략
rag/nlp/       → 다국어 토크나이징·쿼리 처리
rag/flow/      → 파이프라인 플로우
rag/prompts/   → 프롬프트 템플릿
rag/svr/       → 백그라운드 워커(태스크 실행)
rag/benchmark.py → 벤치마크
```

**청킹 템플릿 비교표**

| 템플릿 | 자르는 기준 | 적용 대상 |
|---|---|---|
| `naive` | 일반 (기본값) | 범용 문서 |
| `laws` | 조/항/목 단위 | 법률·규정·약관 |
| `paper` | 초록/서론/결론 섹션 | 논문 |
| `book` | 장/절 단위 | 책 |
| `manual` | 목차 구조 | 제품 매뉴얼 |
| `qa` | Q·A 쌍 | FAQ |
| `table` | 행 단위 | 엑셀·CSV |
| `resume` | 학력/경력/스킬 항목 | 이력서 |
| `presentation` | 슬라이드 단위 | PPT |
| `picture` | 이미지 → 멀티모달 분석 | 사진·도표 |
| `audio` | 음성 → 텍스트 | 녹음 파일 |
| `email` | 메일 단위 | 메일함 |
| `one` | 문서 전체 1덩어리 | 짧은 문서 |
| `tag` | 태그 추출 | 분류 보조 |

> 쪼갠 결과를 웹 UI(`web/src/pages/chunk`)에서 **눈으로 보고 손으로 수정 가능** —
> 블랙박스가 아니라는 점이 RAGFlow의 핵심 차별점.

#### `agent/` — AI 에이전트 워크플로우 엔진

```
agent/canvas.py        → 워크플로우 실행 엔진
agent/dsl_migration.py → DSL 마이그레이션
agent/sandbox/         → gVisor 격리 코드 실행 환경
agent/plugin/          → 플러그인 매니저, embedded_plugins, llm_tool_plugin

agent/component/ (22개 블록)
  llm, agent_with_tools, categorize, switch, iteration, iterationitem,
  loop, loopitem, exit_loop, variable_assigner, variable_aggregator,
  string_transform, excel_processor, docs_generator, data_operations,
  list_operations, browser, invoke, message, fillup, begin, base

agent/tools/ (27개 외부 툴)
  검색: tavily, google, duckduckgo, searxng, youcom, wikipedia
  학술: arxiv, pubmed, googlescholar
  금융: yahoofinance, akshare, tushare, wencai, jin10
  실행: code_exec(샌드박스), exesql(SQL)
  기타: crawler, email, github, deepl, qweather, retrieval,
        bgpt, keenable, querit, sofya, base

agent/templates/ (19개 완성품 템플릿)
  deep_research, seo_article_writer, market_seo_article_writer,
  text2sql_data_expert, smart_customer_service_specialist,
  customer_feedback_dispatcher, stock_market_research_assistant,
  reflective_academic_paper_generator, cajal_scientific_paper_agent,
  trip_planner, photo_text_translator, data_analysis_beginner_assistant,
  advanced_ingestion_pipeline, chunk_summary, title_chunker,
  your_starter_dataset_chatbot, web_search_assistant,
  compiler, user_interaction
```

#### `internal/` — Go 재작성 레이어 (25개 서브패키지)

```
internal/ingestion/       → Go 인제스션 파이프라인 (component/, pipeline/)
internal/parser/          → Go 파서 + chunk 연산자 라이브러리
internal/deepdoc/         → C++/CGO 네이티브 PDF·DOCX 파싱
internal/cpp/             → C++ 소스
internal/rag/agentic-rag/ → 에이전틱 RAG (인용 정밀화)
internal/engine/          → Elasticsearch / Infinity 검색 백엔드
internal/agent/           → Go 에이전트 런타임
internal/handler/         → HTTP 핸들러
internal/router/          → 라우트 등록
internal/service/         → 비즈니스 서비스
internal/dao/             → 데이터 액세스
internal/entity/          → 엔티티/모델 정의
internal/storage/         → 스토리지 백엔드
internal/channels/        → 메신저 연동 Go 구현
internal/cli/             → CLI
internal/mcp/             → Go MCP
internal/server/          → 서버 부트스트랩
internal/syncer/          → 동기화
internal/tokenizer/       → 토크나이저
internal/admin/, binding/, common/, harness/, utility/
internal/development.md   → Go 개발 가이드 (빌드 문제 시 필독)
```

> **성능 핵심부를 Python → Go + C++로 재작성 중.** 현재 Python/Go 듀얼 트랙 상태.
> 수정 시 어느 경로가 실제로 실행되는지 반드시 확인 필요.

### 3.2 연동·확장 레이어

#### `common/data_source/` — 외부 데이터 커넥터 40개+

| 분류 | 커넥터 |
|---|---|
| 클라우드 스토리지 | google_drive, dropbox, onedrive, box, blob, azure_blob, webdav, seafile |
| 협업 도구 | notion, confluence, slack, discord, teams, feishu_wiki, dingtalk_ai_table, asana, airtable, moodle |
| 코드 저장소 | github, gitlab, bitbucket, azure_devops, jira |
| 메일 | gmail, outlook, imap |
| CRM/지원 | salesforce, zendesk |
| DB/기타 | rdbms, bigquery, rest_api, rss, sitemap, xquik |
| 보안 | ssrf_guard.py (SSRF 방어) |

#### `api/apps/restful_apis/` — REST API 29개 모듈

```
dataset_api, document_api, chunk_api, file_api, file2document_api,
file_commit_api, chat_api, agent_api, search_api, memory_api,
mcp_api, plugin_api, connector_api, models_api, provider_api,
tenant_api, user_api, task_api, stats_api, system_api,
chat_channel_api, bot_api, langfuse_api, aimlapi_api,
compilation_template_api, compilation_template_group_api,
openai_api        ← OpenAI 호환 엔드포인트
dify_retrieval_api← Dify 연동
```

#### `mcp/` — MCP 서버 + 클라이언트 양방향

```
mcp/server/server.py                 → RAGFlow를 MCP 서버로 노출
                                       (기본 포트 9382, self-host / host 모드)
                                       (transport: sse / streamable-http)
mcp/client/client.py                 → 외부 MCP 서버를 툴로 가져오기
mcp/client/streamable_http_client.py

docs/develop/mcp/
  overview.md, launch_mcp_server.md, use_ragflow_as_mcp_server.md,
  connect_an_external_mcp_to_ragflow.md, mcp_tools.md,
  mcp_client_example.md, native_go_mcp.md
```

#### 메신저 채널 (`api/channels/` + `internal/channels/`)

```
feishu, discord, telegram, line, wecom, dingtalk, whatsapp, qqbot
```

#### `memory/` — 에이전트 장기 기억

```
memory/services/  messages.py, query.py
memory/utils/     es_conn, infinity_conn, ob_conn, gaussdb_conn,
                  prompt_util, aggregation_utils, msg_util, highlight_utils
```

#### `sdk/python/ragflow_sdk/modules/`

```
dataset.py, document.py, chunk.py, chat.py, session.py, agent.py, memory.py
```

### 3.3 프론트엔드 `web/`

```
스택: React + TypeScript + Vite + TailwindCSS + Radix UI
      + @antv/x6 (워크플로우 캔버스)
      + @antv/g6 (지식그래프 시각화)
      + @antv/g2 (차트)
      + Monaco Editor (코드 에디터)
      + Lexical (리치 텍스트 에디터)
      + react-hook-form + zod
      + Storybook + Jest
      + oxlint / oxfmt

web/src/pages/
  datasets, dataset, document-viewer, chunk   → 지식베이스 관리
  agents, agent                               → 비주얼 워크플로우 캔버스
  next-chats, next-searches, next-search      → 채팅/검색 UI
  memories, memory                            → 기억 관리
  dataflow-result, files, home, skills, admin, login-next, user-setting

web/CLAUDE.md → 프론트엔드 컨벤션 (작업 전 필독)
```

### 3.4 인프라 `docker/`

| 분류 | 서비스 |
|---|---|
| 검색엔진 (택1) | elasticsearch / opensearch / infinity / serenedb |
| 메타DB (택1) | mysql / gaussdb(postgres) / oceanbase / seekdb |
| 객체 스토리지 | minio |
| 캐시/큐 | redis / kvrocks / nats |
| 임베딩 서버 | tei-cpu / tei-gpu (HuggingFace TEI) |
| 관측 | jaeger, kibana, clickhouse |
| 샌드박스 | sandbox-executor-manager (gVisor) |

```
docker/docker-compose.yml         → 메인 (ragflow-cpu / ragflow-gpu)
docker/docker-compose-base.yml    → 인프라만
docker/docker-compose-go.yml      → Go 버전
docker/docker-compose-macos.yml   → macOS
docker/service_conf.yaml.template → 서비스 설정 (LLM 기본값·API 키)
docker/.env                       → 환경 변수
```

> **환경 변수 조합:** `COMPOSE_PROFILES=${DOC_ENGINE},${DEVICE},metadata-${METADATA_DB_PROFILE}`
> → `DOC_ENGINE`(elasticsearch 기본) / `DEVICE`(cpu 기본) / `DB_TYPE`(mysql 기본)

### 3.5 LLM 프로바이더 — 78개 (`conf/llm_factories.json`)

| 분류 | 프로바이더 |
|---|---|
| 해외 메이저 | OpenAI, Anthropic, Gemini, Groq, Mistral, Cohere, xAI, Bedrock, Azure-OpenAI, OpenRouter, TogetherAI, Perplexity, DeepInfra, Replicate, NVIDIA, Voyage AI, Upstage, Google Cloud, HuggingFace, Jina |
| 중국 | DeepSeek, Tongyi-Qianwen, ZHIPU-AI, Moonshot, VolcEngine, BaiChuan, MiniMax, Tencent Hunyuan, XunFei Spark, BaiduYiyan, Tencent Cloud, Xiaomi, ModelScope, SILICONFLOW, PPIO, GiteeAI, LongCat |
| 로컬/셀프호스팅 | **Ollama**, VLLM, LocalAI, LM-Studio, Xinference, GPUStack, FastEmbed |
| OCR 전용 | MinerU, PaddleOCR, Mistral OCR, SoMark, OpenDataLoader |
| 음성 | FunASR, Fish Audio |
| 범용 호환 | **OpenAI-API-Compatible**, New API |
| 기타 애그리게이터 | 302.AI, CometAPI, DeerAPI, Jiekou.AI, aimlapi.com, AnonRouter, API-Route, Astraflow, Avian, n1n, FuturMix, Synthorai, GreenPT, TokenPony, MWS, DaoXE, llmman, Hubris, RAGcon, Cheaper Inference, Builtin |

### 3.6 기타 디렉터리

```
api/           → Python API 서버 (Quart) : apps, channels, common, db, utils
                 ragflow_server.py (엔트리포인트)
api/db/services/ → 30개+ 서비스 (tenant_*, dialog, document, task, memory ...)
cmd/           → Go 엔트리포인트 (ragflow_server.go, ragflow-cli.go)
admin/         → 관리 서버 + 클라이언트 + CLI 릴리스 빌드
mcp/           → MCP 서버/클라이언트
tools/         → chatgpt-on-wechat, firecrawl, migrate-canvas,
                 es-to-oceanbase-migration, gen-component-parity,
                 render_diff, hooks, scripts
example/       → http/ (curl 샘플 6개), sdk/ (Python 샘플 5개),
                 chat_demo/ (임베드 위젯 HTML)
docs/          → quickstart, guides/, develop/, references/,
                 administrator/, basics/, release_notes.md
test/, sdk/python/test/ → 자동화 테스트
helm/          → Kubernetes Helm 차트
conf/          → llm_factories.json, 매핑 설정, system_settings.json
build.sh       → Go/네이티브 빌드 + 테스트 러너 (필수 사용)
run_tests.py, run_go_tests.sh, lefthook.yml
```

---

## 4. 데이터 처리 흐름 (5단계 공장 라인)

```
【1】 받기 (Ingestion)
     파일 업로드 또는 커넥터 동기화 (common/data_source/*)
     → MinIO 원본 보관
            ↓
【2】 읽기 (DeepDoc)
     deepdoc/parser/pdf_parser.py
     + deepdoc/vision/ocr.py                       (글자 인식)
     + deepdoc/vision/layout_recognizer.py         (영역 판단)
     + deepdoc/vision/table_structure_recognizer.py(표 복원)
            ↓
【3】 쪼개기 (Chunking)
     rag/app/naive.py | laws.py | paper.py | table.py ... (14종 템플릿)
     → 웹 UI에서 사람이 눈으로 확인하고 수정 가능 (web/src/pages/chunk)
            ↓
【4】 숫자로 저장 (Embedding & Index)
     rag/llm/embedding_model.py
     → Elasticsearch / Infinity / OpenSearch / SereneDB
     (옵션) rag/graphrag/ → 지식 그래프 동시 구축
            ↓
【5】 답하기 (Retrieval & Generation)
     ① 키워드 검색 + ② 벡터 검색 = 하이브리드 멀티리콜
     → ③ rag/llm/rerank_model.py (재순위, TOP-K 선별)
     → ④ rag/llm/chat_model.py (LLM 답변 생성)
     → ⑤ internal/rag/agentic-rag/ (인용 정밀 매칭)
     → 답변 + 출처 번호 → 클릭하면 원본 PDF 해당 위치로 점프
            ↓
【출구】 웹 UI / REST API / OpenAI호환 API / MCP /
        메신저봇(8종) / 임베드 위젯 / Python SDK / Dify
```

### GraphRAG가 추가로 해결하는 것

```
일반 벡터 검색: "A사 계약 조건?" → 답변 가능
GraphRAG:      "A사와 분쟁 중인 회사?" → 답변 가능 (관계 추론)

문서에서 추출한 관계 그래프:
  [김대표] ──대표이사── [A사] ──계약── [B사]
                         │
                      소송중
                         │
                       [C사]
```

---

## 5. 쓸 수 있는 상황 / 나에게 주는 도움

### 5.1 사용 시나리오

| 상황 | RAGFlow가 해주는 일 | 관련 기능 |
|---|---|---|
| 사내 문서 수천 장 | 질문 한 번에 정확한 답 + 출처 페이지 | 지식베이스 + 하이브리드 검색 |
| 법률/규정 검토 | 조항 단위 정밀 검색 | `laws` 청킹 템플릿 |
| 복잡한 표 PDF | 표 구조 복원 후 숫자까지 정확히 읽기 | DeepDoc `table_structure_recognizer` |
| 논문 리서치 | 멀티소스 리서치 보고서 자동 생성 | `deep_research` + arxiv/pubmed 툴 |
| 고객지원 챗봇 | 상담 봇 생성 + 메신저 배포 | `smart_customer_service_specialist` + 채널 |
| 자연어로 DB 질의 | "매출 top10" → SQL 실행 → 답변 | `text2sql_data_expert` + `exesql` |
| 이력서 대량 심사 | 학력/경력/스킬 항목 추출 | `resume` 파서 + 템플릿 |
| 흩어진 데이터 통합 | 노션+슬랙+드라이브 한 번에 검색 | 커넥터 40개+ |
| AI 에이전트 지식 레이어 | Claude/GPT에 내 지식 공급 | MCP 서버 / REST API |

### 5.2 나에게 주는 도움 (6가지)

1. **"AI 거짓말 제거" 장치를 무료로 획득**
   출처 하이라이트까지 제공 → "AI가 지어낸 것이 아님"을 증명 가능.
   기업 납품에서 가장 중요한 요건 (감사/Audit 대응).

2. **Apache 2.0 = 상업적 이용 합법**
   포크 → 커스터마이징 → 리브랜딩 → 고객사 납품 / SaaS 운영 전부 가능.
   (GPL이 아니므로 **수정 소스 공개 의무 없음**)

3. **"RAG 교과서" 그 자체**
   프로덕션급 정답이 전부 들어있음:
   문서 파싱 전략(18개 파서) / 청킹 전략(14개 템플릿) /
   하이브리드 검색 + 리랭킹 / GraphRAG /
   에이전트 오케스트레이션 / 멀티테넌시 + 과금 구조

4. **AI 에이전트 백엔드로 바로 사용**
   `mcp/server/server.py` → Claude Code/Desktop에 연결 → "내 문서 아는 Claude"

5. **포트폴리오·강의 소재 최상급**
   대형 오픈소스 + Go/Python/React/C++ 전부 + 최신 트렌드(RAG·에이전트·MCP) 커버

6. **40+ 커넥터로 데이터 통합 문제 선해결**
   개별 API 연동에 수 주~수 개월 걸리는 작업이 이미 완료 상태

---

## 6. 설치 및 사용법

### 6.1 방법 A — 클라우드 (설치 0분)

```
https://cloud.ragflow.io 접속 → 회원가입 → 즉시 사용
```
- 장점: 설치 불필요, 체험에 최적
- 단점: 내 문서가 외부 서버로 전송됨

### 6.2 방법 B — Docker 셀프호스팅 (권장)

**시스템 요건**

| 항목 | 최소 | 권장 |
|---|---|---|
| CPU | 4코어 | 8코어 |
| RAM | 16GB | **32GB** |
| 디스크 | 50GB | 100GB SSD |
| Docker | 24.0.0+ | 최신 |
| Docker Compose | v2.26.1+ | 최신 |
| Python (소스 빌드 시) | 3.13+ | 3.13+ |
| 아키텍처 | **x86_64만 공식 지원** | ARM64는 직접 빌드 필요 |
| gVisor | 코드 실행(sandbox) 기능 사용 시만 | - |

**① 커널 파라미터 설정 (Elasticsearch 필수)**

```bash
sysctl vm.max_map_count            # 262144 이상 확인
sudo sysctl -w vm.max_map_count=262144

# 영구 적용
echo "vm.max_map_count=262144" | sudo tee -a /etc/sysctl.conf
```

**② 클론**

```bash
git clone https://github.com/bmshin94/ragflow.git
cd ragflow
```

**③ 실행**

```bash
cd docker

# CPU 버전
docker compose -f docker-compose.yml up -d

# GPU 가속 (DeepDoc OCR 속도 향상)
sed -i '1i DEVICE=gpu' .env
docker compose -f docker-compose.yml up -d
```

**④ 로그 확인 — ASCII 아트가 뜨면 성공**

```bash
docker logs -f docker-ragflow-cpu-1
```

```
      ____   ___    ______ ______ __
     / __ \ /   |  / ____// ____// /____  _      __
    / /_/ // /| | / / __ / /_   / // __ \| | /| / /
   / _, _// ___ |/ /_/ // __/  / // /_/ /| |/ |/ /
  /_/ |_|/_/  |_|\____//_/    /_/ \____/ |__/|__/

 * Running on all addresses (0.0.0.0)
```

> 이 메시지가 뜨기 전에 접속하면 `network abnormal` 에러가 발생한다.

**⑤ 브라우저 접속**

```
http://서버IP          (기본 HTTP 포트 80이므로 포트 생략 가능)
```

**⑥ LLM API 키 등록**

```
우측 상단 아바타 → Model providers → 프로바이더 선택 → API 키 입력
```
또는 `docker/service_conf.yaml.template`의 `user_default_llm` + `API_KEY` 설정

### 6.3 방법 C — 소스 빌드 (개발자용)

**백엔드**

```bash
uv sync --python 3.13 --all-extras
uv run python3 ragflow_deps/download_deps.py
docker compose -f docker/docker-compose-base.yml up -d   # 인프라만 기동
source .venv/bin/activate
export PYTHONPATH=$(pwd)
bash docker/launch_backend_service.sh

# 검증
uv run pytest
ruff check
ruff format
```

**프론트엔드**

```bash
cd web
npm install
npm run dev          # 개발 서버
npm run build        # 프로덕션 빌드
npm run lint         # oxlint
npm run test         # jest
npm run type-check   # tsc --noEmit
```

**Go (주의 필요)**

```bash
uv run ragflow_deps/download_deps.py
bash build.sh --go                        # Go 빌드
bash build.sh --all                        # 전체 바이너리

# 테스트 티어
bash build.sh --test                       # unit (태그 없음)
bash build.sh --test ./internal/parser/... # 패키지 한정
bash build.sh --test-integration ./...     # integration 태그
bash build.sh --test-e2e                   # e2e 태그
bash build.sh --test-manual                # manual 태그 (매우 느림, CI 금지)
bash build.sh --test-all                   # integration + e2e
```

> **중요:** 생짜 `go test` / `go build` 금지.
> CGO 플래그 + 네이티브 정적 라이브러리(`pdfium`, `office_oxide`, `pdf_oxide`)가
> `build.sh`에서만 올바르게 연결된다.
> 빌드 실패 시 `build.sh`와 `internal/development.md` 확인.
> Linux에서는 `lld` 설치 여부가 흔한 원인.

**Go 테스트 티어 규칙**

| 티어 | 빌드 태그 | 기본 실행 | 필요 조건 |
|---|---|---|---|
| Unit | (없음) | 예 | 네이티브 CGO 정적 라이브러리. 외부 서비스 불필요 (SQLite/miniredis/httptest) |
| Integration | `integration` | 아니오 | 실제 서비스 1종 (MySQL/MinIO/ES/Infinity/LLM) |
| E2E | `e2e` | 아니오 | 전체 파이프라인 + 실제 서비스 |
| Manual | `manual` | 아니오 | 매우 느림. **로컬 전용, CI 금지** |
| Native | `cgo` / `!cgo` | CGO_ENABLED=1에서 자동 | 네이티브 정적 라이브러리 |

### 6.4 첫 사용 5분 튜토리얼

```
1. 로그인
2. [지식베이스] → 생성 → 이름 입력
3. 청킹 방법 선택 (일반 문서는 General/naive)
4. 임베딩 모델 선택
5. 파일 업로드 → [파싱 시작]
6. 파싱 완료 후 청크 확인 → 이상한 부분은 직접 수정
7. [채팅] → 어시스턴트 생성 → 지식베이스 연결
8. 질문 → 답변 + 출처 번호 클릭 → 원본 PDF 위치로 점프
```

---

## 7. 플러그인? 스킬? MCP? — 정체 정리

### 결론: **세 가지 모두 아니다. 독립 플랫폼(완제품 서버)이다.**
### 다만 세 가지 방식 모두로 연결은 가능하다.

| 분류 | RAGFlow는? | 설명 |
|---|---|---|
| **본질** | 독립 플랫폼 | Docker로 기동하는 풀스택 웹 서버. DB·벡터DB·스토리지·프론트엔드 전부 포함 |
| **MCP 서버** | 가능 | `mcp/server/server.py` → Claude Desktop/Code에서 내 지식베이스 조회 |
| **MCP 클라이언트** | 가능 | `mcp/client/` → 외부 MCP 서버 툴을 RAGFlow 에이전트에 추가 |
| **Skill** | 별도 존재 | 공식 `ragflow-skill` (OpenClaw / clawhub.ai) — RAGFlow를 호출하는 래퍼 |
| **플러그인 호스트** | 가능 | `agent/plugin/` — RAGFlow **안에** 플러그인을 꽂는 반대 방향 |
| **OpenAI 호환 API** | 가능 | `api/apps/restful_apis/openai_api.py` → base_url 교체만으로 드롭인 |
| **Dify 연동** | 가능 | `dify_retrieval_api.py` |

### 연결 구조도

```
                    ┌─────────────────────────┐
  Claude Code ◄─MCP─┤                         │
  Claude Desktop    │       RAGFlow           │
                    │   (독립 플랫폼/서버)      │
  내 웹서비스 ◄─API──┤                         │
                    │  ┌───────────────────┐  │
  메신저봇    ◄─채널─┤  │ 지식베이스        │  │
                    │  │ 에이전트 캔버스    │  │
  OpenAI SDK ◄─호환─┤  │ 검색엔진          │  │
                    │  └───────────────────┘  │
                    │          ▲              │
                    └──────────┼──────────────┘
                               │
                   ┌───────────┴───────────┐
          외부 MCP 서버          agent/plugin/
          (툴로 가져옴)          (플러그인 꽂기)
```

### MCP 연결 방법

**① RAGFlow를 MCP 서버로 기동**

```bash
# docker/.env 에서 MCP 활성화 후
python mcp/server/server.py --mode=self-host
# 기본: HOST 127.0.0.1, PORT 9382
# transport: sse 또는 streamable-http (둘 다 활성)
# BASE_URL 기본값: http://127.0.0.1:9380
```

**② Claude Code / Desktop에 등록**

```json
{
  "mcpServers": {
    "ragflow": {
      "url": "http://localhost:9382/mcp",
      "headers": { "Authorization": "Bearer <RAGFLOW_API_KEY>" }
    }
  }
}
```

**③ 반대로 외부 MCP를 RAGFlow에 추가**

```
아바타 → User settings → MCP → Add MCP
  Name:        Local file tools
  URL:         https://example.com/mcp
  Server type: streamable-http   (또는 sse)
```

> `stdio` 방식 MCP 서버는 직접 연결 불가.
> SSE 또는 Streamable HTTP 게이트웨이로 감싸야 한다.
> Streamable HTTP가 권장되며, SSE는 구형 호환용으로 유지된다.

---

## 8. API 토큰 사용 여부

### 결론: **두 종류의 키가 있고 용도가 완전히 다르다.**

### 8.1 RAGFlow API Key — 내가 발급하는 키

**용도:** 외부에서 RAGFlow를 조작할 때 (REST API, Python SDK, MCP)

**발급 방법**

```
우측 상단 아바타 클릭 → API 탭 → 키 생성
```
(공식 문서: `docs/develop/acquire_ragflow_api_key.md`)

**사용 예시 — HTTP**

```bash
curl --request POST \
  --url http://localhost:9380/api/v1/retrieval \
  --header 'Content-Type: application/json' \
  --header 'Authorization: Bearer <YOUR_API_KEY>' \
  --data '{
    "question": "계약 위반 조항?",
    "dataset_ids": ["<DATASET_ID>"],
    "top_k": 5
  }'
```

**사용 예시 — Python SDK**

```python
from ragflow_sdk import RAGFlow

rag = RAGFlow(api_key="<YOUR_API_KEY>", base_url="http://localhost:9380")

ds = rag.create_dataset(name="내문서")
ds.upload_documents([{
    "display_name": "계약서.pdf",
    "blob": open("계약서.pdf", "rb").read(),
}])
```

**필요 여부 표**

| 사용 방식 | RAGFlow API Key 필요? |
|---|---|
| 웹 UI로만 사용 | 불필요 (로그인으로 충분) |
| REST API 호출 | **필수** |
| Python SDK | **필수** |
| MCP 연결 | **필수** |
| 임베드 위젯 | **필수** (공유 토큰) |

### 8.2 LLM 프로바이더 API Key — 외부 업체 키

**용도:** RAGFlow가 AI 모델을 호출할 때

| 역할 | 예시 | 필수 여부 |
|---|---|---|
| 채팅 모델 | OpenAI, Anthropic, DeepSeek, Gemini | **필수** |
| 임베딩 모델 | OpenAI embedding, Jina, Voyage, BGE, FastEmbed | **필수** |
| 재순위(rerank) 모델 | Cohere rerank, Jina rerank | 선택 (품질 향상) |
| 멀티모달/OCR | GPT vision, MinerU, Mistral OCR, PaddleOCR | 선택 |
| 음성 (STT/TTS) | Whisper, FunASR, Fish Audio | 선택 |

### 8.3 외부 키 없이 완전 무료로 쓰는 방법

```
1. Ollama 설치 → ollama pull qwen2.5:7b
2. RAGFlow에서 프로바이더 "Ollama" 추가
   → http://host.docker.internal:11434
3. 임베딩은 FastEmbed 또는 TEI (docker-compose에 tei-cpu/tei-gpu 내장)

결과:
  · API 비용 0원
  · 데이터 100% 로컬 (외부 전송 없음)
  · 인터넷 연결 불필요 (완전 폐쇄망 가능)
```

> 이 구성이 금융·의료·공공 납품의 핵심 무기다. (12번 수익화 아이디어 1번 참고)

### 8.4 보안 체크리스트

```
[ ] API 키를 프론트엔드 코드에 넣지 않는다 (브라우저에서 노출됨)
[ ] 반드시 백엔드 프록시를 경유해 호출한다
[ ] docker/.env 가 .gitignore에 포함되어 있는지 확인 (레포에 이미 적용됨)
[ ] 기본 비밀번호 전부 교체 (infini_rag_flow 등 그대로 사용 금지)
    - ELASTIC_PASSWORD, OPENSEARCH_PASSWORD, SERENEDB_PASSWORD,
      OCEANBASE_PASSWORD, MinIO/Redis 자격증명
[ ] 외부 노출 시 HTTPS + 인증 레이어 필수
[ ] 커넥터 SSRF 보호는 common/ssrf_guard.py 가 담당 (확인만)
[ ] 코드 실행(sandbox) 기능 사용 시 gVisor 설치 확인
```

---

## 9. AI 에이전트 구축에 도움이 되는가

### 결론: **매우 도움이 된다. 사실상 "에이전트 공장"이다.**

### 활용 레벨 4단계

#### Level 1 — "지식 공급원"으로 사용 (가장 흔한 용도)

```
내가 만든 에이전트 (LangChain / CrewAI / Claude SDK 등)
         ↓ REST API 또는 MCP 호출
    RAGFlow (지식 검색 담당)
         ↓
    "출처가 붙은 정확한 사실" 반환
```
에이전트의 "뇌"는 내가 만들고, "기억·지식"은 RAGFlow가 담당.
RAG 파이프라인 직접 구축 시 2~3개월 소요되는 작업을 API 호출로 대체.

#### Level 2 — RAGFlow 안에서 에이전트 제작

```
비주얼 캔버스에서 드래그&드롭 → 22개 블록 + 27개 툴 조립
→ 코드 없이 멀티스텝 에이전트 완성
→ REST API / 메신저 / 임베드 위젯으로 즉시 배포
```
Dify / n8n / Flowise 대체 + RAG 품질 우위.

#### Level 3 — MCP 양방향 허브로 사용

```
[Claude Code] ──MCP──> [RAGFlow] ──MCP──> [외부 툴들]
                           │
                     내 지식베이스
```
RAGFlow가 "지식 + 툴 라우터" 역할. 에이전트 생태계의 허브.

#### Level 4 — 코드를 아키텍처 교과서로 사용

그대로 참고할 수 있는 프로덕션 패턴:

| 파일 | 배울 내용 |
|---|---|
| `agent/canvas.py` | 워크플로우 실행 엔진 설계 |
| `agent/component/base.py` | 컴포넌트 추상화 |
| `agent/component/agent_with_tools.py` | ReAct 툴 호출 루프 |
| `agent/tools/base.py` | 툴 인터페이스 표준 |
| `rag/llm/tool_decorator.py` | 함수 → LLM 툴 자동 변환 데코레이터 |
| `agent/sandbox/` | gVisor 안전 코드 실행 |
| `memory/services/` | 장기 기억 설계 |
| `rag/graphrag/` | GraphRAG 구현 |
| `internal/rag/agentic-rag/` | 에이전틱 RAG + 인용 정밀화 |
| `api/db/services/tenant_*` | 멀티테넌시 + 과금 구조 |
| `agent/dsl_migration.py` | DSL 버전 마이그레이션 |

### 실전 조합 레시피

| 만들 것 | 레시피 |
|---|---|
| 사내 Q&A 봇 | 지식베이스 + `smart_customer_service_specialist` + Slack 채널 |
| 리서치 에이전트 | `deep_research` + tavily + arxiv + 자체 논문 KB |
| 데이터 분석가 | `text2sql_data_expert` + `exesql` + `code_exec` + `excel_processor` |
| 콘텐츠 공장 | `seo_article_writer` + crawler + 자사 톤&매너 KB |
| 코드 리뷰 봇 | GitHub 커넥터 + 사내 코딩 규칙 KB + `agent_with_tools` |
| 법무 어시스턴트 | `laws` 청킹 + GraphRAG + 판례 KB |

### 경쟁 도구 비교

| 항목 | RAGFlow | LangChain | Dify | n8n |
|---|---|---|---|---|
| 문서 파싱 품질 | **최강** | 기본 | 보통 | 기본 |
| 비주얼 워크플로우 | 있음 | 없음 | 좋음 | 최고 |
| 코드 자유도 | 보통 | **최고** | 낮음 | 낮음 |
| 즉시 사용 가능 | **완제품** | 직접 조립 | 완제품 | 완제품 |
| 출처/인용 | **최강** | 직접 구현 | 있음 | 없음 |
| GraphRAG | **내장** | 직접 구현 | 없음 | 없음 |
| 리소스 요구 | 무거움 | 가벼움 | 중간 | 가벼움 |

> 문서 품질이 핵심인 에이전트에는 RAGFlow가 우위.
> 가벼운 챗봇에는 오버스펙일 수 있다.

---

## 10. React / PHP로 만들 수 있는가

### 10.1 React — 가능 (이미 React로 구현되어 있음)

`web/` 폴더가 이미 React + TypeScript + Vite 기반이다.

**선택지 3가지**

**① 기존 UI 커스터마이징 (가장 쉬움, 권장)**

```bash
cd web
npm install
npm run dev

# 수정 대상
web/src/pages/*     → 페이지
web/src/locales/*   → 다국어
web/src/assets/     → 로고·이미지
web/src/theme/      → 테마
```
> `web/CLAUDE.md`에 프론트엔드 컨벤션(듀얼 백엔드 Go/Python 변형, 테스트 배치,
> 스타일링, 데이터 페칭 규칙)이 있으므로 작업 전 필독.

**② 내 React 앱에서 RAGFlow를 백엔드로 사용**

```tsx
// API 키는 반드시 내 백엔드 프록시를 경유 (브라우저 노출 금지)
async function ask(question: string) {
  const res = await fetch('/api/my-proxy/ragflow-chat', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ question }),
  });
  return res.json();   // { answer, chunks: [{ content, document_name, page }] }
}
```

**③ 완전히 새 프론트엔드 제작**

RAGFlow는 REST API 서버이므로 Next.js / Remix / Vue / Svelte 등 자유 선택 가능.

### 10.2 PHP — 프론트/연동은 가능, 엔진 재작성은 비권장

**가능한 것**

```php
<?php
// Laravel 예시
use Illuminate\Support\Facades\Http;

$res = Http::withToken(config('services.ragflow.key'))
    ->post(config('services.ragflow.url').'/api/v1/retrieval', [
        'question'    => $request->input('q'),
        'dataset_ids' => [config('services.ragflow.dataset')],
        'top_k'       => 5,
    ]);

return response()->json([
    'answer'  => $res['data']['answer'] ?? null,
    'sources' => $res['data']['chunks'] ?? [],
]);
```

| 할 일 | 가능? | 비고 |
|---|---|---|
| RAGFlow API 호출 래퍼 | 가능 (쉬움) | Guzzle / Laravel Http |
| **워드프레스 플러그인** | 가능 | 수익화 포인트 |
| 라라벨 관리자 패널 | 가능 | 테넌트/과금 관리 |
| 그라브·드루팔 모듈 | 가능 | CMS 생태계 |
| 고객사 대시보드 | 가능 | 기존 PHP 시스템 연동 |
| OpenAI 호환 API 호출 | 가능 (쉬움) | `openai-php/client` 그대로 사용 |

**하지 말아야 할 것**

| 항목 | 이유 |
|---|---|
| DeepDoc 재구현 | PyTorch/ONNX 모델 필요. PHP 생태계 부재 |
| 벡터 검색 엔진 | ES/Infinity 네이티브 영역 |
| 임베딩 추론 | GPU 연산. PHP 불가 |
| 비동기 워커 파이프라인 | PHP의 약점 |

### 10.3 권장 아키텍처

```
┌──────────────────────────────────────────────┐
│ 프론트엔드: React 커스터마이징 또는 Next.js    │  ← 직접 제작
└────────────────────┬─────────────────────────┘
                     │ HTTPS
┌────────────────────▼─────────────────────────┐
│ BFF/프록시: PHP(Laravel) 또는 Node           │  ← 직접 제작
│  · API 키 보관    · 인증/권한                 │
│  · 과금/사용량    · 로깅/감사                 │
│  · 레이트 리밋                                │
└────────────────────┬─────────────────────────┘
                     │ Bearer <RAGFLOW_API_KEY>
┌────────────────────▼─────────────────────────┐
│ RAGFlow (Docker, 업스트림 그대로 사용)         │  ← 수정 최소화
│  DeepDoc · 벡터검색 · 에이전트 · LLM 라우팅    │
└──────────────────────────────────────────────┘
```

> **핵심 원칙: 엔진은 업스트림 그대로 쓰고, 그 위에 내 레이어만 쌓는다.**
> RAGFlow는 변경 속도가 매우 빨라서, 포크를 깊게 수정하면 머지 비용이 폭증한다.

---

## 11. 유튜브 강의 영상 제작 가능성

### 결론: 가능하며, 소재 가치가 매우 높다.

### 11.1 유튜브에 적합한 이유

| 이유 | 설명 |
|---|---|
| 핫한 키워드 | RAG, AI 에이전트, MCP |
| 비주얼이 좋음 | PDF 파싱 → 청크 하이라이트 → 캔버스 드래그 = 눈에 보이는 결과 |
| "와" 모먼트 확실 | "내 회사 문서로 챗봇 만들었다" → 즉시 공감 |
| 진입장벽 낮음 | 클라우드 체험판으로 설치 없이 시연 가능 |
| 한국어 콘텐츠 희박 | 블루오션, 선점 가능 |
| 법적 안전 | Apache 2.0 — 코드 공개·수정·상업적 이용 가능 |

### 11.2 시리즈 커리큘럼 (20편)

**시즌 1: 입문 (조회수 담당) — 5편**

| # | 제목 | 길이 | 포인트 |
|---|---|---|---|
| 1 | ChatGPT가 우리 회사 문서를 모르는 이유 (RAG 5분 정리) | 8분 | 훅 영상, 비개발자 대상 |
| 2 | 무료로 사내 문서 AI 챗봇 만들기 — RAGFlow 설치 | 15분 | Before/After 썸네일 |
| 3 | PDF 표가 깨지는 지옥 탈출 — DeepDoc 실험 | 12분 | **킬러 영상.** 타 툴과 비교 실험 |
| 4 | 청킹이 답변 품질의 90%다 — 템플릿 14종 비교 | 18분 | 실측 데이터 |
| 5 | AI 거짓말 잡는 방법 — 출처 인용 완전정복 | 10분 | 신뢰도 증명 |

**시즌 2: 실전 (구독 전환) — 6편**

| # | 제목 | 포인트 |
|---|---|---|
| 6 | 노션+슬랙+구글드라이브 전부 하나로 검색하기 | 커넥터 40개 쇼케이스 |
| 7 | 코딩 0줄로 AI 에이전트 만들기 — 캔버스 입문 | 드래그&드롭 비주얼 |
| 8 | 자연어로 DB 질문하기 — Text2SQL 에이전트 | `text2sql_data_expert` |
| 9 | 텔레그램/디스코드에 내 챗봇 5분 배포 | 즉각적 성취감 |
| 10 | GraphRAG — 벡터 검색이 못 푸는 질문들 | 차별화 콘텐츠 |
| 11 | API 비용 0원! Ollama 완전 로컬 RAG | 조회수 유리 |

**시즌 3: 고급/개발자 (전문성) — 5편**

| # | 제목 | 포인트 |
|---|---|---|
| 12 | Claude Code에 내 문서 연결하기 — MCP 완전정복 | 트렌디 |
| 13 | React로 RAGFlow UI 커스터마이징 | 프론트 개발자 타겟 |
| 14 | 내 서비스에 RAG 붙이기 — API/SDK 실전 | 코드 중심 |
| 15 | RAGFlow 소스 투어 — 프로덕션 RAG 아키텍처 | 고관여 시청자 |
| 16 | Python→Go 재작성 현장 — 성능 최적화 사례 | 심화 |

**시즌 4: 수익화/비즈니스 (유료 전환) — 4편**

| # | 제목 | 포인트 |
|---|---|---|
| 17 | 오픈소스로 월 500만원 — 폐쇄망 RAG 구축 사업 | 전환율 최고 |
| 18 | 버티컬 AI SaaS 만들기 (법무/의료 특화) | 창업 지향 |
| 19 | RAG 구축 견적서 쓰는 법 + 실제 계약 사례 | 실무 디테일 |
| 20 | RAGFlow vs Dify vs LangChain 끝장 비교 | 검색 유입 최강 |

### 11.3 제작 실전 팁

**알고리즘 공략**

```
· 훅: 첫 10초에 "결과물"부터 보여준다
  "이 PDF 500페이지를 3초만에 답했습니다" → 그 다음 본론
· 썸네일: Before(깨진 표) ↔ After(정확한 답) 분할 비교
· 제목: "RAGFlow 튜토리얼"(X) → "회사 문서 AI 챗봇 30분 완성"(O)
  (도구명보다 "문제/결과"가 검색된다)
· 챕터 타임스탬프 필수 (설치 구간 스킵 수요 많음)
· 쇼츠 분리: "PDF 표 파싱 비교" 30초 버전
```

**촬영 환경 주의**

```
· 설치 과정은 배속 + 편집 (실시간 20분은 이탈률 치명적)
· 메모리 16GB 이상 머신 권장 (녹화 중 버벅임 방지)
· API 키 화면 노출 금지 → 블러 처리 또는 더미 키 사용
· 예제 문서는 공개 자료 사용 (사내 문서 사용 시 유출 사고)
  추천: 정부 공개 보고서, arXiv 논문, 제품 매뉴얼
· 체험은 cloud.ragflow.io로 시작 → 설치 장벽 없는 영상 제작 가능
```

**법적 체크**

```
[ ] Apache 2.0 → 코드 공개/수정/상업적 이용 가능
[ ] 영상 설명란에 라이선스 + 원본 레포 링크 표기
    https://github.com/infiniflow/ragflow
[ ] 로고/브랜드는 "RAGFlow"로 명시 (자작 도구로 포장 금지)
[ ] 리브랜딩 제품 판매는 가능하되 NOTICE 유지
```

**수익 구조**

```
1차: 애드센스 (조회수)
2차: 유료 강의 (인프런/클래스101/자체) 20~30만원
3차: 영상 유입 구축 외주 문의 ← 단가 최고 (건당 수천만원)
4차: 기업 교육 의뢰 (사내 AI 교육 1일 300~800만원)
5차: 멤버십 (템플릿·에이전트 JSON 제공)
```

> **전략:** 유튜브를 "수익 채널"이 아니라 **"B2B 영업 채널"**로 본다.
> 조회수 10만보다 구축 문의 1건의 가치가 더 크다.

---

## 12. 수익화 아이디어 7가지 (상세)

### 12.0 법적 기반

```
라이선스: Apache License 2.0
```

| 가능 여부 | 항목 |
|---|---|
| 가능 | 상업적 이용 |
| 가능 | 수정·개조 |
| 가능 | 재배포 (유료 포함) |
| 가능 | 특허 사용 허가 |
| 가능 | **리브랜딩 후 판매** |
| 가능 | 비공개 소스 유지 (수정분 공개 의무 없음) |
| 의무 | 라이선스 사본 + `NOTICE` 파일 포함 |
| 의무 | 변경 사항 표시 |
| 금지 | 상표권 침해 (RAGFlow/InfiniFlow 상표를 자기 브랜드로 사용) |

> GPL이 아니므로 **수정 소스 공개 의무가 없다.** 상업적으로 가장 유리한 라이선스.

---

### 아이디어 1. 폐쇄망(On-Premise) RAG 구축 대행 ★★★★★

**핵심 통찰**

```
병원·금융·국방·공공·대기업 법무팀의 공통 고민:
  "AI를 도입하고 싶지만, 우리 데이터를 외부(OpenAI)로 보낼 수 없다"

→ RAGFlow + Ollama = 완전 폐쇄망, 인터넷 연결 0, 데이터 유출 0
→ ChatGPT Enterprise도, Copilot도 충족할 수 없는 요건
```

**수익 구조**

| 항목 | 금액 |
|---|---|
| 초기 구축 (서버 세팅 + 데이터 마이그레이션 + 커스터마이징) | 2,000만 ~ 8,000만원 |
| 연간 유지보수 (업데이트·모니터링·튜닝) | 초기 비용의 15~20%/년 |
| 추가 커넥터/파서 개발 | 건당 500만 ~ 2,000만원 |
| 임직원 교육 | 1회 300만 ~ 800만원 |

**타겟 영업 우선순위**

```
1순위: 중견 병원 (의료기록·논문·진료지침 / 개인정보보호법)
2순위: 증권사·자산운용사 (리서치 리포트·규정 / 금융보안원 규제)
3순위: 공공기관·지자체 (민원 응대·내부 규정 / 망분리 의무)
4순위: 제조 대기업 (기술문서·도면·품질이력 / 영업비밀)
5순위: 법무법인 (판례·계약서 / 비밀유지의무)
```

**영업 멘트**

> "저희는 데이터가 서버 밖으로 1바이트도 나가지 않습니다.
>  인터넷 선을 뽑아도 작동합니다.
>  그리고 모든 답변에 원본 문서 페이지가 표시되어 감사(Audit) 대응이 가능합니다."

**ROI 평가**

```
+ 초기 자본 거의 0 (오픈소스 + 노트북)
+ 단가 높음 (건당 수천만원)
+ 반복 매출 (유지보수 계약)
+ 경쟁 적음 (폐쇄망 구축 역량 보유자 희소)
- 영업 난이도 높음 (B2B, 레퍼런스 필요)

→ 전략: 첫 레퍼런스 1건을 저가(또는 무료)로 확보. 이후 영업이 쉬워진다.
```

---

### 아이디어 2. 버티컬 AI SaaS ★★★★★

**핵심 통찰**

```
RAGFlow는 "범용" 도구 → 쓸 줄 아는 사람만 사용
특정 업종에 특화 → 그 업종 사람이 "내 문제 해결 도구"로 인식 → 결제
```

**예시 ① 의료 논문 리서치 SaaS**

```
기반: RAGFlow (헤드리스, 백엔드로만 사용)
커스터마이징:
  · paper 청킹 템플릿 + 의학 용어 사전 튜닝
  · PubMed 커넥터 자동 동기화
  · 의료 가이드라인 KB 사전 탑재
  · GraphRAG로 "약물-질환-부작용" 관계 그래프
프론트: React 전용 UI (의사 UX 맞춤)
가격: 의사 1인당 월 5만원 → 100명 = MRR 500만원
```

**예시 ② 계약서 리스크 검토 SaaS**

```
· laws 청킹 템플릿 (조/항/목 단위)
· "불리한 조항 자동 탐지" 에이전트
· 판례 DB + 표준계약서 KB
· 출력: 리스크 리포트 PDF (docs_generator 컴포넌트)
· 가격: 건당 5만원(종량) 또는 월 50만원(무제한)
· 타겟: 법무팀 없는 스타트업·중소기업 (시장 큼)
```

**예시 ③ 제조 기술문서 Q&A**

```
· 도면·매뉴얼·품질이력 통합 (manual + table 템플릿)
· 현장 작업자용 모바일 UI + 음성 질문 (STT 모델)
· "이 설비 에러코드 E-204 대처법?" → 즉답 + 매뉴얼 페이지
· 가격: 공장당 월 100만 ~ 300만원
```

**예시 ④ 학원/교육 AI 조교**

```
· 교재·기출문제 KB + qa 템플릿
· 학생 질문 24시간 응대 (텔레그램 채널)
· 오답 패턴 분석 리포트 (data_operations 컴포넌트)
· 가격: 학원당 월 30만 ~ 100만원 (영업 난이도 낮음)
```

**SaaS 가격 설계 템플릿**

| 플랜 | 가격 | 제공 |
|---|---|---|
| Free | 0원 | 문서 10개, 질문 50회/월 (바이럴용) |
| Starter | 월 5만원 | 문서 500개, 질문 1,000회 |
| Pro | 월 25만원 | 문서 5,000개, 무제한 질문, API |
| Enterprise | 협의 | 전용 인스턴스, SSO, SLA, 온프레미스 |

> RAGFlow에 **멀티테넌시가 이미 구현**되어 있다.
> (`api/db/services/tenant_*_service.py`, 사용량 통계 `stats_api.py`)
> → SaaS 구축 난이도가 크게 낮아진다.

---

### 아이디어 3. RAG-as-a-Service API ★★★★

**콘셉트**

```
"RAG 파이프라인을 직접 만들지 마세요. API 한 줄로 끝납니다."

POST https://api.myrag.io/v1/ingest   # 문서 업로드
POST https://api.myrag.io/v1/ask      # 질문 → 답변 + 출처
GET  https://api.myrag.io/v1/usage    # 사용량
```

**과금 모델 (종량제)**

| 항목 | 단가 |
|---|---|
| 문서 파싱 | 페이지당 10원 (OCR 필요 시 30원) |
| 임베딩 | 1,000토큰당 1원 |
| 질문 (검색+LLM) | 건당 50~200원 (모델별) |
| 스토리지 | GB당 월 1,000원 |

**타겟**

```
· 1인 개발자 / 인디해커 (RAG 구축 시간 부족)
· 스타트업 MVP (빠른 검증)
· 노코드 툴 사용자 (Bubble, Webflow, Zapier 연동)
· 기존 SaaS에 "AI 검색" 기능 추가하려는 회사
```

**현실 체크**

```
+ 확장성 좋음, 자동화 후 패시브화 가능
- 인프라 비용 선투자 (16GB+ 서버 복수)
- LLM API 원가 관리 필수 (마진 계산 정밀하게)
- 경쟁 존재 (Pinecone, Vectara 등)

→ 차별화: "DeepDoc 파싱 품질" + "출처 정확도"
   → "표가 깨지지 않는 유일한 RAG API" 포지셔닝
```

---

### 아이디어 4. 커넥터·파서 개발 외주 ★★★★

**틈새 발견**

RAGFlow에 커넥터가 40개 있지만 **한국 서비스는 거의 없다.**

**한국형 커넥터 개발 기회**

```
현재 없는 것 (= 사업 기회):
  · 카카오워크 / 네이버웍스
  · 잔디(JANDI) / 플로우(Flow)
  · 한글(.hwp / .hwpx) 파서          ← 최우선
  · 더존 ERP / 영림원 / SAP Korea
  · 전자세금계산서 / 전자결재 시스템
  · 나라장터 / 온나라 문서
  · 그룹웨어 (다우오피스, 하이웍스)
```

**HWP 파서 = 한국 시장 독점 무기**

```
공공기관·학교·병원 문서 상당수가 .hwp
→ 글로벌 RAG 툴은 HWP를 읽지 못한다
→ HWP 파서를 만들면 한국 공공 시장 사실상 독점

구현 경로:
  deepdoc/parser/hwp_parser.py 신규 작성
  (pyhwp, hwp5 등 라이브러리 활용 → DeepDoc 파이프라인 연결)
  내부적으로는 markdown/txt 파서와 동일한 인터페이스로 맞추면 된다
```

**수익**

| 항목 | 금액 |
|---|---|
| 커넥터 1개 개발 | 500만 ~ 2,000만원 |
| HWP 파서 개발 + 유지보수 | 3,000만원 + 연 500만원 |
| 커넥터 라이선스 재판매 (여러 고객) | 건당 300만 ~ 500만원 |
| 업스트림 기여 후 명성 → 인바운드 문의 | 무형 자산 |

> **전략:** HWP 파서를 업스트림에 기여(PR)하면
> "RAGFlow HWP 파서 개발자"라는 타이틀을 얻어
> 한국 RAG 시장에서 독점적 권위를 확보할 수 있다.

---

### 아이디어 5. 교육·강의·컨설팅 ★★★★★ (즉시 시작 가능)

**왜 첫 시작점인가**

```
· 초기 자본 0원
· 즉시 시작 가능
· 아이디어 1~3번 사업의 "영업 채널"이 된다
· 실패 리스크 없음
```

**수익 레이어 (5단)**

| 레이어 | 형태 | 단가 | 난이도 |
|---|---|---|---|
| 1 | 유튜브 애드센스 | 조회수 기반 | ★ |
| 2 | 온라인 강의 (인프런/클래스101/자체) | 20~30만원 × N명 | ★★ |
| 3 | 기업 출강 (사내 AI 교육) | 1일 300만~800만원 | ★★★ |
| 4 | 1:1 컨설팅 | 시간당 30만~50만원 | ★★ |
| 5 | 멤버십/구독 (템플릿·에이전트 제공) | 월 3만~10만원 × N명 | ★★ |

**강의 상품 설계**

```
입문반 (20만원): "사내 문서 AI 챗봇 만들기"
  → 설치 + 지식베이스 + 챗봇 배포. 비개발자 대상

실전반 (35만원): "RAG 품질 튜닝 + 에이전트 구축"
  → 청킹 전략, 리랭킹, GraphRAG, 캔버스 에이전트

사업반 (80만원): "RAG 구축 사업 시작하기"   ← 마진 최고
  → 폐쇄망 구축, 견적서 작성, 계약서 템플릿, 영업 스크립트
  → 수강생이 곧 하청 파트너 네트워크가 된다
```

**핵심 전략**

```
강의로 돈 벌기 < 강의로 "구축 문의" 받기

강의 수강생 100명
  → 그 중 5명이 "우리 회사에도 구축해 주세요"
  → 구축 외주 5건 × 3,000만원 = 1억 5천만원

→ 강의는 최고의 B2B 리드 생성기
```

---

### 아이디어 6. 템플릿·에이전트 마켓플레이스 ★★★

**콘셉트**

```
RAGFlow 에이전트는 JSON 파일로 공유 가능 (agent/templates/*.json)
→ 프리미엄 템플릿 판매 가능
```

**상품 예시**

| 상품 | 가격 | 내용 |
|---|---|---|
| 한국 세무 상담 에이전트 | 15만원 | 세법 KB + 상담 플로우 + 프롬프트 |
| 공공입찰 분석 에이전트 | 30만원 | 나라장터 파싱 + 요건 체크리스트 |
| 의료 논문 리뷰 에이전트 | 25만원 | PubMed + 메타분석 플로우 |
| HWP 공문 처리 파이프라인 | 40만원 | HWP 파서 + 공문 청킹 |
| 이커머스 CS 봇 패키지 | 20만원 | FAQ KB + 주문조회 API + 메신저 연동 |
| 번들 (전체) | 120만원 | 전부 + 업데이트 1년 |

**수익 특성**

```
+ 한 번 제작 후 반복 판매 (패시브 인컴)
+ 원가 0 (디지털 상품)
+ 강의/유튜브와 시너지
- 복제 리스크 (JSON이라 공유 용이)

→ 대응: "템플릿 + 전용 KB 데이터 + 1:1 설치 지원" 패키지로 판매
   (데이터와 지원은 복제 불가)
```

---

### 아이디어 7. 매니지드 호스팅 ★★★

**콘셉트**

```
"RAGFlow를 쓰고 싶지만 16GB 서버 관리가 부담스럽다"
→ 대신 관리해 주고 구독료를 받는다
```

**플랜**

| 플랜 | 월 가격 | 스펙 | 원가(추정) | 마진 |
|---|---|---|---|---|
| Solo | 9만원 | 공유 인스턴스, 문서 1,000개 | ~3만원 | 67% |
| Team | 29만원 | 전용 컨테이너 2vCPU/16GB | ~12만원 | 59% |
| Business | 99만원 | 전용 VM 8vCPU/32GB + 백업 | ~40만원 | 60% |
| Enterprise | 협의 | 전용 클러스터 + SLA + SSO | - | - |

**부가 서비스 (업셀)**

```
· 자동 백업/복구              +월 3만원
· 업스트림 업데이트 대행       +월 5만원
· 모니터링 대시보드           +월 3만원
· 커스텀 도메인/SSL           +월 2만원
· 한국 리전 (데이터 국내 보관)  +월 10만원
```

**주의**

```
- 서버 비용 선투자 필요
- 24/7 장애 대응 부담
- 공식 클라우드 서비스(cloud.ragflow.io)와 경쟁

→ 차별화: "한국 리전 + 한국어 지원 + 개인정보보호법 대응"
```

---

## 13. 수익화 로드맵 4단계

### Phase 1 (0~3개월) — 무자본 시작, 신뢰 쌓기

```
[ ] RAGFlow 완전 숙달 (설치 → 튜닝 → 에이전트 → 배포)
[ ] 유튜브 입문 시리즈 5편 업로드
[ ] 기술 블로그 포스팅 (SEO 유입 확보)
[ ] HWP 파서 프로토타입 개발 → 업스트림 PR
[ ] 무료/저가 레퍼런스 1건 확보 (지인 회사, 소규모 병원 등)

매출 목표: 0 ~ 500만원 (레퍼런스 자체가 자산)
```

### Phase 2 (3~6개월) — 첫 수익, 교육 + 외주

```
[ ] 온라인 강의 출시 (입문반 20만원)
[ ] Phase 1 레퍼런스 기반 유료 구축 2~3건 수주
[ ] 템플릿 2~3개 제작 → 판매 시작
[ ] 기업 출강 영업 (강의 수강생 회사부터)

매출 목표: 3,000만 ~ 8,000만원
```

### Phase 3 (6~12개월) — 스케일, 폐쇄망 + SaaS

```
[ ] 폐쇄망 구축 사업 본격화 (병원/금융 타겟)
[ ] 유지보수 계약 누적 (반복 매출 확보)
[ ] 버티컬 SaaS MVP 1개 출시 (반응 좋았던 업종 선택)
[ ] 사업반 강의 출시 → 하청 파트너 네트워크 구축

매출 목표: 2억 ~ 5억원
```

### Phase 4 (12개월+) — 자산화

```
[ ] SaaS MRR 성장 (구독 기반 안정 매출)
[ ] 하청 파트너 네트워크로 구축 사업 확장
[ ] 템플릿 마켓 패시브 인컴
[ ] (선택) 투자 유치 또는 매각
```

---

## 14. 리스크 체크리스트

| 리스크 | 대응 전략 |
|---|---|
| 업스트림 변경 속도가 매우 빠름 | 포크를 깊게 수정하지 말고 **API 레이어 위에 쌓기** |
| 원 개발사(InfiniFlow)의 상용 진출 | 지역/버티컬 특화로 차별화 (한국 규제·HWP·한국어) |
| 경쟁 증가 (Dify, LangFlow 등) | "DeepDoc 파싱 품질" + "출처 정확도"로 승부 |
| LLM 가격 변동 | 멀티 프로바이더 구조 유지(78개) + Ollama 옵션 확보 |
| 상표권 | "RAGFlow"를 자기 브랜드로 쓰지 않고 "RAGFlow 기반"으로 표기 |
| 기술 지원 부담 | 유지보수 계약에 **SLA 범위·대응시간 명시** (무한 지원 금지) |
| 리소스 요구 (16GB+) | 고객에게 하드웨어 요건 사전 고지 |
| ARM64 미지원 | M1/M2 맥은 직접 빌드 (공식 가이드 참조) |
| Python/Go 듀얼 트랙 | 수정 전 실제 실행 경로 확인 필수 |
| 규제 (개인정보/의료법) | 폐쇄망 구성이 오히려 강점. 법무 검토 선행 |

---

## 15. 부록: 빠른 참조표

### 15.1 숫자 요약

| 항목 | 수량 |
|---|---|
| 전체 파일 | 6,214개 |
| Python / Go / TS 파일 | 1,341 / 2,315 / 1,452 |
| 문서 파서 | 18종 |
| 청킹 템플릿 | 14종 |
| 에이전트 컴포넌트 | 22개 |
| 에이전트 외부 툴 | 27개 |
| 에이전트 완성 템플릿 | 19개 |
| 외부 데이터 커넥터 | 40개+ |
| REST API 모듈 | 29개 |
| LLM 프로바이더 | 78개 |
| 메신저 채널 | 8종 |
| 검색엔진 백엔드 | 4종 (ES/OpenSearch/Infinity/SereneDB) |
| 메타DB 백엔드 | 4종 (MySQL/GaussDB/OceanBase/SeekDB) |

### 15.2 핵심 명령어

```bash
# ── Docker 기동 ───────────────────────────────
sudo sysctl -w vm.max_map_count=262144
cd docker && docker compose -f docker-compose.yml up -d
docker logs -f docker-ragflow-cpu-1

# ── 백엔드 (소스) ─────────────────────────────
uv sync --python 3.13 --all-extras
uv run python3 ragflow_deps/download_deps.py
docker compose -f docker/docker-compose-base.yml up -d
source .venv/bin/activate && export PYTHONPATH=$(pwd)
bash docker/launch_backend_service.sh
uv run pytest && ruff check && ruff format

# ── 프론트엔드 ────────────────────────────────
cd web && npm install
npm run dev / build / lint / test / type-check

# ── Go (build.sh 필수) ────────────────────────
uv run ragflow_deps/download_deps.py
bash build.sh --go
bash build.sh --all
bash build.sh --test ./internal/parser/...
bash build.sh --test-integration ./...
bash build.sh --test-e2e
bash build.sh --test-manual        # 로컬 전용
bash build.sh --test-all

# ── MCP 서버 ──────────────────────────────────
python mcp/server/server.py --mode=self-host   # 포트 9382
```

### 15.3 주요 설정 파일

| 파일 | 역할 |
|---|---|
| `docker/.env` | 환경 변수 (엔진 선택, 포트, 비밀번호) |
| `docker/service_conf.yaml.template` | 서비스 설정 (기본 LLM, API 키) |
| `conf/llm_factories.json` | LLM 프로바이더 정의 (78개) |
| `conf/system_settings.json` | 시스템 설정 |
| `conf/mapping.json`, `infinity_mapping.json`, `os_mapping.json` | 검색 인덱스 매핑 |
| `pyproject.toml` | Python 의존성 + 버전 |
| `go.mod` | Go 의존성 |
| `web/package.json` | 프론트엔드 의존성 |
| `build.sh` | Go/네이티브 빌드 + 테스트 러너 |
| `CLAUDE.md` / `AGENTS.md` / `GEMINI.md` / `web/CLAUDE.md` | AI 코딩 가이드 |
| `internal/development.md` | Go 개발/빌드 트러블슈팅 |

### 15.4 링크

| 대상 | URL |
|---|---|
| **분석 대상 포크** | https://github.com/bmshin94/ragflow |
| 원본 업스트림 | https://github.com/infiniflow/ragflow |
| 공식 문서 | https://ragflow.io/docs/dev/ |
| 클라우드 서비스 | https://cloud.ragflow.io |
| Discord 커뮤니티 | https://discord.gg/NjYzJD3GM3 |
| 로드맵 Issue | https://github.com/infiniflow/ragflow/issues/12241 |
| DeepWiki (코드 Q&A) | https://deepwiki.com/infiniflow/ragflow |
| 공식 Skill (OpenClaw) | https://clawhub.ai/yingfeng/ragflow-skill |
| Docker Hub | https://hub.docker.com/r/infiniflow/ragflow |
| HTTP API 레퍼런스 | `docs/references/http_api_reference.md` |
| Python API 레퍼런스 | `docs/references/python_api_reference.md` |
| API 키 발급 가이드 | `docs/develop/acquire_ragflow_api_key.md` |

### 15.5 최종 3줄 요약

1. **읽기** — 복잡한 PDF·표·스캔본을 사람처럼 정확히 읽는 "눈"(DeepDoc)
2. **찾기** — 하이브리드 검색 + 재순위 + GraphRAG로 정답 조각만 선별
3. **쓰기** — 출처 붙인 답변 + 에이전트 워크플로우 + 8종 메신저 배포

> **= "내 데이터로 환각 없는 AI를 만드는 완제품 플랫폼" (Apache 2.0, 상업적 이용 가능)**

---

*이 문서는 `bmshin94/ragflow` 포크를 Claude Code 세션에서 전수조사하여 작성한 한국어 분석 리포트입니다.*
