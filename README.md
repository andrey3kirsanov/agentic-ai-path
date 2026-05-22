# AI Agent Developer: Навыки и План Обучения
## Для Senior Java Backend Engineer

---

## 1. Что НЕ нужно знать

- Линейная алгебра, матрицы, градиенты
- PyTorch, TensorFlow
- Обучение моделей, fine-tuning на глубоком уровне
- Backpropagation, loss functions
- Архитектура transformer на математическом уровне
- Статистика, теория вероятностей
- Computer vision, NLP на уровне исследователя
- Jupyter notebooks, pandas, numpy

---

## 2. Полный список навыков

### 2.1 Фундамент — уже есть у Senior Java Backend Engineer

- Backend engineering, REST/WebSocket APIs
- Distributed systems, event-driven architecture
- AWS (Lambda, API Gateway, DynamoDB, SNS/SQS, Step Functions)
- Docker, CI/CD
- System design, production debugging

### 2.2 Понимание LLM (концептуально, без математики)

- Что такое токены, context window, temperature
- Как работает inference (вход → выход)
- Что такое embeddings (зачем, не как математически)
- Чем отличаются модели (Claude, GPT, Llama, Gemini)
- Что такое галлюцинации и почему они происходят
- Open-source vs closed-source модели, когда какие

### 2.3 Prompt Engineering

- System / user / assistant roles
- Few-shot, chain-of-thought, tree-of-thought
- Structured outputs (JSON mode)
- Prompt templates и переменные
- Prompt versioning

### 2.4 Function Calling / Tool Use

- Определение tools (JSON Schema)
- Цикл: LLM выбирает tool → твой код выполняет → результат обратно
- Параллельный вызов tools
- Error handling при tool failures
- MCP (Model Context Protocol)

### 2.5 RAG (Retrieval-Augmented Generation)

- Document loading, parsing
- Chunking strategies (fixed, semantic, recursive)
- Embeddings generation через API
- Vector store (pgvector, Pinecone, Chroma)
- Similarity search, hybrid search
- Re-ranking
- Evaluation качества retrieval

### 2.6 Agent Architecture

- Agent loop: perceive → reason → act → observe
- ReAct pattern (Reasoning + Acting)
- Planning — как агент декомпозирует задачу
- Memory: conversation (short-term), persistent (long-term), shared
- Multi-agent communication и delegation
- Orchestration patterns: sequential, parallel, hierarchical
- Guardrails — ограничения на действия агента
- Human-in-the-loop — когда агент спрашивает человека

### 2.7 Фреймворки (знать минимум 2)

- Spring AI (Java, основной стек)
- LangChain / LangGraph (Python, рыночный стандарт)
- CrewAI или OpenAI Agents SDK (один из двух)

### 2.8 Инфраструктура и Operations

- LLM API rate limits, retry, fallback между провайдерами
- Cost tracking — считать расход токенов, бюджетирование
- Observability — трассировка цепочек вызовов (LangSmith, Langfuse)
- Latency optimization — streaming, caching, параллелизм
- Security — prompt injection, data leakage, PII filtering

### 2.9 Evaluation и тестирование

- Как тестировать недетерминированные системы
- Eval datasets — golden sets для проверки качества
- Метрики: faithfulness, relevance, completeness
- A/B тестирование промптов
- Regression testing при смене модели/промпта

### 2.10 System Design для AI систем

- Когда RAG vs fine-tuning vs agents vs simple prompt
- Архитектура: sync vs async agent execution
- Scaling — очереди задач для агентов (Kafka/SQS)
- Fault tolerance — что если LLM API упал
- Data pipeline для indexing документов

---

## 3. План обучения

### Фаза 0: Базовые концепции LLM (4-5 часов)

Отдельный документ "LLM Fundamentals for Java Engineers" покрывает 8 модулей:

**Модуль 1 — Токены (30 мин).** Минимальная единица обработки LLM. 1 англ. слово ≈ 1.3 токена, русское ≈ 2-3. API тарифицируется за токены. Context window — лимит на вход+выход (Claude 200K, GPT-4 Turbo 128K).

**Модуль 2 — Промпты (45 мин).** System prompt (конфигурация) + user prompt (запрос) + few-shot examples (образцы). Техники: chain-of-thought, structured output, prompt templates.

**Модуль 3 — Галлюцинации (20 мин).** LLM предсказывает вероятный текст, не факты. Критично для агентов: агент не просто скажет неправду — он выполнит неправильное действие. RAG, Reflection, Verification — защита от галлюцинаций.

**Модуль 4 — Embeddings (45 мин).** Текст → вектор из 1536 чисел. Похожие по смыслу тексты = близкие векторы. Cosine similarity для измерения близости.

**Модуль 5 — Vector Store (30 мин).** База данных для векторов. pgvector = расширение PostgreSQL. Оператор `<=>` для cosine distance. Поиск по смыслу вместо поиска по ключевым словам.

