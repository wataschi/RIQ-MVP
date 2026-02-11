# RIQ: Архітектурні діаграми v5

> **Оновлено**: Лютий 2026
> **Базується на**: SGR Agent Best Practices, MCP Standards, Hybrid RAG, 19 MVP KPI
> **Продуктова модель**: Платформа терапевтичних програм з AI-помічником

---

## 1. User Journey 

```
┌────────────────────────────────────────────────────────────────┐
│                     USER JOURNEY: RIQ                          │
│                                                                │
│  ┌─────────────┐    ┌──────────────┐    ┌──────────────────┐   │
│  │  LANDING    │──▶│  ДЕМО-        │──▶│ ЗАВАНТАЖЕННЯ     │   │
│  │  PAGE       │    │ ОПИТУВАННЯ   │    │ ДОДАТКУ          │   │
│  │             │    │  (3-5 питань)│    │ (App/Play Store) │   │
│  └─────────────┘    └──────────────┘    └────────┬─────────┘   │
│  Conv: 1-5%          Compl: 25-50%               │             │
│                                                   ▼            │
│  ┌────────────────────────────────────────────────────────────┐│
│  │  РЕЄСТРАЦІЯ → СТАРТОВЕ ОПИТУВАННЯ (5-10 питань)           │ │
│  │  Assessment: тривога, депресія, стрес, цілі, преференції  │ │
│  │  Completion target: 65-75%                                │ │
│  └────────────────────────┬───────────────────────────────────┘│
│                            │                                   │
│                            ▼                                   │
│  ┌────────────────────────────────────────────────────────────┐│
│  │  AI РЕКОМЕНДУЄ ПРОГРАМУ (на основі survey scores)         │ │
│  │  Selection target: 25-45%                                 │ │
│  └────────────────────────┬───────────────────────────────────┘│
│                            │                                   │
│                            ▼                                   │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │  ПРОГРАМА (4-8 тижнів)                       Start: 40-60%│ │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐      │ │
│  │  │ Модуль 1 │→│ Модуль 2 │→│ Модуль 3 │→│ Модуль N │      │ │
│  │  │          │ │          │ │          │ │          │      │ │
│  │  │┌────────┐│ │┌────────┐│ │┌────────┐│ │┌────────┐│      │ │
│  │  ││Вправа 1││ ││Вправа 1││ ││Вправа 1││ ││Вправа 1││      │ │
│  │  ││        ││ ││        ││ ││        ││ ││        ││      │ │
│  │  ││   ↓    ││ ││   ↓    ││ ││   ↓    ││ ││   ↓    ││      │ │
│  │  ││Рефлекс.││ ││Рефлекс.││ ││Рефлекс.││ ││Рефлекс.││      │ │
│  │  │└────────┘│ │└────────┘│ │└────────┘│ │└────────┘│      │ │
│  │  │┌────────┐│ │┌────────┐│ │┌────────┐│ │          │      │ │
│  │  ││Вправа 2││ ││Вправа 2││ ││Вправа 2││ │          │      │ │
│  │  │└────────┘│ │└────────┘│ │└────────┘│ │          │      │ │
│  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘      │ │
│  │                                                           │ │
│  │  Exercises: 25-40%/week | Reflections: 20-35%             │ │
│  │  Completion: 25-35%                                       │ │
│  └───────────────────────────┬───────────────────────────────┘ │
│                               │                                │
│         ┌─────────────────────┼─────────────────────┐          │
│         ▼                     ▼                     ▼          │
│  ┌──────────────┐  ┌──────────────────┐  ┌────────────────┐   │
│  │ AI-ПОМІЧНИК  │  │ MOOD TRACKING    │  │ JOURNAL        │   │
│  │ (в контексті │  │ (паралельно)     │  │ (паралельно)   │   │
│  │  програми)   │  │                  │  │                │   │
│  │ Rate: 30-50% │  │                  │  │                │   │
│  │ CSAT: ≥70%   │  │                  │  │                │   │
│  │ Citation:≥70%│  │                  │  │                │   │
│  └──────────────┘  └──────────────────┘  └────────────────┘   │
│                                                               │
│  ┌───────────────────────────────────────────────────────────┐│
│  │  ЗАВЕРШЕННЯ → РЕКОМЕНДАЦІЯ НАСТУПНОЇ ПРОГРАМИ             ││
│  └───────────────────────────────────────────────────────────┘│
│                                                               │
│  RETENTION: D7 25-35% | D30 12-22% | WAU/MAU 30-40%           │
│  MONETIZATION: Purchase rate 1-5% ($9.99/міс premium)         │
│                                                               │
└───────────────────────────────────────────────────────────────┘
```

---

## 2. Загальна архітектура системи

