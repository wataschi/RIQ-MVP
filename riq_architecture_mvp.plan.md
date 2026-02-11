---
name: RIQ Architecture MVP v5
overview: |
  Архітектура платформи терапевтичних програм RIQ з AI-помічником.
  Програмно-центрична модель: стартове опитування → рекомендована програма → вправи → рефлексії.
  SGR Agent Core, BRIDGE Framework, Hybrid RAG (85%+), MCP, 19 метрик MVP.
todos:
  - id: infra-setup
    content: "Інфраструктура: Render, PostgreSQL+pgvector, Supabase Auth, n8n, Docker Compose"
    status: pending
  - id: landing-page
    content: "Landing Page: Next.js, демо-опитування, analytics, CTA"
    status: pending
  - id: agent-core
    content: "Agent Core: тришаровий execution cycle, ReasoningTool + program_context, Pydantic tools"
    status: pending
  - id: mcp-server
    content: "MCP Server: FastMCP з 17 tools (programs, surveys, exercises, reflections)"
    status: pending
  - id: rag-pipeline
    content: "RAG Pipeline: ETL батарейки, Hybrid Retrieval, Reranker, Citation tracking, Eval"
    status: pending
  - id: database
    content: "Database Schema: programs, exercises, reflections, surveys, subscriptions, analytics"
    status: pending
  - id: backend-api
    content: "Backend API: FastAPI, programs/surveys/subscriptions endpoints, guardrails + citation"
    status: pending
  - id: mobile-app
    content: "Mobile App: Flutter, programs, exercises, reflections, onboarding survey, subscription"
    status: pending
  - id: monetization
    content: "Монетизація: Stripe/RevenueCat, subscription model, paywall logic"
    status: pending
  - id: eval-platform
    content: "Eval Platform: RAGAS + Citation Rate + Hallucination Rate"
    status: pending
  - id: monitoring
    content: "Monitoring: 19 MVP метрик, Sentry + Prometheus + Grafana + Firebase Analytics"
    status: pending
  - id: security
    content: "Security: Guardrails (input+output+citation), PII masking, crisis detection, GDPR"
    status: pending
  - id: testing
    content: "Testing: TDD workflow, coverage >80%, program/exercise/survey tests"
    status: pending
  - id: pilot-launch
    content: "Pilot: 250-600 users, 19 метрик Go/No-Go, feedback loop"
    status: pending
isProject: true
---

# RIQ: Архітектура та план реалізації MVP v5

## 1. Стратегічний контекст

### BRIDGE Framework: Fast Bridge


| Параметр        | Значення                                                             |
| --------------- | -------------------------------------------------------------------- |
| **Impact**      | 8/10 — структуровані програми психопідтримки 24/7, AI-персоналізація |
| **Feasibility** | 8/10 — готові LLM, managed сервіси, Python + Flutter екосистема      |
| **Тип**         | Quick Win (Impact ≥7, Feasibility ≥7)                                |
| **4P Model**    | PoC (2 тижні) → MVP (2 міс) → Pilot (3 міс) → Production             |
| **ROI прогноз** | 200-300% через 12 місяців                                            |


### Продуктова модель

RIQ — **платформа структурованих терапевтичних програм з AI-помічником**. Центральна сутність — **Програма** (набір модулів з вправами), яку користувач проходить послідовно.

**User Journey:**

```
Landing Page → Демо-опитування → Завантаження додатку
→ Реєстрація → Стартове опитування (assessment)
→ AI рекомендує програму → Користувач обирає програму
→ Проходження модулів → Виконання вправ → Рефлексія після вправи
→ AI-помічник в контексті програми/вправи
→ Завершення програми → Рекомендація наступної
→ Mood tracking, journal, assessments — паралельно протягом усього шляху
```

### Функціональні вимоги

- **Програми**: каталог терапевтичних програм (CBT, mindfulness, emotion regulation), модулі з вправами
- **Стартове опитування**: психологічний assessment → AI-рекомендація програми
- **Вправи та рефлексії**: структуровані вправи в рамках програм, рефлексія після кожної вправи
- **AI-помічник**: чат в контексті поточної програми/вправи, з цитуванням джерел (≥70%)
- **Трекінг настрою**: mood, energy, anxiety tracking з візуалізацією
- **Психологічні тести**: PHQ-9, GAD-7, PSS (до/після програми)
- **Щоденник рефлексій**: вільні записи + прив'язані до вправ
- **Landing page**: лендінг з демо-опитуванням, конверсія в завантаження
- **Монетизація**: freemium модель (базовий контент безкоштовно, premium програми за підпискою)
- **Push-нагадування та weekly reports**

### Нефункціональні вимоги

- Response time <2s (p95), Uptime ≥99.5%, Error rate <1%
- RAG accuracy ≥85% (Hybrid + Reranker)
- Citation Rate ≥70%, Hallucination Rate <10%
- Crash-free sessions ≥97%
- Structured output: 100% (Pydantic enforcement)
- GDPR compliance

---

## 2. MVP Цілі та метрики (19 KPI)

### Acquisition


| Метрика                         | Target  | Формула                                             |
| ------------------------------- | ------- | --------------------------------------------------- |
| Кількість користувачів          | 250–600 | Унікальні зареєстровані                             |
| Conversion Rate лендингу        | 1–5%    | (завантаження + реєстрації) / унікальні відвідувачі |
| Completion Rate демо-опитування | 25–50%  | завершені / розпочаті демо-опитування               |


### Activation


| Метрика                       | Target | Формула                                          |
| ----------------------------- | ------ | ------------------------------------------------ |
| Стартове опитування           | 65–75% | % користувачів, що завершили стартове опитування |
| Вибір рекомендованої програми | 25–45% | % з ≥1 відкриттям рекомендованої програми        |
| Розпочата програма            | 40–60% | % з ≥1 розпочатою програмою                      |
| Взаємодія з AI-помічником     | 30–50% | % з ≥1 AI-сесією                                 |


### Engagement