**Модуль 6 — RAG (45 мин).** Indexing: документы → чанки → embeddings → vector store. Retrieval: вопрос → embedding → поиск ближайших. Generation: найденные чанки + вопрос → LLM → ответ.

**Модуль 7 — Function Calling (45 мин).** LLM получает список tools → анализирует запрос → возвращает JSON с именем функции и аргументами → твой код выполняет → результат обратно. MCP — стандарт подключения tools к LLM.

**Модуль 8 — Собираем всё вместе (30 мин).** Полная схема от запроса до ответа, маппинг на Java-стек.

### Фаза 1: Agentic AI паттерны (1-2 дня, бесплатно)

**DeepLearning.AI — "Agentic AI" от Andrew Ng**
- Четыре ключевых паттерна: Reflection, Tool Use, Planning, Multi-Agent Collaboration
- Vendor-neutral, на чистом Python, без привязки к фреймворкам
- https://www.deeplearning.ai/courses/agentic-ai/

**DeepLearning.AI — "AI Agents in LangGraph"**
- От создателя LangChain Harrison Chase
- Контролируемые агенты через LangGraph

### Фаза 2: Spring AI — hands-on на Java (1-2 недели)

**Udemy — "Spring AI Masterclass for Generative & Agentic AI Developers"**
- Cameron McKenzie, обновлён апрель 2026
- RAG, MCP, tool calling, structured outputs, memory, agentic workflows
- Всё на Spring Boot, project-driven
- https://www.udemy.com/course/spring-ai-masterclass-java-agentic-generative-mcp-rag-openai-ollama/

**Udemy — "Spring AI: Creating Workflows, Agents and Parsing Data"**
- Agentic workflows в Java
- AutoGPT-style системы, LangGraph-подобные архитектуры на Java

### Фаза 3: Сертификация (1 месяц)

**Coursera — IBM "Generative AI for Java and Spring Developers" Specialization**
- 4 курса: agentic AI, vibe coding, Spring AI интеграция
- Сертификат от IBM
- https://www.coursera.org/specializations/generative-ai-for-java-and-spring-developers

**Coursera — IBM "Building AI Agents and Agentic Workflows"**
- LangGraph, CrewAI, BeeAI, AG2 (AutoGen)
- Три курса по ~9 часов

**Coursera — Vanderbilt "AI Agent Developer" Specialization**
- Агентный фреймворк на Python с нуля
- Function calling, tool discovery, multi-agent collaboration с shared memory

### Фаза 4: Pet-проекты для портфолио

**Проект 1 — RAG-система (простой)**
RAG-чатбот поверх документации. Spring AI + pgvector + Claude API. Показывает: embeddings, vector store, RAG pipeline, Spring Boot integration.

**Проект 2 — Multi-agent система (средний)**
AI-слой для финансового трекера: агент парсит выписки, другой категоризирует расходы, третий генерирует отчёт. Показывает: multi-agent orchestration, function calling, tool use, agent communication.

**Проект 3 — Autonomous agent с MCP (продвинутый)**
Мультиагентная система на VPS с Telegram-интеграцией. Показывает: production deployment, MCP, memory management, cost optimization, monitoring.

Все проекты — на GitHub с хорошим README.

### Фаза 5: YouTube — Spring I/O 2026

- **"The Spring AI Ecosystem in 2026: From Foundations to Agents"** — Christian Tzolov, Dariusz Jędrzejczyk, Mark Pollack
- **"From Completions to Goals: AI Agents with Spring AI"** — Dr. Mark Pollack
- **"From Assistants to Agents: Self-Improving Agentic Systems with Spring AI"** — Christian Tzolov
- **"Comparing Agentic AI Frameworks for Java"** — Timo Salm, Sandra Ahlgrimm

Канал: https://www.youtube.com/@SpringIOConference

---

## 4. Маппинг Java → AI концепций

| LLM-концепция | Java-аналог |
|---------------|-------------|
| Токенизация | Сериализация (Jackson ObjectMapper) |
| Промпт | HTTP-запрос с headers (system) и body (user) |
| Галлюцинация | Race condition — обычно работает, иногда нет |
| Embedding | hashCode(), но многомерный |
| Vector Store | Elasticsearch, но по смыслу |
| RAG | Service layer: query DB → enrich request → call external API |
| Function Calling | Dependency Injection — LLM выбирает какую функцию вызвать |
| MCP | Service Discovery (Eureka/Consul) для AI tools |
| Context Window | Request size limit (как max HTTP body) |
| Temperature | Уровень рандомизации (0 = детерминированный) |
| Чанки | Pagination / streaming |
| Agent Loop | Event loop: получил задачу → вызвал tool → проверил результат → repeat |
| Multi-Agent | Microservices: каждый агент = сервис со своей ответственностью |
| Guardrails | Validation layer / circuit breaker |
| Prompt Versioning | API versioning (v1, v2) |