```
┌────────────────────────────────────────────────────────────────────┐
│                        CLIENTS                                     │
│  ┌──────────────┐   ┌──────────────┐   ┌───────────────────────┐   │
│  │  iOS App     │   │ Android App  │   │  Landing Page         │   │
│  │  (Flutter)   │   │  (Flutter)   │   │                       │   │
│  └──────┬───────┘   └──────┬───────┘   └──────────┬────────────┘   │
└─────────┼──────────────────┼───────────────────────┼───────────────┘
          │                  │                       │
          └──────────────────┼───────────────────────┘
                             │ HTTPS / WebSocket (SSE)
┌────────────────────────────┼──────────────────────────────────────────┐
│  BACKEND (Render.com)      │                                          │
│  ┌─────────────────────────┼────────────────────────────┐            │
│  │  FastAPI Server         ▼                            │            │
│  │  ┌──────────────────────────────────────────────┐    │            │
│  │  │ API Layer (SSE Stream + REST)                │    │            │
│  │  │ /v1/chat     /v1/programs    /v1/exercises   │    │            │
│  │  │ /v1/surveys  /v1/mood        /v1/assessments │    │            │
│  │  │ /v1/reflections /v1/subscriptions /v1/privacy│    │            │
│  │  └────────────────────┬─────────────────────────┘    │            │
│  │                       │                              │            │
│  │  ┌────────────────────▼─────────────────────────┐    │            │
│  │  │         GUARDRAILS LAYER                     │    │            │
│  │  │  Input:  PII ↔ Crisis ↔ Injection            │    │            │
│  │  │  Output: Hallucination ↔ Citation ↔ Diag ↔ Toxic│  │            │
│  │  └────────────────────┬─────────────────────────┘    │            │
│  │                       │                              │            │
│  │  ┌────────────────────▼─────────────────────────┐    │            │
│  │  │         AGENT CORE                           │    │            │
│  │  │  TherapyAgent (SGRToolCallingAgent)           │    │            │
│  │  │  ┌──────────┐ ┌──────────┐ ┌──────────┐     │    │            │
│  │  │  │Reasoning │→│ Action   │→│ Action   │     │    │            │
│  │  │  │  Phase   │ │Selection │ │Execution │     │    │            │
│  │  │  │(Pydantic)│ │  (FC)    │ │ (Tools)  │     │    │            │
│  │  │  │+program  │ │          │ │          │     │    │            │
│  │  │  │ context  │ │          │ │          │     │    │            │
│  │  │  └──────────┘ └──────────┘ └──────────┘     │    │            │
│  │  └────────────────────┬─────────────────────────┘    │            │
│  │                       │ MCP Protocol                 │            │
│  │  ┌────────────────────▼─────────────────────────┐    │            │
│  │  │         MCP SERVER (FastMCP)                 │    │            │
│  │  │  17 Tools + get_capabilities()               │    │            │
│  │  │  programs | exercises | surveys | therapy     │    │            │
│  │  │  analysis | assessments | system              │    │            │
│  │  └────┬──────────┬──────────┬───────────────────┘    │            │
│  └───────┼──────────┼──────────┼────────────────────────┘            │
│          │          │          │                                      │
│  ┌───────▼───┐ ┌────▼────┐ ┌──▼────────┐  ┌──────────┐             │
│  │ RAG       │ │   DB    │ │ LLM       │  │ Stripe/  │             │
│  │ Service   │ │ Layer   │ │ Router    │  │RevenueCat│             │
│  │ (Hybrid)  │ │         │ │           │  │          │             │
│  │ +Citation │ │         │ │           │  │          │             │
│  └─────┬─────┘ └────┬────┘ └──┬────────┘  └──────────┘             │
└────────┼────────────┼─────────┼──────────────────────────────────────┘
         │            │         │
    ┌────▼────┐  ┌────▼────┐  ┌▼───────────────────────┐
    │pgvector │  │Postgres │  │ Gemini        │ Claude  │
    │(Vector) │  │ (Data)  │  │Flash/Pro/Emb  │(crisis) │
    │ 768 dim │  │         │  │               │         │
    └─────────┘  └─────────┘  └───────────────┴─────────┘
```

---

## 3. Agent Core: Execution Cycle (with Program Context)

```
                    ┌───────────────────────────┐
                    │     USER MESSAGE          │
                    └────────────┬──────────────┘
                                 │
                    ┌────────────▼──────────────┐
                    │   GUARDRAILS INPUT        │
                    │   PII → Crisis → Inject   │
                    └────────────┬──────────────┘
                                 │
                ┌────────────────▼────────────────┐
                │                                 │
                │   ╔═══════════════════════════╗ │
        ┌───────│───║  PHASE 1: REASONING       ║ │
        │       │   ║  (Pydantic enforcement)   ║ │
        │       │   ║                           ║ │
        │       │   ║  reasoning_steps: [...]   ║ │
        │       │   ║  current_situation: "..." ║ │
        │       │   ║  therapeutic_approach: CBT║ │
        │       │   ║  program_context: "W2/E3" ║ │
        │       │   ║  task_completed: false    ║ │
        │       │   ╚═══════════╤═══════════════╝ │
        │       │               │                 │
        │       │   ╔═══════════▼═══════════════╗ │
        │       │   ║  PHASE 2: ACTION SELECT   ║ │
        │       │   ║  (Function Calling)       ║ │
  LOOP  │       │   ║                           ║ │
  until │       │   ║  Tool: get_exercise_      ║ │
  task_ │       │   ║  content(exercise_id)     ║ │
  comp- │       │   ╚═══════════╤═══════════════╝ │
  leted │       │               │                 │
        │       │   ╔═══════════▼═══════════════╗ │
        │       │   ║  PHASE 3: ACTION EXEC     ║ │
        │       │   ║  (Tool Execution)         ║ │
        │       │   ║                           ║ │
        │       │   ║  → MCP Server call        ║ │
        │       │   ║  → Result + Citations     ║ │
        │       │   ╚═══════════╤═══════════════╝ │
        │       │               │                 │
        │       │       task_completed?           │
        └───────│───── NO ◄─────┤                 │
                │               │ YES             │
                │   ╔═══════════▼═══════════════╗ │
                │   ║  FINAL ANSWER             ║ │
                │   ║  (Streaming SSE)          ║ │
                │   ║  + Citations [{book,ch}]  ║ │
                │   ╚═══════════════════════════╝ │
                └────────────────┬────────────────┘
                                 │
                    ┌────────────▼──────────────┐
                    │   GUARDRAILS OUTPUT       │
                    │  Halluc → Citation → Diag │
                    │  → Toxic                  │
                    └────────────┬──────────────┘
                                 │
                    ┌────────────▼──────────────┐
                    │     AI RESPONSE           │
                    │     (з цитуваннями)       │
                    └───────────────────────────┘
```

---

## 4. RAG Pipeline: Hybrid + Citation Tracking