| Метрика            | Target | Формула                                    |
| ------------------ | ------ | ------------------------------------------ |
| Day 7 Retention    | 25–35% | % з ≥1 активністю за 7 днів                |
| WAU/MAU            | 30–40% | Weekly Active Users / Monthly Active Users |
| Пройдено вправ     | 25–40% | % з ≥1 вправою/тиждень                     |
| Записано рефлексій | 20–35% | % з ≥1 рефлексією/пройдену вправу          |


### Retention & Value


| Метрика                  | Target       | Формула                                     |
| ------------------------ | ------------ | ------------------------------------------- |
| Day 30 Retention         | 12–22%       | % з ≥1 активністю за 30 днів                |
| Completion rate програми | 25–35%       | завершені / розпочаті програми              |
| Людино-тижні             | 8 000–12 000 | ∑ по всіх користувачах (тижні з активністю) |


### AI Quality


| Метрика            | Target | Формула                                               |
| ------------------ | ------ | ----------------------------------------------------- |
| CSAT               | ≥70%   | позитивні оцінки AI-сесій / загальна к-ть оцінок      |
| Citation Rate      | ≥70%   | % відповідей з посиланнями на матеріали/джерела       |
| Hallucination Rate | <10%   | тверджень без підтвердження / загальна к-ть тверджень |


### Business & Technical


| Метрика             | Target | Формула                               |
| ------------------- | ------ | ------------------------------------- |
| Purchase rate       | 1–5%   | підписки / загальна к-ть користувачів |
| Crash-free sessions | ≥97%   | crash-free сесії / всі сесії          |


---

## 3. Технологічний стек

### Build vs Buy


| Компонент                                 | Рішення                       | Обґрунтування                                    |
| ----------------------------------------- | ----------------------------- | ------------------------------------------------ |
| **Agent Core**                            | BUILD (~500 LOC)              | "Own your core, integrate the rest"              |
| **Backend API**                           | BUILD (FastAPI)               | Async, OpenAPI docs, AI ecosystem                |
| **LLM Inference**                         | BUY (API)                     | Gemini (основний) + Claude (тільки crisis)       |
| **Database**                              | BUILD (PostgreSQL + pgvector) | Всі дані + вектори в одній БД                    |
| **Auth (можливо у нас є власне рішення)** | BUY (Supabase Auth)           | JWT, social login, готове рішення                |
| **Automation**                            | BUY\BUILD (n8n)               | No-code workflows                                |
| **MCP Server**                            | BUILD (FastMCP)               | Стандартизований інтерфейс для AI tools          |
| **Mobile App**                            | BUILD (Flutter)               | Один codebase iOS+Android, висока якість UI      |
| **Landing Page**                          | BUILD (Next.js)               | SSR, SEO, швидкий FCP, легкий bundle             |
| **Payments**                              | BUY (RevenueCat + Stripe)     | In-app purchases iOS/Android + web subscriptions |


### Python Dependencies

```python
# Core
fastapi==0.115.0
pydantic==2.10.0
pydantic-settings==2.7.0
uvicorn[standard]==0.34.0

# Agent Core
fastmcp>=1.0.0            # MCP server
structlog>=24.0.0          # Structured logging

# Database
sqlalchemy==2.0.36
alembic==1.14.0
asyncpg==0.30.0
pgvector>=0.3.0

# AI/ML — Два провайдери: Gemini (основний) + Claude (crisis)
google-generativeai>=0.8.0  # Gemini 2.0 Pro/Flash + Embeddings
anthropic>=0.40.0           # Claude 4.5 Sonnet (ТІЛЬКИ crisis)
numpy>=1.24.0

# RAG (без langchain — own core)
markitdown>=0.1.0           # Document parsing (MIT)

# Eval
ragas>=0.2.0                # RAG evaluation
langsmith>=0.2.5            # Tracing

# Payments
stripe>=8.0.0               # Subscription webhooks (backend)

# Auth & Security
python-jose[cryptography]==3.3.0
passlib[bcrypt]==1.7.4

# Monitoring
prometheus-fastapi-instrumentator==7.0.0
sentry-sdk[fastapi]==2.19.2

# Testing
pytest>=8.0.0
pytest-asyncio>=0.24.0
httpx>=0.27.0
```

> **Примітка**: LangChain прибрано згідно з принципом "Own your core".
> Для RAG з 5 книг через pgvector достатньо asyncpg + pgvector напряму.

---

## 4. Agent Core

> SGR Agent Best Practices: тришаровий execution cycle,
> Reasoning Tool enforcement, Pattern Registry.

### 4.1 Модульна архітектура

```
Шар 1: Базові класи (BaseTool, BaseAgent, AgentContext)
Шар 2: Конфігурація (GlobalConfig, AgentDefinition — Pydantic + YAML)
Шар 3: Сервіси (LLMRouter, RAGService, GuardrailsService, ProgramService)
Шар 4: Агент (TherapyAgent — SGRToolCallingAgent)
Шар 5: Tools (MCP tools + internal Pydantic tools)
Шар 6: API (FastAPI endpoints, SSE streaming)
```

### 4.2 Тришаровий Execution Cycle

```python
while agent.state not in FINISH_STATES:
    # Фаза 1: Reasoning (Pydantic enforcement — НЕ промпт!)
    reasoning = await agent._reasoning_phase()

    # Фаза 2: Action Selection (Function Calling)
    action_tool = await agent._select_action_phase(reasoning)

    # Фаза 3: Action Execution
    result = await agent._action_phase(action_tool)
```

### 4.3 Reasoning Tool Enforcement

> LLM фізично НЕ МОЖЕ пропустити reasoning — гарантовано через Pydantic + Structured Output.

```python
class ReasoningTool(BaseTool):
    """Phase 1: LLM ЗОБОВ'ЯЗАНИЙ заповнити всі поля перед дією"""
    tool_name: ClassVar[str] = "reasoning"

    reasoning_steps: list[str]       # Кроки міркування
    current_situation: str           # Аналіз емоційного стану
    therapeutic_approach: str        # CBT, mindfulness...
    program_context: Optional[str]   # Поточна програма/модуль/вправа користувача
    task_completed: bool             # Агент сам вирішує коли стоп

    async def __call__(self, context: AgentContext, config: AgentConfig) -> str:
        context.last_reasoning = {
            "steps": self.reasoning_steps,
            "situation": self.current_situation,
            "approach": self.therapeutic_approach,
            "program": self.program_context
        }
        if self.task_completed:
            context.state = AgentStatesEnum.COMPLETING
        return f"Reasoning: {len(self.reasoning_steps)} steps"
```

