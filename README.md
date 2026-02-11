# RIQ: Правила, стандарти та план проєкту

## Папка Plans

`.cursor/plans/` — архітектурний план проєкту RIQ (v5):

- **`riq_architecture_mvp.plan.md`** — основний план MVP (19 секцій)
- **`RIQ_ARCHITECTURE_DIAGRAMS.md`** — архітектурні діаграми (15 діаграм)

**RIQ** — платформа терапевтичних програм з AI-помічником.

User Journey: Landing → Стартове опитування → Програма → Вправи → Рефлексії.

**Принципи**: Own your core (~500 LOC), Програми як центр продукту, Reasoning через Pydantic, MCP (17 tools), Hybrid RAG 85%+ з Citation ≥70%, Multi-model (Gemini + Claude crisis), 19 MVP метрик, Freemium, TDD.

**Стек**: FastAPI + PostgreSQL/pgvector + Supabase Auth | Flutter | Next.js | Gemini 2.0 + Claude 4.5 | RevenueCat + Stripe

---

## Rules

Правила та стандарти, що застосовані для створення архітектурного плану (Priority визначає порядок при конфліктах):


| Файл | Pri | Опис |
| ---- | --- | ---- |
| `ai-agent-best-practices.md` | 100 | BRIDGE Protocol, основи AI/ML, RAG, агенти, 4P Model, ROI |
| `sgr-agent-best-practices.md` | 95 | SGR Agent Core: модульна архітектура, execution cycle, TDD |
| `modern-ai-infrastructure-2026.md` | 95 | MCP, fastMCP, Prompt Caching, інфраструктура 2026 |
| `mcp-server-design-standards.md` | 92 | MCP tools: контракти, `get_capabilities()`, безпека, антипатерни |
| `prompt-engineering-mastery.md` | 90 | Промпт-інженерія: CoT, ToT, ReAct, Few-Shot, шаблони |
| `technical-standards.md` | 90 | Structured Output, Pydantic, Evaluation, Guardrails, Cost |
| `technical-implementation.md` | 85 | MLOps, RAG implementation, Monitoring, Security, Deployment |
| `business-decision-framework.md` | 75 | Build vs Buy, ROI, Impact vs Feasibility, Change Management |


---

## Ключові принципи

**Технічні**: Модульна архітектура (6 шарів) / Execution cycle: Reasoning → Action → Execution / Reasoning Tool Enforcement (Pydantic + Structured Output) / MCP: tool = одна інтенція, `get_capabilities()` обов'язковий / Hybrid RAG (Vector + BM25 + Reranker) / ETL як батарейки / "Own your core, integrate the rest" / RAGAS Eval обов'язковий / TDD: Red → Green → Refactor

**Бізнес**: BRIDGE Protocol / 4P Model (PoC → MVP → Pilot → Production) / ROI першочергово / Quick Wins (Impact ≥7, Feasibility ≥7) / 80/20 правило

---

## Посилання

- [SGR Agent Core](https://github.com/vamplabAI/sgr-agent-core) | [Docs](https://vamplabai.github.io/sgr-agent-core/)
- [MCP Protocol](https://modelcontextprotocol.io/) | [fastMCP](https://github.com/anthropics/fastmcp)
- [Google AI Studio](https://ai.google.dev/) | [Anthropic Docs](https://docs.anthropic.com/)
- [Pydantic](https://docs.pydantic.dev/) | [Cursor IDE](https://cursor.sh/)

**Джерела**: red_mad_robot (V. Kovalskii) — Production AI Systems (2026), SGR Agent Core (vamplabAI), офіційна документація Anthropic та Google.