```
┌─────────────────────────────────────────────────────────────────┐
│  KNOWLEDGE BASE: 5 електронних книг з психології               │
│                                                                 │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ │
│  │ Книга 1 │ │ Книга 2 │ │ Книга 3 │ │ Книга 4 │ │ Книга 5 │ │
│  │(PDF/EPUB)│ │(PDF/EPUB)│ │(PDF/EPUB)│ │(PDF/EPUB)│ │(PDF/EPUB)│ │
│  └────┬────┘ └────┬────┘ └────┬────┘ └────┬────┘ └────┬────┘ │
│       └───────────┴───────────┴───────────┴───────────┘       │
│                            │                                    │
│                            ▼                                    │
│  INGESTION PIPELINE (модульні "батарейки")                      │
│                                                                 │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐   │
│  │ Document │ → │ Chunking │ → │Embedding │ → │ Indexing  │   │
│  │ Parsing  │   │          │   │          │   │          │   │
│  │MarkItDown│   │ Semantic │   │Gemini    │   │ pgvector │   │
│  │(замінна) │   │512 tokens│   │emb-004   │   │IVFFLAT   │   │
│  │          │   │overlap 50│   │768 dim   │   │+ GIN FTS │   │
│  └──────────┘   └──────────┘   └──────────┘   └──────────┘   │
│                                                                 │
│  Metadata per chunk: {book_id, chapter, approach, page}         │
│  ~8000-16000 chunks total | Embedding cost: ~$5-15 one-time    │
│                                                                 │
│   При зміні будь-якої батарейки → запустити RAGAS Eval!     │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│  RETRIEVAL PIPELINE (Hybrid — 85%+) + CITATION EXTRACTION      │
│                                                                 │
│  ┌──────────┐                                                  │
│  │  Query   │                                                  │
│  │Enrichment│  + emotional_state + program_context             │
│  └────┬─────┘                                                  │
│       │                                                         │
│  ┌────▼─────────────────────────────────────┐                  │
│  │         HYBRID SEARCH                     │                  │
│  │  ┌─────────────┐   ┌──────────────────┐  │                  │
│  │  │ Vector      │   │  BM25            │  │                  │
│  │  │ (pgvector   │   │  (PostgreSQL     │  │                  │
│  │  │  cosine)    │   │   Full-Text)     │  │                  │
│  │  │  k=20       │   │   k=20           │  │                  │
│  │  └──────┬──────┘   └────────┬─────────┘  │                  │
│  │         │    alpha=0.5      │             │                  │
│  │         └────────┬──────────┘             │                  │
│  │                  │                        │                  │
│  │         ┌────────▼─────────┐              │                  │
│  │         │  Merge & Dedup   │              │                  │
│  │         └────────┬─────────┘              │                  │
│  └──────────────────┼────────────────────────┘                  │
│                     │                                           │
│  ┌──────────────────▼────────────────────────┐                  │
│  │         RERANKER                          │                  │
│  │  Cross-encoder scoring                    │                  │
│  │  top_k = 5                                │                  │
│  └──────────────────┬────────────────────────┘                  │
│                     │                                           │
│  ┌──────────────────▼────────────────────────┐                  │
│  │  CITATION EXTRACTION                      │                  │
│  │  chunk.metadata → Citation(book, ch, page)│                  │
│  │  Target: ≥70% відповідей з цитуваннями    │                  │
│  └──────────────────┬────────────────────────┘                  │
│                     │                                           │
│  ┌──────────────────▼────────────────────────┐                  │
│  │  RANKED RESULTS (5 chunks + citations)    │                  │
│  │  → Agent Context                          │                  │
│  └───────────────────────────────────────────┘                  │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│  EVAL PLATFORM (обов'язковий)                                   │
│                                                                 │
│  RAGAS Metrics:                                                 │
│  ├── Faithfulness      > 0.9   ██████████░  target              │
│  ├── Answer Relevancy  > 0.85  █████████░░  target              │
│  ├── Context Precision > 0.8   ████████░░░  target              │
│  └── Citation Rate     ≥ 70%   ███████░░░░  target              │
│                                                                 │
│  Запуск: при зміні ETL, при зміні промптів, щотижнево          │
└─────────────────────────────────────────────────────────────────┘
```

---

## 5. Program Domain Model

```
┌─────────────────────────────────────────────────────────────────┐
│                  PROGRAM DOMAIN MODEL                            │
│                                                                  │
│  ┌─────────────────┐                                            │
│  │    programs      │  CBT for Anxiety (4 weeks)                │
│  │                  │  Mindfulness Basics (6 weeks)             │
│  │ title            │  Emotion Regulation (8 weeks)             │
│  │ approach         │  ...                                       │
│  │ duration_weeks   │                                            │
│  │ difficulty       │                                            │
│  │ is_premium       │                                            │
│  └────────┬────────┘                                            │
│           │ 1:N                                                  │
│  ┌────────▼────────┐                                            │
│  │ program_modules  │  Week 1: Understanding Anxiety            │
│  │                  │  Week 2: Thought Patterns                 │
│  │ title            │  Week 3: Coping Strategies                │
│  │ order_index      │  ...                                       │
│  └────────┬────────┘                                            │
│           │ 1:N                                                  │
│  ┌────────▼────────┐                                            │
│  │   exercises      │  Thought Record, Body Scan,               │
│  │                  │  Progressive Relaxation,                  │
│  │ title            │  Journaling Prompt, ...                   │
│  │ type             │                                            │
│  │ content_json     │  {instructions, steps, tips, media}       │
│  │ duration_min     │                                            │
│  │ order_index      │                                            │
│  └─────────────────┘                                            │
│                                                                  │
│  ── USER PROGRESS ──────────────────────────────────────────── │
│                                                                  │
│  ┌─────────────────┐     ┌──────────────────┐                   │
│  │  user_programs   │────▶│  user_exercises   │                   │
│  │                  │     │                  │                   │
│  │ user_id          │     │ user_id          │                   │
│  │ program_id       │     │ exercise_id      │                   │
│  │ status:          │     │ user_program_id  │                   │
│  │  active          │     │ status:          │                   │
│  │  completed       │     │  pending         │                   │
│  │  paused          │     │  in_progress     │                   │
│  │  abandoned       │     │  completed       │                   │
│  │ started_at       │     │  skipped         │                   │
│  │ completed_at     │     │ completed_at     │                   │
│  └─────────────────┘     └────────┬─────────┘                   │
│                                    │ 1:1                         │
│                           ┌────────▼─────────┐                   │
│                           │   reflections     │                   │
│                           │                  │                   │
│                           │ content          │                   │
│                           │ mood_after (1-10)│                   │
│                           │ insights (AI)    │                   │
│                           └──────────────────┘                   │
│                                                                  │
│  ── SURVEYS & RECOMMENDATIONS ─────────────────────────────── │
│                                                                  │
│  ┌─────────────────┐     ┌──────────────────┐                   │
│  │   surveys        │     │ survey_responses  │                   │
│  │                  │◄────│                  │                   │
│  │ type: starting   │     │ user_id          │                   │
│  │   | demo         │     │ answers_json     │                   │
│  │   | assessment   │     │ score            │                   │
│  │ questions_json   │     └────────┬─────────┘                   │
│  └─────────────────┘              │ 1:N                          │
│                           ┌────────▼──────────────┐              │
│                           │ program_recommendations│              │
│                           │                       │              │
│                           │ program_id            │              │
│                           │ match_score (0-1)     │              │
│                           │ selected (bool)       │              │
│                           └───────────────────────┘              │
└──────────────────────────────────────────────────────────────────┘
```