**Результат**: точність 72% → 86%, утримання плану на 25+ кроків.

### 4.4 Resilience

- **Retry**: exponential backoff (3 спроби) для LLM calls, RAG, n8n
- **Circuit Breaker**: auto-disconnect failing сервісу (5 failures → OPEN → 60s → HALF_OPEN)
- **Fallback chain**: Gemini Flash → Gemini Pro → Claude → Error response

### 4.5 Context Optimization

- **Prompt Caching**: system prompt + user profile + program context (>1024 tokens) → cache
- **Sliding Window**: останні 20 повідомлень у повному вигляді
- **Summarization**: старіші повідомлення → стислий summary
- **Semantic Dedup**: видалення дублікатів з RAG результатів

### 4.6 Multi-Model LLM Router

```python
class LLMRouter:
    async def route(self, message: str, context: TherapyContext) -> LLMResponse:
        if context.crisis_detected or context.anxiety > 8:
            return await self._call_claude(message, context)      # 10%
        elif context.task_type in ("mood_check", "classification", "survey"):
            return await self._call_gemini_flash(message, context) # 70%
        else:
            return await self._call_gemini_pro(message, context)   # 20%
```


| Модель            | Використання                 | Context | Ціна (in/out 1M) | % запитів |
| ----------------- | ---------------------------- | ------- | ---------------- | --------- |
| Gemini 2.0 Flash  | Mood, classification, survey | 1M      | $0.10/$0.30      | 70%       |
| Gemini 2.0 Pro    | Терапевтичні сесії, програми | 2M      | $1.25/$10        | 20%       |
| Claude 4.5 Sonnet | Кризові ситуації             | 200K    | $3/$15           | 10%       |


---

## 5. MCP Server Design

> mcp-server-design-standards: 10-30 tools, get_capabilities(), ToolResponse.

### 5.1 Tools (17 інструментів)

```python
mcp = FastMCP("RIQ Therapy Assistant")

# Обов'язковий service tool
@mcp.tool()
def get_capabilities() -> dict:
    """Викликай ПЕРШИМ, щоб дізнатись доступні операції та обмеження."""
    return { "entities": [...], "constraints": {...}, "examples": [...] }

# Programs & Exercises
get_program_context()            # Поточна програма, модуль, вправа користувача
recommend_programs()             # Рекомендація програм на основі survey results
get_exercise_content()           # Контент вправи з інструкціями
save_reflection()                # Збереження рефлексії після вправи
get_user_program_progress()      # Прогрес по програмі (модулі, вправи, %)

# Therapy & RAG
search_therapy_materials()       # Hybrid RAG по 5 книгах, з citation metadata
save_therapy_insight()           # Збереження інсайту для RAG

# Analysis
analyze_mood_state()             # Mood, energy, anxiety, trends
detect_crisis_indicators()       # Multi-layer crisis check
search_user_history()            # RAG по історії користувача

# Assessments & Surveys
process_survey_response()        # Обробка стартового/демо опитування
get_assessment_results()         # PHQ-9, GAD-7 scores + trends

# System
get_session_context()            # 3-рівневий контекст + program state
trigger_notification()           # n8n webhook
create_weekly_report()           # Summary + program progress + recommendations
```

> `generate_therapeutic_exercise()` прибрано — вправи беруться з програм, а не генеруються AI.
> `get_user_progress()` замінено на `get_user_program_progress()` — прогрес прив'язаний до програми.

### 5.2 Стандартизована відповідь

```python
class ToolResponse(BaseModel):
    success: bool
    data: Optional[Any] = None
    error: Optional[str] = None
    metadata: Optional[dict] = None      # count, latency, source
    citations: Optional[list[Citation]] = None  # Для RAG-відповідей

class Citation(BaseModel):
    source: str          # "Книга: {title}"
    chapter: str
    page: Optional[int]
    relevance_score: float
```

**Security**: allowlist параметрів, scope-filtering per user, rate limiting 60/min, audit log.

---

## 6. RAG Pipeline: Hybrid (85%+)

### 6.0 Knowledge Base: 5 електронних книг з психології

Основне джерело знань — 5 електронних книг (перелік буде надано окремо).

**Вимоги до обробки**:

1. Парсинг: PDF/EPUB → чистий текст (MarkItDown)
2. Очистка: колонтитули, зміст, бібліографія
3. Чанкінг: семантичний, 512 tokens, overlap 50, збереження структури глав
4. Metadata: `{book_id, chapter, section, page, approach}`
5. Embedding: Gemini text-embedding-004 (768 dim)
6. Індексація: pgvector (IVFFLAT) + PostgreSQL FTS
7. Eval: RAGAS на тестових запитах після завантаження

**RAG sources (пріоритет)**:


| Пріоритет | Джерело      | Source tag             | Опис                                |
| --------- | ------------ | ---------------------- | ----------------------------------- |
| 1         | 5 книг       | `therapeutic_material` | Вправи, рекомендації, психоедукація |
| 2         | Історія чату | `chat_history`         | Персоналізація                      |
| 3         | Щоденник     | `journal`              | Контекст рефлексій                  |


### 6.1 ETL Pipeline (модульні "батарейки")

```
Document Parsing → Chunking → Embedding        → Indexing (pgvector)
  (MarkItDown)    (512 tok)  (Gemini emb-004)   (IVFFLAT + GIN)
```

> При зміні будь-якої батарейки → обов'язково запустити Eval!

### 6.2 Retrieval: Hybrid Search + Reranker + Citations

```python
async def retrieve(query: str, user_id: str, context: ProgramContext) -> RetrievalResult:
    # 1. Query enrichment (+ emotional context + program context)
    # 2. Vector search (pgvector cosine, k=20)
    # 3. BM25 search (PostgreSQL FTS, k=20)
    # 4. Merge + dedup (alpha=0.5)
    # 5. Reranker (cross-encoder, top_k=5)
    # 6. Extract citations (book_id, chapter, page) для кожного chunk
    return RetrievalResult(chunks=reranked, citations=extracted_citations)
```

