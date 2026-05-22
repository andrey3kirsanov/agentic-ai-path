# LLM Fundamentals для Java-инженера
## Мини-курс на 4-5 часов: Tokens → Prompts → Hallucinations → Embeddings → Vector Store → RAG → Function Calling

---

## Модуль 1: Токены (30 минут)

### Что это

Токен — это минимальная единица, которую обрабатывает LLM. Модель не видит текст как строку символов — она разбивает его на куски (токены) и работает с ними как с числами.

### Как это работает

Предложение `"Hello, how are you?"` токенизатор разобьёт на: `["Hello", ",", " how", " are", " you", "?"]` — 6 токенов. Слово `"tokenization"` может стать двумя токенами: `["token", "ization"]`.

Каждый токен получает числовой ID из словаря модели. Предложение превращается в массив чисел, например `[1, 2, 3, 4, 5, 6]`. Модель работает только с этими числами.

### Почему это важно для разработчика

- **Стоимость**: API тарифицируется за токены (input + output). Чем длиннее промпт — тем дороже.
- **Context window**: у каждой модели есть лимит токенов на вход+выход. Claude — 200K токенов, GPT-4 Turbo — 128K. Если текст не помещается — его нужно обрезать или разбивать.
- **Правило для оценки**: 1 английское слово ≈ 1.3 токена. Для русского языка коэффициент выше (~2-3 токена на слово), потому что кириллица разбивается на субтокены.

### Аналогия для Java-разработчика

Токенизация — это как сериализация объекта в байты перед отправкой по сети. Текст → токены → числа → модель обрабатывает → числа → токены → текст. Те же encode/decode, просто другой домен.

### Что прочитать