---

## 6. 3-Рівнева Memory Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                    USER MESSAGE                              │
└────────────────────────┬─────────────────────────────────────┘
                         │
┌────────────────────────▼─────────────────────────────────────┐
│              CONTEXT BUILDER                                 │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  Level 1: SHORT-TERM (in context window)              │  │
│  │  ┌──────────────────────────────────────────────────┐  │  │
│  │  │ Last 20 messages          │ Current mood state   │  │  │
│  │  │ Session topics            │ Crisis flags         │  │  │
│  │  │ Active program/exercise   │ Session duration     │  │  │
│  │  └──────────────────────────────────────────────────┘  │  │
│  │  Джерело: in-memory, context window Gemini (2M tokens) │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  Level 2: MEDIUM-TERM (PostgreSQL query)              │  │
│  │  ┌──────────────────────────────────────────────────┐  │  │
│  │  │ Journal entries (7d)      │ Mood trends (7d)     │  │  │
│  │  │ Program progress          │ Assessment results   │  │  │
│  │  │ Completed exercises       │ Reflections          │  │  │
│  │  │ Therapy milestones        │ Notification history │  │  │
│  │  └──────────────────────────────────────────────────┘  │  │
│  │  Джерело: PostgreSQL SELECT WHERE created_at > 7d ago  │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  Level 3: LONG-TERM (Hybrid RAG)                      │  │
│  │  ┌──────────────────────────────────────────────────┐  │  │
│  │  │ 5 книг з психології (основне джерело знань!)    │  │  │
│  │  │ Вправи, техніки, рекомендації, психоедукація    │  │  │
│  │  │ Similar past moments      │ Therapy patterns     │  │  │
│  │  │ User growth trajectory    │ Historical insights  │  │  │
│  │  │ Past reflections          │ Program history      │  │  │
│  │  └──────────────────────────────────────────────────┘  │  │
│  │  Джерело: pgvector + BM25 + Reranker (85%+ accuracy)  │  │
│  │  Пріоритет: книги > chat_history > journal > reflection│  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
│  Assembled Context → Agent Reasoning Phase                   │
└──────────────────────────────────────────────────────────────┘
```

---

## 7. MCP Server Architecture

```
┌──────────────────────────────────────────────────────────────┐
│  TherapyAgent (SGRToolCallingAgent)                          │
│  ┌────────────────────────────────────────────────────────┐  │
│  │ ReasoningTool → ActionSelection → ToolExecution        │  │
│  └──────────────────────────┬─────────────────────────────┘  │
└─────────────────────────────┼────────────────────────────────┘
                              │ MCP Protocol (JSON-RPC)
┌─────────────────────────────▼────────────────────────────────┐
│  FastMCP Server "RIQ Therapy Assistant"                       │
│                                                              │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │  get_capabilities()  — ОБОВ'ЯЗКОВО ПЕРШИМ              │ │
│  │  → entities, approaches, constraints, examples          │ │
│  └─────────────────────────────────────────────────────────┘ │
│                                                              │
│  ┌───────────────┐ ┌───────────────┐ ┌────────────────────┐ │
│  │ PROGRAMS &    │ │ THERAPY &     │ │ ANALYSIS           │ │
│  │ EXERCISES     │ │ RAG           │ │                    │ │
│  │               │ │               │ │ analyze_           │ │
│  │ get_program_  │ │ search_       │ │  mood_state        │ │
│  │  context      │ │  therapy_     │ │                    │ │
│  │               │ │  materials    │ │ detect_            │ │
│  │ recommend_    │ │  (+citations) │ │  crisis_           │ │
│  │  programs     │ │               │ │  indicators        │ │
│  │               │ │ save_therapy_ │ │                    │ │
│  │ get_exercise_ │ │  insight      │ │ search_user_       │ │
│  │  content      │ │               │ │  history           │ │
│  │               │ │               │ │                    │ │
│  │ save_         │ │               │ │                    │ │
│  │  reflection   │ │               │ │                    │ │
│  │               │ │               │ │                    │ │
│  │ get_user_     │ │               │ │                    │ │
│  │  program_     │ │               │ │                    │ │
│  │  progress     │ │               │ │                    │ │
│  └───────┬───────┘ └───────┬───────┘ └──────────┬─────────┘ │
│          │                 │                     │           │
│  ┌───────────────┐ ┌──────────────────────────────────────┐ │
│  │ SURVEYS &     │ │ SYSTEM                               │ │
│  │ ASSESSMENTS   │ │                                      │ │
│  │               │ │ get_session_context (+ program state)│ │
│  │ process_      │ │ trigger_notification                 │ │
│  │  survey_      │ │ create_weekly_report                 │ │
│  │  response     │ │                                      │ │
│  │               │ │                                      │ │
│  │ get_          │ │                                      │ │
│  │  assessment_  │ │                                      │ │
│  │  results      │ │                                      │ │
│  └───────┬───────┘ └──────────────────────┬───────────────┘ │
│          │                                │                 │
│  ┌───────▼────────────────────────────────▼───────────────┐ │
│  │  ToolResponse (standard format)                        │ │
│  │  { success, data, error, metadata, citations }         │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  SECURITY: allowlist | scope-filtering | rate limiting │  │
│  │  Rate: 60/min per user | Audit: all tool calls logged  │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
│  Total: 17 tools (Programs 5, Therapy 2, Analysis 3,        │
│         Surveys 2, System 3) + get_capabilities()            │
└──────────────────────────────────────────────────────────────┘
         │                │                    │
    ┌────▼──┐        ┌────▼──┐           ┌─────▼────┐
    │ RAG   │        │ DB    │           │ n8n      │
    │Service│        │Layer  │           │Webhooks  │
    └───────┘        └───────┘           └──────────┘