**Citation Enforcement**: кожна відповідь AI, що спирається на RAG, ПОВИННА включати `citations` з metadata чанків. Output guardrail перевіряє наявність citations (target: ≥70% відповідей).

### 6.3 Eval (обов'язковий)

RAGAS + Citation метрики при будь-якій зміні RAG pipeline:

- **Faithfulness**: target >0.9
- **Answer Relevancy**: target >0.85
- **Context Precision**: target >0.8
- **Citation Rate**: target ≥70% (% відповідей з цитуванням)
- **Hallucination tracking**: eval results зберігаються в LangSmith + Prometheus metrics

---

## 7. 3-Рівнева Memory


| Рівень          | Джерело                                                   | Зберігання                              |
| --------------- | --------------------------------------------------------- | --------------------------------------- |
| **Short-term**  | Останні 20 повідомлень, mood, crisis flags, program state | Context window (in-memory)              |
| **Medium-term** | Журнал за 7 днів, trends, assessments, program progress   | PostgreSQL SELECT                       |
| **Long-term**   | 5 книг + історія + журнал + рефлексії                     | Hybrid RAG (pgvector + BM25 + Reranker) |


---

## 8. Backend API (FastAPI)

### 8.1 Структура проєкту

```
riq-backend/
├── app/
│   ├── api/v1/endpoints/
│   │   ├── auth.py              # Login, register, refresh
│   │   ├── chat.py              # AI chat (SSE streaming)
│   │   ├── programs.py          # Каталог, деталі, enroll, progress
│   │   ├── exercises.py         # Контент, start, complete
│   │   ├── reflections.py       # Create, list per exercise
│   │   ├── surveys.py           # Starting survey, demo survey
│   │   ├── assessments.py       # PHQ-9, GAD-7
│   │   ├── mood.py              # Mood tracking
│   │   ├── journal.py           # Journal entries
│   │   ├── progress.py          # Overall + program progress
│   │   ├── subscriptions.py     # Plans, checkout, webhooks
│   │   ├── analytics.py         # Event tracking + aggregation (retention, WAU/MAU, human-weeks)
│   │   └── privacy.py           # GDPR export/delete
│   ├── agent/
│   │   ├── therapy_agent.py     # TherapyAgent (SGRToolCallingAgent)
│   │   ├── tools/               # reasoning_tool, therapy_tools, program_tools, final_answer
│   │   ├── context.py           # AgentContext + ProgramState
│   │   └── config.py            # AgentConfig (Pydantic)
│   ├── mcp/                     # FastMCP server + 17 tools
│   ├── services/
│   │   ├── llm_router.py
│   │   ├── rag.py               # + citation extraction
│   │   ├── guardrails.py        # + citation enforcement
│   │   ├── crisis.py
│   │   ├── mood.py
│   │   ├── program.py           # Program logic, recommendations
│   │   ├── survey.py            # Survey processing, scoring
│   │   ├── subscription.py      # Stripe/RevenueCat webhooks
│   │   └── n8n.py
│   ├── db/                      # SQLAlchemy models, migrations (Alembic)
│   ├── schemas/                 # Pydantic request/response
│   ├── core/                    # config, security, metrics
│   └── main.py
├── tests/                       # TDD: agent, rag, programs, surveys, guardrails, api
├── eval/                        # RAGAS datasets + runner
├── prompts/                     # system_therapist.md, crisis_response.md, program_context.md
├── config.yaml
├── requirements.txt
├── docker-compose.yml
└── README.md
```

### 8.2 Конфігурація (YAML)

```yaml
llm:
  gemini:
    api_key: ${GEMINI_API_KEY}
    pro: gemini-2.0-pro
    flash: gemini-2.0-flash
    embedding: text-embedding-004     # 768 dim, embeddings для RAG
  claude:
    api_key: ${ANTHROPIC_API_KEY}
    model: claude-4.5-sonnet-20260201 # ТІЛЬКИ crisis situations

rag:
  chunk_size: 512
  chunk_overlap: 50
  retrieval_k: 20
  rerank_top_k: 5
  hybrid_alpha: 0.5
  citation_required: true             # Enforce citations in responses

agents:
  therapy_agent:
    base_class: "app.agent.therapy_agent.TherapyAgent"
    tools: [reasoning, get_program_context, search_therapy_materials,
            analyze_mood_state, detect_crisis_indicators,
            get_exercise_content, save_reflection, final_answer]
    llm: { default_model: gemini-pro, temperature: 0.7 }

programs:
  max_per_user: 3                     # Активних програм одночасно
  exercise_reminder_hours: 48         # Нагадування якщо не виконав вправу

subscriptions:
  provider: revenucat                 # RevenueCat для mobile + Stripe для web
  stripe_webhook_secret: ${STRIPE_WEBHOOK_SECRET}
  plans:
    free: { programs: 1, ai_sessions: 5/week }
    premium: { programs: unlimited, ai_sessions: unlimited, price: $9.99/month }
```

### 8.3 Guardrails (Input + Output + Citation)

**Input**: PII masking → Crisis detection (multi-layer, score > 8 → crisis protocol) → Prompt injection detection

**Output**:

- Hallucination check (grounded in context?)
- **Citation check** (відповідь містить посилання на джерела? target ≥70%)
- Diagnosis language filter (AI не лікар!)
- Toxicity check

### 8.4 Secrets Management

Всі секрети через `pydantic-settings` + env vars. На Render.com — зашифровані Environment Variables. `.env` тільки для dev, завжди в `.gitignore`.

---

## 9. Database Schema

### 9.1 Core Tables

```sql
CREATE EXTENSION vector;
CREATE EXTENSION pg_trgm;

-- Users
CREATE TABLE users (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    email varchar(255) UNIQUE NOT NULL,
    display_name varchar(100),
    preferences jsonb DEFAULT '{}',
    subscription_plan varchar(20) DEFAULT 'free',  -- free | premium
    created_at timestamp DEFAULT now(),
    gdpr_consent boolean DEFAULT false
);

-- AI Sessions
CREATE TABLE sessions (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id uuid REFERENCES users(id),
    started_at timestamp DEFAULT now(),
    ended_at timestamp,
    mood_start smallint,
    mood_end smallint,
    topics jsonb,
    program_id uuid REFERENCES programs(id)  -- AI сесія в контексті програми
);

-- Messages with citations
CREATE TABLE messages (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    session_id uuid REFERENCES sessions(id),
    role varchar(20) NOT NULL,  -- user | assistant
    content text NOT NULL,
    citations jsonb,            -- [{book_id, chapter, page, quote}]
    metadata jsonb,
    created_at timestamp DEFAULT now()
);
```