- **[Understanding Tokens — Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/ai/conceptual/understanding-tokens)** — лаконичное объяснение с примерами разных методов токенизации (word, character, subword). Показывает как context window работает на практике.
- **[The Building Blocks of LLMs: Vectors, Tokens and Embeddings — The New Stack](https://thenewstack.io/the-building-blocks-of-llms-vectors-tokens-and-embeddings/)** — связывает токены с embeddings и векторами в одной статье. Хорошо для понимания полной цепочки.
- **[OpenAI Tokenizer](https://platform.openai.com/tokenizer)** — интерактивный инструмент. Вставь любой текст и посмотри, как он разбивается на токены. Попробуй русский текст vs английский.

---

## Модуль 2: Промпты / Prompt Engineering (45 минут)

### Что это

Промпт — это инструкция, которую ты отправляешь модели. Prompt engineering — это умение формулировать эти инструкции так, чтобы получать нужный результат.

### Структура промпта

Промпт через API состоит из трёх типов сообщений:

```
System: "Ты — senior Java-разработчик. Отвечай кратко, с примерами кода."
User: "Как реализовать retry с exponential backoff в Spring Boot?"
Assistant: (ответ модели)
```

- **System prompt** — задаёт роль, контекст, ограничения. Работает как конфигурация.
- **User prompt** — конкретный запрос.
- **Few-shot examples** — примеры input→output, которые показывают модели ожидаемый формат.

### Ключевые техники

**1. Будь конкретен:**
- Плохо: `"Напиши код для обработки ошибок"`
- Хорошо: `"Напиши Spring Boot @ControllerAdvice для обработки NotFoundException и ValidationException. Верни JSON с полями error, message, timestamp. Java 21."`

**2. Chain-of-Thought (CoT):**
Добавь `"Рассуждай пошагово"` — модель покажет ход мысли и даст более точный ответ.

**3. Few-shot examples:**
```
Input: "2024-01-15" → Output: "January 15, 2024"
Input: "2024-03-22" → Output: "March 22, 2024"
Input: "2024-12-01" → Output: ?
```
Модель поймёт паттерн и продолжит.

**4. Structured output:**
Попроси модель ответить в JSON/XML — легче парсить в коде.

### Аналогия для Java-разработчика

System prompt — это как application.yml: конфигурация, которая определяет поведение. User prompt — это HTTP-запрос. Few-shot examples — это тесты, которые показывают ожидаемое поведение. Temperature — это как уровень рандомизации: 0 = детерминированный ответ, 1 = креативный.

### Что прочитать

- **[Prompt Engineering Guide](https://www.promptingguide.ai/)** — самый полный открытый гайд. Покрывает всё: от zero-shot до tree-of-thought. Можно пройти за 30 минут, читая основные разделы (Introduction → Techniques → Applications).
- **[OpenAI: Best Practices for Prompt Engineering](https://help.openai.com/en/articles/6654000-best-practices-for-prompt-engineering-with-the-openai-api)** — практические рецепты с конкретными примерами промптов для разных задач.
- **[Anthropic Prompt Engineering Docs](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview)** — гайд от создателей Claude. Особенно полезны разделы про XML-теги и structured output.

---

## Модуль 3: Галлюцинации (20 минут)

### Что это

Галлюцинация — это когда LLM уверенно генерирует неправильную информацию. Модель не "врёт" намеренно — она предсказывает наиболее вероятный следующий токен, и иногда статистически вероятный ответ оказывается фактически неверным.

### Почему это происходит

LLM — это не база данных. Она не "знает" факты. Она обучена предсказывать, какой текст выглядит правдоподобно. Если спросить "кто написал книгу X?", модель выдаст ответ, который *выглядит* как правильный — даже если такой книги не существует.

Основные причины:
- **Устаревшие данные**: модель обучена до определённой даты и не знает, что произошло после
- **Пробелы в знаниях**: модель "заполняет пустоты" правдоподобным, но выдуманным текстом
- **Статистическая природа**: модель оптимизирована на правдоподобие, а не на точность

### Почему это критично для agentic AI

Если обычный чатбот галлюцинирует — пользователь прочитает неверный текст. Если галлюцинирует *агент* — он может вызвать API с неправильными параметрами, отправить неверные данные, или принять ошибочное решение в автоматическом режиме. Поэтому половина agentic-паттернов — это способы борьбы с галлюцинациями:

- **RAG** — дай модели реальные данные, чтобы она не придумывала
- **Reflection** — агент проверяет свой собственный ответ и исправляет ошибки
- **Verification** — второй агент валидирует результат первого
- **Grounding** — требуй от модели ссылаться на конкретные источники

### Аналогия для Java-разработчика

Галлюцинация — это как race condition: всё *обычно* работает правильно, но иногда выдаёт неожиданный результат. И как с race conditions — ты не можешь полностью их устранить, но можешь спроектировать систему так, чтобы минимизировать последствия (retry, validation, idempotency → reflection, RAG, verification).

### Что прочитать

- **[Anthropic: Reducing Hallucinations](https://docs.anthropic.com/en/docs/build-with-claude/reduce-hallucinations)** — практические техники от создателей Claude: цитирование источников, chain-of-thought, splitting complex tasks.
- **[OpenAI Cookbook: Techniques to Reduce Hallucinations](https://cookbook.openai.com/articles/techniques_to_improve_reliability)** — набор приёмов для повышения надёжности ответов.

---

## Модуль 4: Embeddings (45 минут)

### Что это

Embedding — это числовое представление смысла текста. Модель-энкодер берёт текст и превращает его в массив чисел (вектор), например из 1536 элементов. Похожие по смыслу тексты получают похожие векторы.

### Как это работает

```
"кот сидит на диване"  → [0.23, -0.41, 0.87, 0.12, ...]  (1536 чисел)
"кошка лежит на софе"  → [0.21, -0.39, 0.85, 0.14, ...]  (близкий вектор!)
"курс доллара растёт"  → [0.78, 0.55, -0.23, 0.91, ...]  (далёкий вектор)
```

Расстояние между первыми двумя векторами маленькое (смысл похожий), а между первым и третьим — большое.

### Cosine Similarity

Близость двух векторов измеряется через cosine similarity — значение от -1 до 1:
- **1.0** = идентичный смысл
- **0.0** = не связаны
- **-1.0** = противоположный смысл

В коде это просто скалярное произведение нормализованных векторов.

### Зачем это разработчику

Embeddings — это фундамент для:
- **Семантический поиск**: найти документы по смыслу, а не по ключевым словам
- **RAG**: найти релевантные куски данных для подстановки в промпт
- **Классификация**: определить категорию текста
- **Кластеризация**: сгруппировать похожие документы

### Аналогия для Java-разработчика

Embedding — это как hashCode(), только вместо одного числа ты получаешь 1536 чисел, и "коллизии" — это фича (похожие объекты дают похожие хеши). Вместо `equals()` ты используешь cosine similarity.

### Как вызвать API (пример)

```java
// Spring AI пример
EmbeddingResponse response = embeddingModel.call(
    new EmbeddingRequest(
        List.of("кот сидит на диване"),
        EmbeddingOptions.builder().build()
    )
);
float[] vector = response.getResult().getOutput(); // [0.23, -0.41, ...]
```

### Что прочитать

- **[An Intuitive Introduction to Text Embeddings — Stack Overflow Blog](https://stackoverflow.blog/2023/11/09/an-intuitive-introduction-to-text-embeddings/)** — лучшая статья для понимания интуиции за embeddings. Написана инженером для инженеров, без лишней математики.
- **[What Are Embeddings? — AWS](https://aws.amazon.com/what-is/embeddings-in-machine-learning/)** — краткий обзор с практическими примерами применения.
- **[Embeddings — Google ML Crash Course](https://developers.google.com/machine-learning/crash-course/embeddings)** — интерактивный курс от Google с визуализациями. Помогает "увидеть" как работает пространство эмбеддингов.

---

## Модуль 5: Vector Store (30 минут)

### Что это

Vector store (или vector database) — база данных, оптимизированная для хранения и поиска по векторам. Обычная БД ищет по точному совпадению (`WHERE name = 'X'`). Vector store ищет по близости (`найди 5 ближайших по смыслу`).

### Как это работает

1. Берёшь документ → разбиваешь на куски (chunks) по 500-1000 токенов
2. Каждый кусок пропускаешь через embedding model → получаешь вектор
3. Сохраняешь пару (текст, вектор) в vector store
4. При запросе: вопрос пользователя → embedding → поиск ближайших векторов → возвращаешь соответствующие тексты

### pgvector — для тех, кто уже знает PostgreSQL

```sql
-- Установка расширения
CREATE EXTENSION vector;

-- Таблица с векторной колонкой
CREATE TABLE documents (
    id SERIAL PRIMARY KEY,
    content TEXT,
    embedding vector(1536)  -- 1536-мерный вектор
);

-- Индекс для быстрого поиска
CREATE INDEX ON documents USING ivfflat (embedding vector_cosine_ops);

-- Поиск 5 ближайших по смыслу документов
SELECT content, 1 - (embedding <=> $query_vector) AS similarity
FROM documents
ORDER BY embedding <=> $query_vector
LIMIT 5;
```

`<=>` — оператор cosine distance. Чем меньше расстояние — тем ближе по смыслу.

### Варианты vector store

| Решение | Тип | Когда использовать |
|---------|-----|-------------------|
| **pgvector** | Расширение PostgreSQL | Уже есть PostgreSQL, до ~1M документов |
| **Pinecone** | Managed cloud | SaaS, не хочется управлять инфрой |
| **Weaviate** | Self-hosted / cloud | Нужны фильтры + векторный поиск |
| **Chroma** | In-memory / embedded | Прототипы, локальная разработка |
| **Redis** (с vector search) | In-memory | Уже используешь Redis |

### Аналогия для Java-разработчика

Vector store — это как Elasticsearch, только вместо full-text search по слову `"перевозка"` ты ищешь по смыслу `"можно ли лететь с собакой"` и находишь документ `"правила транспортировки домашних животных"`. Архитектурно — обычный data store, ничего нового.

### Что прочитать

- **[pgvector GitHub](https://github.com/pgvector/pgvector)** — документация расширения. Ты уже знаешь PostgreSQL — это самый быстрый путь начать. README покрывает установку, создание таблиц, индексов и запросов.
- **[What Are Vector Databases? — Pinecone Learning Center](https://www.pinecone.io/learn/vector-database/)** — подробный обзор архитектуры vector databases, алгоритмов поиска (HNSW, IVFFlat) и сравнение решений.

---

## Модуль 6: RAG — Retrieval-Augmented Generation (45 минут)

### Что это

RAG — это архитектурный паттерн, который позволяет LLM отвечать на вопросы, используя твои данные. Вместо того чтобы полагаться на знания модели (которые могут быть устаревшими или неполными), система сначала находит релевантные документы, а потом подставляет их в промпт.

### Как работает пайплайн

```
Пользователь: "Какие ограничения на перевозку животных?"
         │
         ▼
   ┌─────────────┐
   │  Embedding   │  Вопрос → вектор
   │   Model      │
   └──────┬──────┘
          │
          ▼
   ┌─────────────┐
   │ Vector Store │  Поиск ближайших документов
   │  (pgvector)  │
   └──────┬──────┘
          │  Возвращает 3-5 релевантных чанков
          ▼
   ┌─────────────────────────────────────┐
   │  Augmented Prompt:                  │
   │                                     │
   │  System: "Отвечай на основе         │
   │  предоставленного контекста."       │
   │                                     │
   │  Context: [чанк1] [чанк2] [чанк3]  │
   │                                     │
   │  User: "Какие ограничения на        │
   │  перевозку животных?"               │
   └──────────┬──────────────────────────┘
              │
              ▼
        ┌───────────┐
        │    LLM    │  Генерирует ответ на основе
        │  (Claude) │  реальных данных
        └───────────┘
```

### Три фазы

**1. Indexing (один раз):**
- Загрузи документы (PDF, Confluence, базу знаний)
- Разбей на чанки по 500-1000 токенов с overlap 100-200 токенов
- Сгенерируй embeddings для каждого чанка
- Сохрани в vector store

**2. Retrieval (каждый запрос):**
- Вопрос пользователя → embedding
- Поиск top-K ближайших чанков в vector store
- Опционально: re-ranking для повышения точности

**3. Generation (каждый запрос):**
- Собери промпт: system instruction + найденные чанки + вопрос
- Отправь в LLM
- Получи ответ, основанный на реальных данных

### Пример на Spring AI

```java
@RestController
public class ChatController {

    private final ChatClient chatClient;
    private final VectorStore vectorStore;

    @PostMapping("/ask")
    public String ask(@RequestBody String question) {
        // 1. Retrieval: поиск релевантных документов
        List<Document> docs = vectorStore.similaritySearch(
            SearchRequest.query(question).withTopK(5)
        );
        
        // 2. Augmentation: собираем контекст
        String context = docs.stream()
            .map(Document::getContent)
            .collect(Collectors.joining("\n\n"));
        
        // 3. Generation: отправляем в LLM
        return chatClient.prompt()
            .system("Отвечай только на основе предоставленного контекста.")
            .user(u -> u.text("""
                Context: {context}
                
                Question: {question}
                """)
                .param("context", context)
                .param("question", question))
            .call()
            .content();
    }
}
```

### Когда RAG, а когда fine-tuning?

| Критерий | RAG | Fine-tuning |
|----------|-----|-------------|
| Данные обновляются часто | ✅ Подходит | ❌ Нужно перетренировать |
| Нужны ссылки на источники | ✅ Легко | ❌ Сложно |
| Мало данных для обучения | ✅ Работает | ❌ Нужно много данных |
| Нужен специфический стиль/тон | ❌ Ограничен | ✅ Подходит |
| Стоимость | 💰 Дешевле | 💰💰💰 Дороже |

Для 90% задач RAG — правильный выбор. Fine-tuning нужен, когда модель должна "думать" по-другому, а не просто знать новые факты.

### Что прочитать

- **[What is RAG? — NVIDIA Blog](https://blogs.nvidia.com/blog/what-is-retrieval-augmented-generation/)** — лучшее введение в RAG. Объясняет зачем это нужно и как работает, без погружения в код.
- **[What is RAG? — AWS](https://aws.amazon.com/what-is/retrieval-augmented-generation/)** — практический обзор с акцентом на бизнес-применение. Хорошо объясняет проблему (LLM галлюцинирует) и решение (RAG).
- **[Spring AI Reference — RAG](https://docs.spring.io/spring-ai/reference/api/retrieval-augmented-generation.html)** — официальная документация Spring AI по RAG. Показывает как всё это реализовать именно в Spring Boot.

---

## Модуль 7: Function Calling / Tool Use (45 минут)

### Что это

Function calling — это механизм, который превращает LLM из генератора текста в агента, способного выполнять действия. Модель получает список доступных функций (tools), анализирует запрос пользователя и решает, какую функцию вызвать и с какими аргументами.

**Важно**: LLM не выполняет функцию сама. Она возвращает JSON с именем функции и аргументами. Выполнение — на стороне твоего кода.

### Как это работает

```
Пользователь: "Какая погода в Стамбуле?"
         │
         ▼
┌──────────────────────────────────────────────┐
│  Твой код отправляет в LLM:                  │
│  - Промпт: "Какая погода в Стамбуле?"        │
│  - Tools: [get_weather(location, unit)]      │
└──────────────────┬───────────────────────────┘
                   │
                   ▼
┌──────────────────────────────────────────────┐
│  LLM анализирует и возвращает:               │
│  {                                           │
│    "tool_call": {                            │
│      "name": "get_weather",                  │
│      "arguments": {                          │
│        "location": "Istanbul",               │
│        "unit": "celsius"                     │
│      }                                       │
│    }                                         │
│  }                                           │
└──────────────────┬───────────────────────────┘
                   │
                   ▼
┌──────────────────────────────────────────────┐
│  Твой код:                                   │
│  1. Парсит JSON                              │
│  2. Вызывает реальный Weather API            │
│  3. Получает: {"temp": 24, "sunny": true}    │
│  4. Отправляет результат обратно в LLM       │
└──────────────────┬───────────────────────────┘
                   │
                   ▼
┌──────────────────────────────────────────────┐
│  LLM генерирует финальный ответ:             │
│  "В Стамбуле сейчас 24°C, солнечно."        │
└──────────────────────────────────────────────┘
```

### Почему это фундамент agentic AI

Без function calling агент — это просто чат-бот. С function calling агент может:
- Читать и писать в базу данных
- Вызывать REST API
- Отправлять email/сообщения
- Искать в интернете
- Запускать код
- Управлять другими агентами

Agentic AI = LLM + Function Calling + Loop (агент вызывает функции, получает результаты, решает что делать дальше, вызывает следующие функции — и так пока задача не решена).

### Пример определения tools (Spring AI)

```java
@Bean
@Description("Get current weather for a given city")
public Function<WeatherRequest, WeatherResponse> getWeather() {
    return request -> weatherService.getWeather(
        request.location(), 
        request.unit()
    );
}

record WeatherRequest(String location, String unit) {}
record WeatherResponse(double temp, String condition) {}
```

Spring AI автоматически сгенерирует JSON-схему из Java record и передаст её модели. Модель вернёт JSON с аргументами → Spring AI вызовет функцию → результат отправится обратно в LLM.

### MCP — Model Context Protocol

MCP (от Anthropic) — это стандартизированный протокол для подключения tools к LLM. Вместо того чтобы хардкодить список функций, агент может динамически узнавать, какие tools доступны, через MCP-сервер. Думай о нём как об OpenAPI/Swagger, но для AI-агентов.

### Аналогия для Java-разработчика

Function calling — это как Dependency Injection для LLM. Ты определяешь интерфейсы (tools), регистрируешь их реализации (функции), и фреймворк (LLM) решает в рантайме, какую реализацию вызвать, исходя из контекста (промпта). MCP — это как Service Discovery (Eureka/Consul) для tools.

### Что прочитать

- **[Function Calling — Martin Fowler (martinfowler.com)](https://martinfowler.com/articles/function-call-LLM.html)** — отличная статья от Мартина Фаулера. Объясняет function calling, сравнивает с rules engines, покрывает MCP. Инженерный подход.
- **[Function Calling with LLMs — Prompt Engineering Guide](https://www.promptingguide.ai/applications/function_calling)** — практический гайд с примерами кода. Показывает полный цикл: определение функций → вызов → обработка результата.
- **[Function Calling — OpenAI Docs](https://developers.openai.com/api/docs/guides/function-calling)** — официальная документация OpenAI. Самое детальное описание протокола и best practices.
- **[Spring AI — Function Calling](https://docs.spring.io/spring-ai/reference/api/functions.html)** — как это делается в Spring AI. Java records, автогенерация схем, интеграция с Spring Boot.

---

## Модуль 8: Собираем всё вместе (30 минут)

### Полная картина

```
Пользователь
    │
    ▼
┌──────────────┐    ┌──────────────────┐
│   Промпт     │───▶│   Токенизация    │
│  (текст)     │    │  текст → токены  │
└──────────────┘    └───────┬──────────┘
                            │
    ┌───────────────────────┼─────────────────────┐
    │ RAG path              │                     │ Direct path
    │                       ▼                     │
    │              ┌────────────────┐             │
    │              │   Embedding    │             │
    │              │  токены →      │             │
    │              │  вектор [1536] │             │
    │              └───────┬────────┘             │
    │                      │                      │
    │                      ▼                      │
    │              ┌────────────────┐             │
    │              │  Vector Store  │             │
    │              │  (pgvector)    │             │
    │              │  → top-K docs  │             │
    │              └───────┬────────┘             │
    │                      │                      │
    │                      ▼                      │
    │              ┌────────────────┐             │
    │              │  Augmented     │             │
    │              │  Prompt        │             │
    │              │  docs+вопрос   │             │
    │              └───────┬────────┘             │
    │                      │                      │
    └──────────────────────┼──────────────────────┘
                           │
                           ▼
                  ┌────────────────┐
                  │      LLM      │
                  │   (Claude)    │
                  │  генерирует   │
                  │  ответ        │
                  └───────┬────────┘
                          │
                          ▼
                  ┌────────────────┐
                  │  Детокенизация │
                  │  токены → текст│
                  └────────────────┘
                          │
                          ▼
                     Ответ пользователю
```

### Маппинг на Java/Spring стек

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

### Что делать дальше

1. ✅ Эту статью ты прочитал — все базовые концепции покрыты
2. 🔜 **Andrew Ng "Agentic AI"** на DeepLearning.AI (бесплатно) — 4 паттерна: Reflection, Tool Use, Planning, Multi-Agent
3. 🔜 **Spring AI Masterclass** на Udemy — всё это на Java/Spring Boot с кодом
4. 🔜 IBM Java+Spring AI Specialization на Coursera — для сертификата в LinkedIn
5. 🔜 Свой pet-project: RAG + Function Calling поверх документации одного из проектов

---

## Все ссылки в одном месте

### Токены
- [Understanding Tokens — Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/ai/conceptual/understanding-tokens)
- [Building Blocks of LLMs — The New Stack](https://thenewstack.io/the-building-blocks-of-llms-vectors-tokens-and-embeddings/)
- [OpenAI Tokenizer (интерактивный)](https://platform.openai.com/tokenizer)

### Промпты
- [Prompt Engineering Guide (DAIR.AI)](https://www.promptingguide.ai/)
- [OpenAI: Best Practices](https://help.openai.com/en/articles/6654000-best-practices-for-prompt-engineering-with-the-openai-api)
- [Anthropic Prompt Engineering](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview)

### Галлюцинации
- [Anthropic: Reducing Hallucinations](https://docs.anthropic.com/en/docs/build-with-claude/reduce-hallucinations)
- [OpenAI Cookbook: Techniques to Improve Reliability](https://cookbook.openai.com/articles/techniques_to_improve_reliability)

### Embeddings
- [Intuitive Introduction — Stack Overflow Blog](https://stackoverflow.blog/2023/11/09/an-intuitive-introduction-to-text-embeddings/)
- [What Are Embeddings? — AWS](https://aws.amazon.com/what-is/embeddings-in-machine-learning/)
- [Google ML Crash Course — Embeddings](https://developers.google.com/machine-learning/crash-course/embeddings)

### Vector Store
- [pgvector — GitHub](https://github.com/pgvector/pgvector)
- [Vector Database Explained — Pinecone](https://www.pinecone.io/learn/vector-database/)

### RAG
- [What is RAG? — NVIDIA](https://blogs.nvidia.com/blog/what-is-retrieval-augmented-generation/)
- [What is RAG? — AWS](https://aws.amazon.com/what-is/retrieval-augmented-generation/)
- [Spring AI — RAG Docs](https://docs.spring.io/spring-ai/reference/api/retrieval-augmented-generation.html)

### Function Calling / Tool Use
- [Function Calling — Martin Fowler](https://martinfowler.com/articles/function-call-LLM.html)
- [Function Calling — Prompt Engineering Guide](https://www.promptingguide.ai/applications/function_calling)
- [Function Calling — OpenAI Docs](https://developers.openai.com/api/docs/guides/function-calling)
- [Spring AI — Function Calling](https://docs.spring.io/spring-ai/reference/api/functions.html)

### Следующий шаг: курсы
- [Andrew Ng — Agentic AI (бесплатно)](https://www.deeplearning.ai/courses/agentic-ai/)
- [Spring AI Masterclass — Udemy](https://www.udemy.com/course/spring-ai-masterclass-java-agentic-generative-mcp-rag-openai-ollama/)
- [IBM — Generative AI for Java/Spring — Coursera](https://www.coursera.org/specializations/generative-ai-for-java-and-spring-developers)