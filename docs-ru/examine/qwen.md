# Qwen MVP-архитектурный review

**Дата:** 23 марта 2026  
**Reviewer:** Qwen  
**Статус:** ⚠️ Валидировано с критическими MVP-рисками

---

## 1. Executive Summary

Архитектура CeeVee **в целом солидна** и следует проверенным паттернам (Hexagonal, Monorepo, Adapter-first). Однако для MVP я идентифицирую **5 критических рисков**, угрожающих способности доставки, а также **3 конкретные оптимизации**, значительно увеличивающие скорость реализации.

---

## 2. Сильные стороны текущей архитектуры

| Сильная сторона | Оценка |
|--------|-----------|
| Гексагональное наслоение | ✅ Отлично – domain-логика чисто отделена от инфраструктуры |
| Структура монорепозитория | ✅ Хорошо – `packages/domain` и `packages/shared` позволяют ясные контракты |
| Отдельная backend-runtime | ✅ Правильно – MCP, скрапинг и AI требуют собственный runtime-контекст |
| Разделение Sync/Async | ✅ Продумано – модель job предусмотрена для длительных tasks |
| Интеграция pgvector | ✅ Прагматично – векторный поиск без дополнительной инфраструктуры |

---

## 3. Критические MVP-риски

### 3.1 ❌ Отсутствует auth-стратегия в MVP

**Проблема:**
Архитектура упоминает auth только как "будущую опцию" в `system-context.md`. MVP определен как "single-user", но:
- Supabase Auth не запланирован
- Session-handling в backend не определен
- Доступ к MCP-инструментам не имеет auth-границ

**Риск:**
Последующая установка auth потребует breaking changes во всех интерфейсах.

**Рекомендация:**
```
Приоритет: HIGH
Усилия: 2–3 дня (если реализовано рано)

→ Активировать Supabase Auth сразу в MVP (даже если только 1 user)
→ Определить валидацию session-token в backend как порт
→ Предоставить MCP-инструменты с auth-контекстом (даже если только 1 user)
```

---

### 3.2 ❌ Риск scraping-timeout не адресован

**Проблема:**
`runtime-observability.md` упоминает async-jobs, но:
- Queue-система не предусмотрена в MVP
- Скрапинг 50+ career pages может занять 5–10 минут
- HTTP-request-timeouts (типично 30s) сработают

**Риск:**
Users испытывают timeout-ошибки при discovery-потоках → продукт выглядит сломанным.

**Рекомендация:**
```
Приоритет: HIGH
Усилия: 1–2 дня

→ Запланировать BullMQ или In-Memory-queue с Redis-backup
→ Предусмотреть progress-polling для scraping-jobs во frontend
→ Лимитировать размеры scraping-batch (max 10 companies на job)
```

---

### 3.3 ❌ Resume-chunking-стратегия слишком размыта

**Проблема:**
`data-model.md` определяет `ResumeChunk`, но:
- Нет chunking-правил (размер, overlap, семантика)
- Нет метрики качества для chunk-границ
- Embedding-стратегия не специфицирована

**Риск:**
Плохой retrieval → плохие matches → продукт теряет доверие.

**Рекомендация:**
```
Приоритет: MEDIUM
Усилия: 1 день

→ Документировать chunking-правило (например 300–500 tokens, 50 token overlap)
→ Определить embedding-модель (например text-embedding-3-small)
→ Настроить Retrieval-Top-K и score-threshold как конфигурируемые параметры
```

---

### 3.4 ❌ MCP-deployment-стратегия неясна

**Проблема:**
`interfaces.md` определяет MCP-инструменты, но:
- Нет указания, как MCP-clients обнаруживают сервер
- Нет auth для MCP-транспорта
- Нет rate-limiting-стратегии для MCP-calls

**Риск:**
MCP-интеграция станет в MVP брешью безопасности или performance-узким местом.

**Рекомендация:**
```
Приоритет: MEDIUM
Усилия: 1 день

→ Эксплуатировать MCP-сервер как интегрированную часть apps/api (не отдельный процесс)
→ Валидировать MCP-tool-calls через backend-auth
→ Документировать rate-limits на MCP-инструмент
```

---

### 3.5 ❌ Обработка ошибок при ATS-скрапинге слишком оптимистична

**Проблема:**
`module-design.md` перечисляет ATS-адаптеры (Greenhouse, Lever, Workday, Ashby), но:
- Нет fallback-стратегии при CAPTCHA или IP-blocking
- Нет retry-логики с backoff
- Нет mock-данных для разработки без live-скрапинга

**Риск:**
Разработка останавливается, потому что скрапинг в окружении не работает.

**Рекомендация:**
```
Приоритет: HIGH
Усилия: 1–2 дня

→ Mock-адаптеры для всех ATS-провайдеров в development-режиме
→ Retry с экспоненциальным backoff (max 3 попытки)
→ Circuit-breaker для повторных scraping-ошибок
→ Ручные job-entry-fallbacks (URL + JSON-upload)
```

---

## 4. MVP-оптимизации (Quick Wins)

### 4.1 ✅ Zod-схемы как Single Source of Truth

**Текущее состояние:** TypeScript-интерфейсы в `packages/shared` без валидации.

**Оптимизация:**
```typescript
// packages/shared/schemas/resume.ts
import { z } from 'zod';

export const ResumeSchema = z.object({
  id: z.string().uuid(),
  userId: z.string().uuid(),
  title: z.string().min(1),
  content: z.string(),
  createdAt: z.coerce.date(),
  updatedAt: z.coerce.date(),
});

export type Resume = z.infer<typeof ResumeSchema>;
```

**Преимущества:**
- API-валидация автоматически генерируема
- MCP-JSON-schema выводима из Zod
- Frontend-валидация форм разделяема