```

---

## 8. Multi-Model LLM Routing

```
┌────────────────────────────────────────────────┐
│              USER MESSAGE + CONTEXT            │
└────────────────────┬───────────────────────────┘
                     │
┌────────────────────▼───────────────────────────┐
│              LLM ROUTER                        │
│                                                │
│  Аналіз:                                      │
│  ├── crisis_detected?                          │
│  ├── anxiety_level > 8?                        │
│  ├── token_count > 100K?                       │
│  └── task_type = quick/survey?                 │
│                                                │
│  ┌───────────────┬──────────┬────────────────┐ │
│  │               │          │                │ │
│  ▼               ▼          ▼                │ │
│  Crisis/         Large      Quick            │ │
│  Anxiety>8       context    task/survey      │ │
│  │               │          │                │ │
│  ▼               ▼          ▼                │ │
│ ┌─────────┐ ┌──────────┐ ┌──────────────┐   │ │
│ │ Claude  │ │ Gemini   │ │ Gemini       │   │ │
│ │ 4.5     │ │ 2.0 Pro  │ │ 2.0 Flash    │   │ │
│ │ Sonnet  │ │          │ │              │   │ │
│ │         │ │ Context: │ │ Context: 1M  │   │ │
│ │ 10%     │ │ 2M       │ │              │   │ │
│ │ $3/$15  │ │ 20%      │ │ 70%          │   │ │
│ │         │ │ $1.25/$10│ │ $0.10/$0.30  │   │ │
│ └────┬────┘ └────┬─────┘ └──────┬───────┘   │ │
│      │           │              │            │ │
│      └───────────┼──────────────┘            │ │
│                  ▼                            │ │
│         ┌────────────────┐                   │ │
│         │ FALLBACK CHAIN │                   │ │
│         │ Flash → Pro →  │                   │ │
│         │ Claude → Error │                   │ │
│         └────────────────┘                   │ │
│                                              │ │
│  Prompt Caching: увімкнено (>1024 tokens)    │ │
│  Embeddings: Gemini text-embedding-004       │ │
│  Провайдери: Gemini (основний) + Claude      │ │
│  Cost: ~$275-545/міс на 250-600 users        │ │
└──────────────────────────────────────────────┘ │
```

---

## 9. Guardrails Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                 ДВОСТОРОННІ GUARDRAILS                       │
│                                                             │
│  ═══════════════ INPUT GUARDRAILS ═══════════════           │
│                                                             │
│  User Message                                               │
│       │                                                     │
│  ┌────▼───────────────────────────────────────────┐        │
│  │  1. PII Detection & Masking                    │        │
│  │     ├── Phone: +380... → [PHONE]               │        │
│  │     ├── Email: user@... → [EMAIL]              │        │
│  │     └── Name: Іван Петров → [NAME]             │        │
│  └────┬───────────────────────────────────────────┘        │
│       │                                                     │
│  ┌────▼───────────────────────────────────────────┐        │
│  │  2. Crisis Detection (Multi-layer)             │        │
│  │     ├── Layer 1: Keyword matching              │        │
│  │     ├── Layer 2: Semantic analysis             │        │
│  │     └── Layer 3: Context-aware scoring         │        │
│  │     Score > 8 → CRISIS PROTOCOL                │        │
│  │              → n8n alert                        │        │
│  │              → Claude Sonnet (highest empathy)  │        │
│  └────┬───────────────────────────────────────────┘        │
│       │                                                     │
│  ┌────▼───────────────────────────────────────────┐        │
│  │  3. Prompt Injection Detection                 │        │
│  │     ├── Pattern matching                       │        │
│  │     └── Instruction boundary enforcement       │        │
│  └────┬───────────────────────────────────────────┘        │
│       │                                                     │
│       ▼  Cleaned message → Agent Core                       │
│                                                             │
│  ═══════════════ OUTPUT GUARDRAILS ══════════════           │
│                                                             │
│  Agent Response                                             │
│       │                                                     │
│  ┌────▼───────────────────────────────────────────┐        │
│  │  4. Hallucination Check                        │        │
│  │     └── Is response grounded in context?       │        │
│  │     └── Target: <10% ungrounded statements     │        │
│  └────┬───────────────────────────────────────────┘        │
│       │                                                     │
│  ┌────▼───────────────────────────────────────────┐        │
│  │  5. Citation Enforcement (NEW)                 │        │
│  │     └── Response references knowledge base?    │        │
│  │     └── Target: ≥70% responses with citations  │        │
│  │     └── Track: book_id, chapter, page          │        │
│  └────┬───────────────────────────────────────────┘        │
│       │                                                     │
│  ┌────▼───────────────────────────────────────────┐        │
│  │  6. Diagnosis Language Filter                  │        │
│  │     └── Remove medical diagnosis statements    │        │
│  │     └── Add disclaimer if needed               │        │
│  └────┬───────────────────────────────────────────┘        │
│       │                                                     │
│  ┌────▼───────────────────────────────────────────┐        │
│  │  7. Toxicity Check                             │        │
│  │     └── Score < 0.5 → PASS                     │        │
│  └────┬───────────────────────────────────────────┘        │
│       │                                                     │
│       ▼  Validated response (з цитуваннями) → User          │
└─────────────────────────────────────────────────────────────┘
```

---

## 10. Database Schema