### 9.2 Programs Domain

```sql
-- Програми
CREATE TABLE programs (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    title varchar(200) NOT NULL,
    description text,
    approach varchar(50),         -- CBT, mindfulness, emotion_regulation
    duration_weeks smallint,      -- 4, 6, 8 тижнів
    difficulty varchar(20),       -- beginner, intermediate, advanced
    is_premium boolean DEFAULT false,
    cover_image_url varchar(500),
    created_at timestamp DEFAULT now()
);

-- Модулі програми (тижні/блоки)
CREATE TABLE program_modules (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    program_id uuid REFERENCES programs(id) ON DELETE CASCADE,
    title varchar(200) NOT NULL,
    description text,
    order_index smallint NOT NULL,
    UNIQUE(program_id, order_index)
);

-- Вправи
CREATE TABLE exercises (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    module_id uuid REFERENCES program_modules(id) ON DELETE CASCADE,
    title varchar(200) NOT NULL,
    type varchar(50),             -- breathing, journaling, thought_record, meditation, body_scan
    content_json jsonb NOT NULL,  -- Структурований контент вправи
    duration_min smallint,        -- Тривалість у хвилинах
    order_index smallint NOT NULL,
    UNIQUE(module_id, order_index)
);

-- Прогрес користувача по програмі
CREATE TABLE user_programs (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id uuid REFERENCES users(id),
    program_id uuid REFERENCES programs(id),
    status varchar(20) DEFAULT 'active',  -- active | completed | paused | abandoned
    started_at timestamp DEFAULT now(),
    completed_at timestamp,
    UNIQUE(user_id, program_id)
);

-- Прогрес по вправах
CREATE TABLE user_exercises (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id uuid REFERENCES users(id),
    exercise_id uuid REFERENCES exercises(id),
    user_program_id uuid REFERENCES user_programs(id),
    status varchar(20) DEFAULT 'pending',  -- pending | in_progress | completed | skipped
    started_at timestamp,
    completed_at timestamp
);

-- Рефлексії (прив'язані до вправ)
CREATE TABLE reflections (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id uuid REFERENCES users(id),
    user_exercise_id uuid REFERENCES user_exercises(id),
    content text NOT NULL,
    mood_after smallint,          -- 1-10 після вправи
    insights jsonb,               -- AI-extracted insights
    created_at timestamp DEFAULT now()
);
```

### 9.3 Surveys

```sql
-- Опитування (стартове, демо, assessment)
CREATE TABLE surveys (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    type varchar(30) NOT NULL,    -- starting | demo | assessment
    title varchar(200),
    questions_json jsonb NOT NULL,
    version smallint DEFAULT 1,
    is_active boolean DEFAULT true
);

-- Відповіді
CREATE TABLE survey_responses (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id uuid REFERENCES users(id),  -- NULL для демо (анонімне)
    survey_id uuid REFERENCES surveys(id),
    answers_json jsonb NOT NULL,
    score jsonb,                  -- Розрахований score
    completed_at timestamp DEFAULT now()
);

-- Рекомендації програм (на основі survey)
CREATE TABLE program_recommendations (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id uuid REFERENCES users(id),
    survey_response_id uuid REFERENCES survey_responses(id),
    program_id uuid REFERENCES programs(id),
    match_score float,            -- Релевантність 0-1
    selected boolean DEFAULT false,
    created_at timestamp DEFAULT now()
);
```

### 9.4 Subscriptions & Analytics

```sql
-- Підписки
CREATE TABLE subscriptions (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id uuid REFERENCES users(id) UNIQUE,
    plan varchar(20) NOT NULL,    -- free | premium
    status varchar(20),           -- active | cancelled | expired | trial
    provider varchar(20),         -- stripe | revenucat
    provider_subscription_id varchar(200),
    started_at timestamp DEFAULT now(),
    expires_at timestamp
);

-- Analytics events (tracking для 19 MVP метрик)
-- user_id може бути NULL для анонімних подій (landing, demo survey)
CREATE TABLE analytics_events (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id uuid REFERENCES users(id),       -- NULL для anonymous (landing)
    event_type varchar(100) NOT NULL,         -- landing_visitor, demo_survey_started,
                                              -- survey_completed, program_started,
                                              -- exercise_completed, reflection_written,
                                              -- ai_session_started, ai_session_rated,
                                              -- subscription_purchased, etc.
    metadata_json jsonb,
    created_at timestamp DEFAULT now()
);
CREATE INDEX idx_analytics_type ON analytics_events(event_type);
CREATE INDEX idx_analytics_user ON analytics_events(user_id, created_at);
CREATE INDEX idx_analytics_created ON analytics_events(created_at);  -- для retention queries
```

### 9.5 Existing Tables

```sql
-- Mood records
CREATE TABLE mood_records (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id uuid REFERENCES users(id),
    mood smallint,       -- 1-10
    energy smallint,     -- 1-10
    anxiety smallint,    -- 1-10
    notes text,
    created_at timestamp DEFAULT now()
);

-- Journal entries
CREATE TABLE journal_entries (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id uuid REFERENCES users(id),
    content text NOT NULL,
    mood_score smallint,
    tags jsonb,
    created_at timestamp DEFAULT now()
);

-- Assessments
CREATE TABLE assessments (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id uuid REFERENCES users(id),
    type varchar(20),     -- PHQ9, GAD7, PSS
    score smallint,
    interpretation varchar(50),
    created_at timestamp DEFAULT now()
);

-- Document embeddings (RAG)
CREATE TABLE document_embeddings (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id uuid REFERENCES users(id),  -- NULL для книг
    content text NOT NULL,
    embedding vector(768),     -- Gemini text-embedding-004
    metadata jsonb,       -- {book_id, chapter, page, approach}
    source varchar(50),   -- therapeutic_material | chat_history | journal | reflection
    book_id smallint,     -- 1-5 для книг
    created_at timestamp DEFAULT now()
);

-- Indexes
CREATE INDEX idx_emb_vector ON document_embeddings
    USING ivfflat (embedding vector_cosine_ops) WITH (lists = 100);
CREATE INDEX idx_emb_fts ON document_embeddings
    USING gin(to_tsvector('simple', content));
CREATE INDEX idx_emb_user ON document_embeddings(user_id);
CREATE INDEX idx_emb_source ON document_embeddings(source);
```