**Усилия:** 1 день  
**Impact:** HIGH

---

### 4.2 ✅ MCP-сервер как интегрированная часть apps/api

**Текущее состояние:** MCP обозначен как отдельная поверхность.

**Оптимизация:**
```typescript
// apps/api/src/mcp/server.ts
import { MCPServer } from '@modelcontextprotocol/sdk';
import { discoverCompaniesHandler } from './handlers/discover-companies';

const server = new MCPServer({
  name: 'ceevee-api',
  version: '1.0.0',
});

server.registerTool('discover_companies', discoverCompaniesHandler);
server.registerTool('scrape_career_page', scrapeCareerPageHandler);
// ... другие инструменты

// В apps/api/src/index.ts
if (process.env.ENABLE_MCP) {
  attachMCPServer(server);
}
```

**Преимущества:**
- Нет отдельного deploy-процесса
- Та же auth-middleware что и HTTP
- Более простая локальная разработка

**Усилия:** 0.5 дня  
**Impact:** MEDIUM

---

### 4.3 ✅ Development-seed-скрипты для demo-данных

**Текущее состояние:** Нет seed-стратегии документировано.

**Оптимизация:**
```bash
# package.json scripts
{
  "scripts": {
    "db:seed:demo": "tsx scripts/seed-demo-data.ts",
    "db:seed:resumes": "tsx scripts/seed-sample-resumes.ts",
    "db:seed:companies": "tsx scripts/seed-sample-companies.ts"
  }
}
```

**Преимущества:**
- Немедленно тестируемый MVP без ручного ввода данных
- Demo для стейкхолдеров без live-скрапинга
- Консистентные тестовые данные для разработки

**Усилия:** 0.5 дня  
**Impact:** MEDIUM

---

## 5. Отсутствующие MVP-фичи (Gap-анализ)

| Фича | Статус в архитектуре | MVP-релевантность | Рекомендация |
|---------|----------------------|--------------|------------|
| **Auth (Supabase)** | ❌ Не определено | HIGH | Немедленно запланировать |
| **Resume-Upload (S3)** | ⚠️ Упомянуто, не специфицировано | HIGH | Использовать Supabase Storage |
| **Job-Queue** | ⚠️ Концепт, нет имплементации | HIGH | BullMQ или In-Memory |
| **Health-Check Endpoint** | ❌ Отсутствует | MEDIUM | `/health` для deployment |
| **Structured Logging** | ⚠️ Упомянуто, не специфицировано | MEDIUM | Pino или Winston |
| **Error-Tracking** | ❌ Отсутствует | LOW | Sentry позже добавляемо |
| **Rate-Limiting** | ❌ Отсутствует | MEDIUM | Express-Rate-Limit |

---

## 6. Конкретные следующие шаги

### Phase 1: Foundation (Неделя 1)
```
□ Настройка монорепозитория с Turborepo
□ Создать Supabase-проект (Auth + DB + Storage)
□ Определить Zod-схемы в packages/shared
□ Определить базовые порты в packages/domain
```

### Phase 2: Core-Features (Неделя 2–3)
```
□ Resume-upload с chunking
□ Company-discovery с mock-адаптерами
□ Scraping-queue с BullMQ
□ Match-engine с простым scoring
```

### Phase 3: MVP-Polish (Неделя 4)
```
□ Frontend-интеграция (Next.js)
□ MCP-сервер-подключение
□ Seed-скрипты для demo-данных
□ Deployment на Railway или Render
```

---

## 7. Вывод

**Общая оценка:** 🟢 **Хорошо с ясной потребностью оптимизации**

Архитектура **выше среднего хорошо продумана** для ранней стадии проекта. Гексагональная структура и ясное разделение ответственностей окупятся при масштабировании.

**Критично для MVP-доставки:**
1. Auth-стратегия должна быть определена рано (даже для single-user)
2. Scraping-timeouts требуют queue-системы
3. Resume-chunking нуждается в конкретных правилах
4. ATS-адаптеры требуют mock-fallbacks
5. MCP-deployment должен быть упрощен

**Рекомендуемая приоритизация:**
1. ✅ Auth + Queue-система (Неделя 1)
2. ✅ Zod-схемы + MCP-интеграция (Неделя 1)
3. ✅ Mock-адаптеры + Seed-скрипты (Неделя 2)

**Уровень риска без изменений:** 🔴 **MVP-доставка под угрозой** (scraping-timeouts, последующая установка auth)  
**Уровень риска с рекомендациями:** 🟢 **MVP доставим за 4 недели**

---

## 8. Приложение: Конкретные code-рекомендации

### 8.1 Queue-настройка (BullMQ)
```typescript
// apps/api/src/jobs/queue.ts
import { Queue, Worker } from 'bullmq';
import Redis from 'ioredis';

const redis = new Redis(process.env.REDIS_URL!);

export const scrapingQueue = new Queue('scraping', { connection: redis });
export const scrapingWorker = new Worker('scraping', async (job) => {
  const { companyUrl } = job.data;
  // Scraping-логика здесь
}, { connection: redis });
```

### 8.2 Auth-порт определение
```typescript
// packages/domain/ports/auth-port.ts
export interface AuthPort {
  validateSession(token: string): Promise<Session>;
  getUserById(userId: string): Promise<User>;
  createSession(userId: string): Promise<Session>;
}
```

### 8.3 Zod к MCP JSON-Schema
```typescript
// apps/api/src/mcp/schema-utils.ts
import { z } from 'zod';
import { zodToJsonSchema } from 'zod-to-json-schema';

export function toMCPInputSchema(schema: z.ZodType) {
  return zodToJsonSchema(schema, {
    target: 'jsonSchema7',
  });
}
```

---

**Конец review**