```
┌─────────────────────────────────────────────────────────────────┐
│                    PostgreSQL + pgvector                          │
│                                                                  │
│  ── CORE ──────────────────────────────────────────────────── │
│                                                                  │
│  ┌──────────────┐     ┌──────────────────┐                      │
│  │   users       │     │   sessions        │                      │
│  ├──────────────┤     ├──────────────────┤                      │
│  │ id (uuid PK) │◄────│ user_id (FK)      │                      │
│  │ email         │     │ id (uuid PK)      │                      │
│  │ display_name  │     │ started_at        │                      │
│  │ preferences   │     │ ended_at          │                      │
│  │ subscription_ │     │ mood_start/end    │                      │
│  │  plan         │     │ topics (jsonb)    │                      │
│  │ created_at    │     │ program_id (FK)   │ ◄── AI в контексті  │
│  │ gdpr_consent  │     └────────┬─────────┘     програми         │
│  └──────┬───────┘              │                                 │
│         │              ┌────────▼─────────┐                      │
│         │              │  messages         │                      │
│         │              ├──────────────────┤                      │
│         │              │ id (uuid PK)     │                      │
│         │              │ session_id (FK)  │                      │
│         │              │ role             │                      │
│         │              │ content          │                      │
│         │              │ citations (jsonb)│ ◄── [{book,ch,page}] │
│         │              │ metadata         │                      │
│         │              └──────────────────┘                      │
│                                                                  │
│  ── PROGRAMS ────────────────────────────────────────────────── │
│                                                                  │
│  ┌──────────────┐     ┌──────────────────┐                      │
│  │  programs     │     │ program_modules   │                      │
│  ├──────────────┤     ├──────────────────┤                      │
│  │ id (uuid PK) │◄────│ program_id (FK)   │                      │
│  │ title         │     │ id (uuid PK)      │                      │
│  │ description   │     │ title             │                      │
│  │ approach      │     │ order_index       │                      │
│  │ duration_weeks│     └────────┬─────────┘                      │
│  │ difficulty    │              │                                 │
│  │ is_premium    │     ┌────────▼─────────┐                      │
│  └──────────────┘     │  exercises        │                      │
│                        ├──────────────────┤                      │
│                        │ id (uuid PK)     │                      │
│  ┌──────────────┐     │ module_id (FK)   │                      │
│  │ user_programs │     │ title            │                      │
│  ├──────────────┤     │ type             │                      │
│  │ id (uuid PK) │     │ content_json     │                      │
│  │ user_id (FK) │     │ duration_min     │                      │
│  │ program_id FK│     │ order_index      │                      │
│  │ status       │     └──────────────────┘                      │
│  │ started_at   │                                                │
│  │ completed_at │     ┌──────────────────┐                      │
│  └──────┬───────┘     │ user_exercises    │                      │
│         │              ├──────────────────┤                      │
│         └─────────────▶│ user_program_id  │                      │
│                        │ exercise_id (FK) │                      │
│                        │ status           │                      │
│                        │ completed_at     │                      │
│                        └────────┬─────────┘                      │
│                                 │                                 │
│                        ┌────────▼─────────┐                      │
│                        │  reflections      │                      │
│                        ├──────────────────┤                      │
│                        │ user_exercise_id │                      │
│                        │ content          │                      │
│                        │ mood_after (1-10)│                      │
│                        │ insights (jsonb) │                      │
│                        └──────────────────┘                      │
│                                                                  │
│  ── SURVEYS ──────────────────────────────────────────────── │
│                                                                  │
│  ┌──────────────┐     ┌──────────────────┐                      │
│  │  surveys      │     │ survey_responses  │                      │
│  ├──────────────┤     ├──────────────────┤                      │
│  │ type: starting│◄────│ survey_id (FK)    │                      │
│  │   | demo      │     │ user_id (FK)      │                      │
│  │   | assessment│     │ answers_json      │                      │
│  │ questions_json│     │ score (jsonb)     │                      │
│  └──────────────┘     └────────┬─────────┘                      │
│                                 │                                 │
│                        ┌────────▼──────────────┐                 │
│                        │program_recommendations │                 │
│                        ├───────────────────────┤                 │
│                        │ program_id (FK)       │                 │
│                        │ match_score (float)   │                 │
│                        │ selected (bool)       │                 │
│                        └───────────────────────┘                 │
│                                                                  │
│  ── MONETIZATION & ANALYTICS ────────────────────────────── │
│                                                                  │
│  ┌──────────────┐     ┌──────────────────┐                      │
│  │ subscriptions │     │ analytics_events  │                      │
│  ├──────────────┤     ├──────────────────┤                      │
│  │ user_id (FK) │     │ user_id (FK)     │                      │
│  │ plan         │     │ event_type       │                      │
│  │ status       │     │ metadata_json    │                      │
│  │ provider     │     │ created_at       │                      │
│  │ expires_at   │     └──────────────────┘                      │
│  └──────────────┘                                                │
│                                                                  │
│  ── EXISTING ────────────────────────────────────────────── │
│                                                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐       │
│  │ mood_records  │  │ journal_     │  │ assessments      │       │
│  │              │  │  entries     │  │                  │       │
│  │ mood (1-10)  │  │ content      │  │ type (PHQ9/GAD7) │       │
│  │ energy(1-10) │  │ mood_score   │  │ score            │       │
│  │ anxiety(1-10)│  │ tags (jsonb) │  │ interpretation   │       │
│  └──────────────┘  └──────────────┘  └──────────────────┘       │
│                                                                  │
│  ┌──────────────────────────────────────────────────────┐       │
│  │  document_embeddings (RAG)                           │       │
│  │  embedding vector(768)  — Gemini text-embedding-004  │       │
│  │  source: therapeutic_material | chat_history |       │       │
│  │          journal | reflection                        │       │
│  │  metadata: {book_id, chapter, page, approach}        │       │
│  │  Indexes: IVFFLAT (cosine) + GIN (FTS)              │       │
│  └──────────────────────────────────────────────────────┘       │
└─────────────────────────────────────────────────────────────────┘
```

---

## 11. n8n Integration