---

## 10. Mobile App (Flutter)

### Обґрунтування вибору


| Критерій           | Flutter                               | Чому важливо                                          |
| ------------------ | ------------------------------------- | ----------------------------------------------------- |
| **UI якість**      | Pixel-perfect, Material 3 + Cupertino | Mental health app потребує красивий, заспокійливий UI |
| **Продуктивність** | Компілюється в native ARM             | Плавні анімації, швидкий запуск                       |
| **Один codebase**  | iOS + Android                         | Менше розробки, менше багів                           |
| **Dart**           | Строго типізований, null-safe         | Менше runtime помилок                                 |
| **Екосистема**     | pub.dev 50K+ пакетів                  | Всі необхідні пакети є                                |
| **Hot reload**     | Subsecond reload                      | Швидка ітерація UI                                    |


### Core залежності

```yaml
# pubspec.yaml
dependencies:
  flutter:
    sdk: flutter

  # Navigation & State
  go_router: ^14.0.0              # Declarative routing
  flutter_riverpod: ^2.5.0        # State management
  riverpod_annotation: ^2.3.0

  # UI
  flutter_material_design_icons: ^1.0.0
  fl_chart: ^0.68.0               # Mood/progress charts
  flutter_animate: ^4.5.0         # Smooth animations
  step_progress_indicator: ^1.0.0  # Program/exercise progress

  # Networking
  dio: ^5.4.0                     # HTTP client
  web_socket_channel: ^2.4.0      # SSE streaming

  # Local Storage
  shared_preferences: ^2.2.0
  sqflite: ^2.3.0                 # Offline cache

  # Auth
  supabase_flutter: ^2.5.0        # Supabase Auth + DB

  # Notifications
  firebase_messaging: ^15.0.0     # Push notifications
  flutter_local_notifications: ^17.0.0

  # In-App Purchases
  purchases_flutter: ^7.0.0       # RevenueCat SDK

  # Forms & Validation
  reactive_forms: ^17.0.0

  # Analytics
  firebase_analytics: ^11.0.0     # Event tracking for 19 MVP metrics

  # Utils
  intl: ^0.19.0                   # i18n
  freezed_annotation: ^2.4.0      # Immutable models

dev_dependencies:
  flutter_test:
    sdk: flutter
  build_runner: ^2.4.0
  freezed: ^2.5.0
  riverpod_generator: ^2.4.0
  mockito: ^5.4.0
  integration_test:
    sdk: flutter
```

### Структура проєкту

```
riq-mobile/
├── lib/
│   ├── main.dart
│   ├── app/
│   │   ├── router.dart              # GoRouter configuration
│   │   └── theme.dart               # Calm, therapeutic theme
│   ├── features/
│   │   ├── auth/                    # Login, Register
│   │   ├── onboarding_survey/       # Стартове опитування (multi-step wizard)
│   │   ├── programs/                # Каталог програм, деталі, enrolled
│   │   ├── exercises/               # Виконання вправи, таймер, інструкції
│   │   ├── reflections/             # Рефлексія після вправи
│   │   ├── chat/                    # AI chat в контексті програми (SSE)
│   │   ├── mood/                    # Mood tracking + charts
│   │   ├── journal/                 # Reflection journal
│   │   ├── assessments/             # PHQ-9, GAD-7, PSS
│   │   ├── progress/                # Program progress + overall insights
│   │   └── subscription/            # Paywall, pricing, RevenueCat
│   ├── core/
│   │   ├── api/                     # Dio client, interceptors
│   │   ├── models/                  # Freezed data models
│   │   ├── providers/               # Riverpod providers
│   │   ├── analytics/               # Event tracking helper
│   │   └── utils/
│   └── shared/
│       ├── widgets/                 # Reusable UI components
│       └── constants/
├── test/                            # Unit + widget tests
├── integration_test/                # E2E tests
├── pubspec.yaml
└── README.md
```

### Ключові екрани

1. **Onboarding Survey**: Multi-step wizard (5-10 питань) → AI-рекомендація програми
2. **Home**: поточна програма + progress bar + quick actions + daily tip
3. **Program Catalog**: список програм, фільтр за підходом, premium badge
4. **Program Detail**: опис, модулі, тривалість, CTA "Розпочати"
5. **Exercise**: контент вправи, таймер (для meditation), інструкції, CTA "Завершити"
6. **Reflection**: prompt після вправи, текстове поле, mood rating
7. **Chat**: AI в контексті програми, streaming, citations, crisis banner
8. **Mood Tracker**: emotion wheel + 1-10 scale + notes → fl_chart trends
9. **Journal**: rich text entries + tags + mood association
10. **Assessments**: PHQ-9/GAD-7 flow + score visualization
11. **Progress**: program completion %, weekly trends, milestones
12. **Subscription**: pricing, free vs premium comparison, RevenueCat checkout

### Analytics Events (для MVP метрик)

```dart
// Ключові events для tracking 19 метрик
analytics.logEvent('survey_started');            // → Starting survey metric
analytics.logEvent('survey_completed');           // → Starting survey metric
analytics.logEvent('program_recommended_viewed'); // → Program recommendation metric
analytics.logEvent('program_started');            // → Program started metric
analytics.logEvent('exercise_completed');          // → Exercise completed metric
analytics.logEvent('reflection_written');          // → Reflection metric
analytics.logEvent('program_completed');           // → Program completion metric
analytics.logEvent('ai_session_started');          // → AI interaction metric
analytics.logEvent('ai_session_rated', {'rating': 'positive'}); // → CSAT metric
analytics.logEvent('subscription_purchased');      // → Purchase rate metric
```

---

## 11. Landing Page

### Архітектурне рішення

**Next.js** (не Flutter Web) — оптимально для лендінгу:


| Критерій        | Next.js  | Flutter Web     |
| --------------- | -------- | --------------- |
| **Bundle size** | <100KB   | ~2MB            |
| **SEO**         | SSR/SSG  | Погане (canvas) |
| **FCP**         | <1s      | 3-5s            |
| **Вартість**    | Vercel 0 | Потребує CDN    |


### Структура

```
riq-landing/
├── src/
│   ├── app/
│   │   ├── page.tsx               # Hero + Features + CTA
│   │   ├── demo-survey/page.tsx   # Демо-опитування (embedded)
│   │   └── layout.tsx
│   ├── components/
│   │   ├── Hero.tsx
│   │   ├── Features.tsx
│   │   ├── DemoSurvey.tsx         # Multi-step mini-survey (3-5 питань)
│   │   ├── Testimonials.tsx
│   │   └── CTA.tsx
│   └── lib/
│       └── api.ts                 # Calls to FastAPI backend
├── package.json
└── next.config.js
```

### Функціонал

- **Hero**: ціннісна пропозиція, CTA "Спробувати безкоштовно"
- **Демо-опитування**: 3-5 питань (спрощена версія стартового), результат → CTA завантажити додаток
- **Features**: програми, AI-помічник, mood tracking, приватність
- **Social proof**: відгуки, статистика
- **Store links**: App Store + Google Play
- **Analytics**: UTM tracking → backend `analytics_events` table:
  - `landing_visitor` (anonymous, user_id=NULL)
  - `demo_survey_started`, `demo_survey_completed`
  - `app_download_clicked`, `signup_from_landing`

---

## 12. Monitoring та Observability

### MVP Метрики Dashboard (19 KPI)

**Acquisition Dashboard**:

- Users (registered) → target 250–600
- Landing visitors → conversion 1–5%
- Demo survey starts → completion 25–50%

**Activation Dashboard**:

- Starting survey → completion 65–75%
- Program recommendation → selection 25–45%
- Programs → started 40–60%
- AI sessions → interaction rate 30–50%

**Engagement Dashboard**:

- Day 7 Retention → 25–35%
- WAU/MAU ratio → 30–40%
- Exercises → completed/week 25–40%
- Reflections → written/exercise 20–35%

**Retention Dashboard**:

- Day 30 Retention → 12–22%
- Program completion rate → 25–35%
- Human-weeks → 8 000–12 000

**AI Quality Dashboard**:

- CSAT (% positive AI ratings) → ≥70%
- Citation Rate → ≥70%
- Hallucination Rate → <10%

**Business & Tech Dashboard**:

- Purchase rate → 1–5%
- Crash-free sessions → ≥97%
- Response p95 → <2s
- Uptime → ≥99.5%
- Error rate → <1%
- LLM cost → tracking daily

### ML Metrics

- RAG faithfulness >0.9 (RAGAS)
- RAG answer relevancy >0.85
- RAG context precision >0.8
- Crisis detection accuracy >95%

### Alerting


| Alert                  | Condition        | Severity           |
| ---------------------- | ---------------- | ------------------ |
| CrisisDetectionFailure | Будь-яка помилка | P0 — Slack + SMS   |
| HighErrorRate          | >5% за 5 хв      | P1 — Slack + Email |
| HighCrashRate          | <95% crash-free  | P1 — Slack + Email |
| SlowResponse           | p95 >5s за 10 хв | P2 — Slack         |
| HighAICost             | >$100/day        | P2 — Email         |
| LowCitationRate        | <50% за день     | P2 — Slack         |


### Інструменти

- **Prometheus + Grafana**: dashboards для всіх 19 метрик
- **Sentry**: crash tracking (crash-free sessions), error monitoring
- **Firebase Analytics**: user events, funnels, retention, WAU/MAU
- **structlog**: structured JSON logs
- **LangSmith**: LLM tracing, prompt versions, eval runs

---

## 13. Testing (TDD)

> Red → Green → Refactor для кожної фічі.

### Ключові тести

```python
# Agent: reasoning with program context
async def test_reasoning_includes_program_context(agent):
    context = AgentContext(program=active_program, exercise=current_exercise)
    reasoning = await agent._reasoning_phase()
    assert isinstance(reasoning, ReasoningTool)
    assert reasoning.program_context is not None

# RAG: citation extraction
async def test_retrieval_returns_citations():
    result = await rag.retrieve("техніка дихання при тривозі", user_id)
    assert len(result.citations) > 0
    assert all(c.source for c in result.citations)

# Programs: enrollment flow
async def test_user_can_start_program():
    program = await program_service.enroll(user_id, program_id)
    assert program.status == "active"
    exercises = await program_service.get_exercises(program.id)
    assert len(exercises) > 0

# Surveys: recommendation flow
async def test_survey_generates_recommendations():
    response = await survey_service.submit(user_id, survey_id, answers)
    recommendations = await survey_service.get_recommendations(response.id)
    assert len(recommendations) >= 1
    assert all(r.match_score > 0 for r in recommendations)

# Guardrails: citation enforcement
async def test_citation_enforcement():
    response_with_citations = "Згідно з CBT підходом [Книга 1, Глава 3]..."
    result = await guardrails.validate_output(response_with_citations)
    assert result.citation_present == True

# Subscription: paywall check
async def test_premium_program_requires_subscription():
    with pytest.raises(PaywallError):
        await program_service.enroll(free_user_id, premium_program_id)
```

**Targets**: Coverage >80%, всі critical paths покриті.

---

## 14. n8n Workflows


| Workflow            | Тригер             | Дія                                                |
| ------------------- | ------------------ | -------------------------------------------------- |
| Daily Reminder      | Schedule 9:00      | Push → users без check-in 24h                      |
| Program Reminder    | Schedule 10:00     | Push → users без exercise 48h в активній програмі  |
| Exercise Completion | Webhook (exercise) | Push → "Напишіть рефлексію" prompt                 |
| Program Completion  | Webhook (program)  | Congratulations + recommend next program           |
| Weekly Report       | Schedule Mon 10:00 | Program progress + mood trends → Email             |
| Crisis Alert        | Webhook (FastAPI)  | Slack + Email admin + Log                          |
| Onboarding          | Webhook (new user) | 3-email sequence (1h, 24h, 72h)                    |
| Subscription Expiry | Schedule daily     | Email → users з підпискою що закінчується за 3 дні |
| Backup              | Schedule 3:00 AM   | pg_dump → S3 → cleanup >30d                        |