```
┌──────────────────────────────────────────────────────────────┐
│                   n8n WORKFLOWS (9)                           │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  1. Daily Reminder                                     │  │
│  │  Schedule 9:00 → Check inactive users → Push notif.    │  │
│  │  [Cron] → [HTTP: GET /inactive] → [Push: FCM]         │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  2. Program Reminder (NEW)                             │  │
│  │  Schedule 10:00 → Users without exercise 48h → Push    │  │
│  │  [Cron] → [HTTP: GET /inactive-program] → [Push: FCM] │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  3. Exercise Completion (NEW)                          │  │
│  │  Webhook → "Напишіть рефлексію" push prompt            │  │
│  │  [Webhook] → [Push: FCM "Як вам вправа?"]             │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  4. Program Completion (NEW)                           │  │
│  │  Webhook → Congratulations + recommend next program     │  │
│  │  [Webhook] → [Push] + [Email: next program suggestion] │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  5. Crisis Alert  P0                                   │  │
│  │  Webhook → Slack + Email admin + Audit log             │  │
│  │  [Webhook] → [Slack] + [Email] + [HTTP: POST /log]    │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  6. Weekly Report                                      │  │
│  │  Monday 10:00 → Program progress + mood trends → Email │  │
│  │  [Cron] → [HTTP: POST /report] → [Email: SendGrid]    │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  7. Onboarding Sequence                                │  │
│  │  New user → 3 emails (1h, 24h, 72h)                    │  │
│  │  [Webhook] → [Wait 1h] → [Email] → [Wait 24h] → ...   │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  8. Subscription Expiry (NEW)                          │  │
│  │  Daily → Users with subscription expiring in 3d → Email│  │
│  │  [Cron] → [HTTP: GET /expiring] → [Email: renewal]    │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  9. DB Backup                                          │  │
│  │  Daily 3:00 AM → pg_dump → S3 → cleanup >30d          │  │
│  │  [Cron] → [SSH: pg_dump] → [S3: Upload] → [Cleanup]   │  │
│  └────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────┘
```

---

## 12. Monitoring: 19 MVP Metrics Dashboard

```
┌──────────────────────────────────────────────────────────────────┐
│                  MONITORING & OBSERVABILITY (v5)                   │
│                                                                   │
│  ┌──────────────────────────────────────────────────────────────┐│
│  │  ACQUISITION (3 KPI)                                         ││
│  │  ┌──────────────────┐ ┌──────────────────┐ ┌──────────────┐ ││
│  │  │ Users            │ │ Landing Conv.    │ │ Demo Survey  │ ││
│  │  │ 250-600          │ │ 1-5%             │ │ Compl 25-50% │ ││
│  │  └──────────────────┘ └──────────────────┘ └──────────────┘ ││
│  └──────────────────────────────────────────────────────────────┘│
│                                                                   │
│  ┌──────────────────────────────────────────────────────────────┐│
│  │  ACTIVATION (4 KPI)                                          ││
│  │  ┌──────────────┐ ┌──────────────┐ ┌───────────┐ ┌────────┐││
│  │  │ Starting     │ │ Program      │ │ Program   │ │ AI     │││
│  │  │ Survey       │ │ Recommend.   │ │ Started   │ │ Interac│││
│  │  │ 65-75%       │ │ 25-45%       │ │ 40-60%    │ │ 30-50% │││
│  │  └──────────────┘ └──────────────┘ └───────────┘ └────────┘││
│  └──────────────────────────────────────────────────────────────┘│
│                                                                   │
│  ┌──────────────────────────────────────────────────────────────┐│
│  │  ENGAGEMENT (4 KPI)                                          ││
│  │  ┌──────────────┐ ┌──────────────┐ ┌───────────┐ ┌────────┐││
│  │  │ D7 Retention │ │ WAU/MAU      │ │ Exercises │ │ Reflec.│││
│  │  │ 25-35%       │ │ 30-40%       │ │ 25-40%/wk │ │ 20-35% │││
│  │  └──────────────┘ └──────────────┘ └───────────┘ └────────┘││
│  └──────────────────────────────────────────────────────────────┘│
│                                                                   │
│  ┌──────────────────────────────────────────────────────────────┐│
│  │  RETENTION & VALUE (3 KPI)                                   ││
│  │  ┌──────────────────┐ ┌──────────────────┐ ┌──────────────┐ ││
│  │  │ D30 Retention    │ │ Program Compl.   │ │ Human-weeks  │ ││
│  │  │ 12-22%           │ │ 25-35%           │ │ 8000-12000   │ ││
│  │  └──────────────────┘ └──────────────────┘ └──────────────┘ ││
│  └──────────────────────────────────────────────────────────────┘│
│                                                                   │
│  ┌──────────────────────────────────────────────────────────────┐│
│  │  AI QUALITY (3 KPI)                                          ││
│  │  ┌──────────────────┐ ┌──────────────────┐ ┌──────────────┐ ││
│  │  │ CSAT             │ │ Citation Rate    │ │ Hallucination│ ││
│  │  │ ≥70% positive    │ │ ≥70%             │ │ <10%         │ ││
│  │  └──────────────────┘ └──────────────────┘ └──────────────┘ ││
│  └──────────────────────────────────────────────────────────────┘│
│                                                                   │
│  ┌──────────────────────────────────────────────────────────────┐│
│  │  BUSINESS & TECHNICAL (2 KPI + ops)                          ││
│  │  ┌──────────┐ ┌──────────┐ ┌────────┐ ┌───────┐ ┌────────┐ ││
│  │  │ Purchase │ │ Crash-   │ │ p95    │ │Uptime │ │ Error  │ ││
│  │  │ 1-5%     │ │ free≥97% │ │ <2s    │ │≥99.5% │ │ <1%    │ ││
│  │  └──────────┘ └──────────┘ └────────┘ └───────┘ └────────┘ ││
│  └──────────────────────────────────────────────────────────────┘│
│                                                                   │
│  ┌──────────────────────────────────────────────────────────────┐│
│  │  ML METRICS (continuous eval)                                ││
│  │  RAG faith >0.9 | relevancy >0.85 | precision >0.8          ││
│  │  Crisis detection >95%                                       ││
│  └──────────────────────────────────────────────────────────────┘│
│                                                                   │
│  TOOLS: Prometheus + Grafana | Sentry | Firebase Analytics       │
│         structlog | LangSmith                                     │
│                                                                   │
│  ALERTS:                                                          │
│  P0: CrisisFailure → Slack + SMS                                 │
│  P1: HighErrorRate, HighCrashRate → Slack + Email                 │
│  P2: SlowResponse, HighCost, LowCitationRate → Slack             │
└──────────────────────────────────────────────────────────────────┘
```

---