---

## 15. Масштабування (roadmap)

**MVP** (250–600 users): API calls (Gemini + Claude crisis) — $275-545/міс

**Pilot** (1000–5000 users): + prompt caching — $600-1400/міс

**Production** (>5000 users): vLLM + XGrammar self-hosted — фіксовано $200-400/міс + API fallback для crisis

> vLLM + XGrammar: 100% structured output guarantee через constrained decoding. Підтримка Qwen-3, Llama-4, gpt-oss.

---

## 16. Бюджет MVP

```
Infrastructure:
├── Render Backend:              $7/міс
├── Render PostgreSQL+pgvector:  $7/міс
├── Supabase Auth:               $25/міс
├── n8n Cloud:                   $20/міс
├── Vercel (Landing):            $0/міс (free tier)

AI Services (2 провайдери):
├── Gemini Flash (70%):          $50-100/міс
├── Gemini Pro (20%):            $100-200/міс
├── Gemini Embeddings:           $10-30/міс
├── Claude Sonnet (crisis 10%):  $30-80/міс

Payments:
├── RevenueCat:                  $0/міс (free до $2.5K MRR)
├── Stripe (2.9% + $0.30):      ~$0-50/міс (від purchases)

Monitoring:
├── Sentry:                      $26/міс
├── Grafana Cloud:               $0 (free tier)
├── Firebase Analytics:          $0 (free tier)

═══════════════════════════════
TOTAL:                           $275-545/міс
```

---

## 17. Етапи реалізації (4P Model)

### PoC — 2 тижні ($200)

- Базовий чат з Gemini Pro (без RAG)
- Тестовий промпт терапевта, 10 діалогів, manual review
- Прототип стартового опитування (Typeform/Google Forms)
- **Go/No-Go**: Response quality ≥3.5/5

### MVP — 2 місяці ($3-5K)

- Agent Core з Reasoning Tool enforcement + program context
- MCP Server з 17 tools (programs, surveys, exercises)
- Hybrid RAG pipeline + Citation tracking + Eval (5 книг)
- Database: повна schema (programs, exercises, reflections, surveys, subscriptions)
- Flutter app (onboarding survey, programs, exercises, reflections, chat, mood, subscription)
- Landing page (Next.js) з демо-опитуванням
- Guardrails (crisis + PII + hallucination + citation)
- n8n workflows, monitoring (Sentry + Firebase Analytics)
- **Go/No-Go (ключові з 19 метрик)**:
  - Users: 250–600
  - Day 7 Retention: 25–35%
  - Starting survey completion: 65–75%
  - Program started: 40–60%
  - AI interaction: 30–50%
  - CSAT: ≥70%
  - Citation Rate: ≥70%
  - Crash-free: ≥97%

### Pilot — 3 місяці ($5-8K)

- Всі програми, production monitoring (Grafana dashboards з 19 KPI)
- A/B testing промптів та програм
- Load testing, security audit + GDPR
- Subscription optimization
- **Go/No-Go (повний набір 19 метрик)**:
  - Day 30 Retention: 12–22%
  - WAU/MAU: 30–40%
  - Exercises completed: 25–40%
  - Reflections: 20–35%
  - Program completion: 25–35%
  - Hallucination: <10%
  - Purchase rate: 1–5%
  - Human-weeks: 8 000–12 000

### Production — Ongoing

- vLLM + XGrammar, auto-scaling, multi-language, B2B

---

## 18. Ризики та мітігація


| #   | Ризик                           | Prob | Impact   | Мітігація                                        |
| --- | ------------------------------- | ---- | -------- | ------------------------------------------------ |
| 1   | Низька якість AI відповідей     | 40%  | Critical | Reasoning Tool, Eval, A/B промптів, citations    |
| 2   | Ethical/Legal проблеми          | 30%  | Critical | Guardrails, disclaimers, психолог-консультант    |
| 3   | Висока вартість AI              | 50%  | High     | Multi-model routing, prompt caching, Flash 70%   |
| 4   | Низька retention                | 60%  | High     | Програми з прогресом, personalization, reminders |
| 5   | Data breach                     | 20%  | Critical | Encryption, GDPR, PII masking, audit logs        |
| 6   | Низький completion rate програм | 50%  | High     | UX оптимізація, нагадування, gamification        |
| 7   | Низький citation rate AI        | 40%  | Medium   | Citation enforcement guardrail, RAG tuning       |
| 8   | Конверсія лендінгу <1%          | 40%  | Medium   | A/B тестування, демо-опитування як hook          |


---

## 19. Висновки

### Архітектурні принципи

1. **"Own your core"** — Agent Core ~500 LOC, без LangChain
2. **Програми як центр** — структуровані терапевтичні програми, не просто чат
3. **Reasoning через типізацію** — Pydantic enforcement + program_context
4. **MCP як контракт** — 17 tools, get_capabilities(), ToolResponse + Citations
5. **RAG як батарейки** — модульний ETL, Hybrid + Reranker = 85%+, Citation ≥70%
6. **Eval обов'язковий** — RAGAS + Citation Rate при будь-якій зміні pipeline
7. **TDD** — Red → Green → Refactor
8. **3-рівнева memory** — Short + Medium + Long-term (RAG)
9. **Multi-model routing** — правильна модель під задачу
10. **Guardrails** — input + output + citation enforcement
11. **Flutter** — pixel-perfect UI для mental health
12. **19 MVP метрик** — кожна має DB, API, Flutter та Monitoring покриття

### ROI прогноз

- **Витрати** (6 місяців): ~$6,000
- **Потенціал**: 500 users x $10/міс = $5,000/міс (при purchase rate 5%)
- **ROI (12 місяців)**: 200-300%
- **Payback**: ~6-8 місяців

**Рекомендація**: GO