## 13. Deployment Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                   PRODUCTION                                  │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Render.com                                          │   │
│  │  ├── Web Service: riq-backend ($7/міс)               │   │
│  │  │   ├── Docker (python:3.12-slim)                   │   │
│  │  │   ├── FastAPI + Uvicorn (4 workers)               │   │
│  │  │   ├── Auto-deploy from main branch                │   │
│  │  │   └── Health check: /health                       │   │
│  │  └── PostgreSQL: riq-db ($7/міс)                     │   │
│  │      ├── pgvector + pg_trgm extensions               │   │
│  │      └── Daily backups (automated)                   │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Vercel (Landing Page)                  FREE         │   │
│  │  ├── Next.js SSR/SSG                                 │   │
│  │  ├── Demo survey → FastAPI API calls                 │   │
│  │  └── Analytics: UTM tracking, conversion events      │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  External Services:                                  │   │
│  │  ├── Supabase Auth:        $25/міс                   │   │
│  │  ├── n8n Cloud:            $20/міс                   │   │
│  │  ├── RevenueCat:           $0 (free до $2.5K MRR)    │   │
│  │  ├── Stripe:               2.9% + $0.30/txn          │   │
│  │  ├── Sentry:               $26/міс                   │   │
│  │  ├── Grafana Cloud:        Free tier                 │   │
│  │  ├── Firebase Analytics:   Free tier                 │   │
│  │  └── LangSmith:            Free tier                 │   │
│  └──────────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────┐
│                   DEVELOPMENT (Local)                         │
│                                                              │
│  docker-compose.yml:                                        │
│  ├── app:      FastAPI (hot-reload)                         │
│  ├── db:       PostgreSQL 16 + pgvector                     │
│  ├── n8n:      n8n local (optional)                         │
│  └── tests:    pytest runner                                │
└──────────────────────────────────────────────────────────────┘
```

---

## 14. Security Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                    SECURITY LAYERS                            │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  Layer 1: Transport                                    │  │
│  │  ├── HTTPS/TLS 1.3 (Render managed)                   │  │
│  │  ├── CORS whitelist (mobile app + landing domains)     │  │
│  │  └── Rate limiting (60 req/min per user)               │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  Layer 2: Authentication                               │  │
│  │  ├── Supabase Auth (JWT + refresh tokens)              │  │
│  │  ├── Social login (Google, Apple)                      │  │
│  │  └── Token validation middleware                       │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  Layer 3: Data Protection                              │  │
│  │  ├── PII masking before LLM (phone, email, name)       │  │
│  │  ├── Encryption at rest (PostgreSQL)                    │  │
│  │  ├── No PII in logs (structlog filtering)              │  │
│  │  └── API keys in env vars (never in code)              │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  Layer 4: AI Safety                                    │  │
│  │  ├── Input guardrails (PII, crisis, injection)         │  │
│  │  ├── Output guardrails (hallucination, citation, diag) │  │
│  │  ├── MCP tool allowlist (17 tools only)                │  │
│  │  ├── MCP scope-filtering (user sees own data only)     │  │
│  │  └── All tool calls logged (audit trail)               │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  Layer 5: Payments Security                            │  │
│  │  ├── Stripe webhook signature validation               │  │
│  │  ├── RevenueCat server-side receipt validation          │  │
│  │  ├── Subscription status checked server-side           │  │
│  │  └── No pricing logic on client                        │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  Layer 6: GDPR Compliance                              │  │
│  │  ├── GET /v1/privacy/export   → JSON export            │  │
│  │  ├── DELETE /v1/privacy/delete → Full erasure           │  │
│  │  ├── Consent tracking (gdpr_consent field)             │  │
│  │  ├── Data retention policy (auto-cleanup >12 months)   │  │
│  │  └── Right to be forgotten (cascade delete)            │  │
│  └────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────┘
```

---

## 15. Cost Optimization Strategy

```
┌──────────────────────────────────────────────────────────────┐
│               COST OPTIMIZATION                              │
│                                                              │
│  ┌──────────────────────────────────────────────────┐       │
│  │  1. Multi-Model Routing (економія 60-70%)        │       │
│  │                                                  │       │
│  │  Before: 100% Claude Sonnet = ~$2000/міс         │       │
│  │  After:  Gemini (90%) + Claude crisis (10%)      │       │
│  │          = ~$275/міс                              │       │
│  │                                                  │       │
│  │  Savings: $1,725/міс ($20,700/рік)               │       │
│  └──────────────────────────────────────────────────┘       │
│                                                              │
│  ┌──────────────────────────────────────────────────┐       │
│  │  2. Prompt Caching (економія 20-30%)             │       │
│  │                                                  │       │
│  │  System prompt (2K tokens) → cached              │       │
│  │  User profile + program context → cached         │       │
│  │  Therapy materials → cached                       │       │
│  │                                                  │       │
│  │  Savings: ~$50-100/міс                           │       │
│  └──────────────────────────────────────────────────┘       │
│                                                              │
│  ┌──────────────────────────────────────────────────┐       │
│  │  3. pgvector vs Pinecone (економія $100-200/міс) │       │
│  │                                                  │       │
│  │  Pinecone: $70-200/міс (managed vector DB)       │       │
│  │  pgvector: $0 (same PostgreSQL instance)          │       │
│  │                                                  │       │
│  │  Savings: $70-200/міс                            │       │
│  └──────────────────────────────────────────────────┘       │
│                                                              │
│  ┌──────────────────────────────────────────────────┐       │
│  │  4. Freemium Model (revenue offset)              │       │
│  │                                                  │       │
│  │  Free: 1 program, 5 AI sessions/week             │       │
│  │  Premium: unlimited, $9.99/міс                    │       │
│  │  Target: 1-5% purchase rate                       │       │
│  │  Revenue: 250-600 users × 1-5% × $9.99           │       │
│  │         = $25-300/міс (offset)                    │       │
│  └──────────────────────────────────────────────────┘       │
│                                                              │
│  Total Monthly Budget: $275-545                              │
│  vs Naive approach:    $2,000+                               │
│  Savings:              75-85%                                │
│  Провайдери: тільки Gemini + Claude (crisis)                 │
└──────────────────────────────────────────────────────────────┘
```

