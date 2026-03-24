# Qwen MVP-Architektur-Review

**Datum:** 23. März 2026  
**Reviewer:** Qwen  
**Status:** ⚠️ Validiert mit kritischen MVP-Risiken

---

## 1. Executive Summary

Die CeeVee-Architektur ist **grundsätzlich solide** und folgt bewährten Mustern (Hexagonal, Monorepo, Adapter-first). Für ein MVP identifiziere ich jedoch **5 kritische Risiken**, die die Lieferfähigkeit gefährden, sowie **3 konkrete Optimierungen**, die die Umsetzungsgeschwindigkeit deutlich erhöhen.

---

## 2. Stärken der aktuellen Architektur

| Stärke | Bewertung |
|--------|-----------|
| Hexagonale Schichtung | ✅ Exzellent – Domain-Logik ist sauber von Infrastruktur getrennt |
| Monorepo-Struktur | ✅ Gut – `packages/domain` und `packages/shared` ermöglichen klare Verträge |
| Separate Backend-Runtime | ✅ Richtig – MCP, Scraping und AI benötigen eigenen Runtime-Kontext |
| Sync/Async-Trennung | ✅ Durchdacht – Job-Modell für langlaufende Tasks vorgesehen |
| pgvector-Integration | ✅ Pragmatisch – Vektorsuche ohne zusätzliche Infrastruktur |

---

## 3. Kritische MVP-Risiken

### 3.1 ❌ Fehlende Auth-Strategie im MVP

**Problem:**
Die Architektur erwähnt Auth nur als "zukünftige Option" in `system-context.md`. Das MVP ist als "single-user" definiert, aber:
- Supabase-Auth ist nicht eingeplant
- Session-Handling im Backend ist undefiniert
- MCP-Tool-Zugriff hat keine Auth-Grenzen

**Risiko:**
Nachträglicher Auth-Einbau erfordert Breaking Changes an allen Interfaces.

**Empfehlung:**
```
Priorität: HIGH
Aufwand: 2–3 Tage (wenn früh implementiert)

→ Supabase Auth sofort im MVP aktivieren (auch wenn nur 1 User)
→ Session-Token-Validierung im Backend als Port definieren
→ MCP-Tools mit Auth-Context versehen (auch wenn nur 1 User)
```

---

### 3.2 ❌ Scraping-Timeout-Risiko nicht adressiert

**Problem:**
`runtime-observability.md` erwähnt Async-Jobs, aber:
- Kein Queue-System im MVP vorgesehen
- Scraping von 50+ Career Pages kann 5–10 Minuten dauern
- HTTP-Request-Timeouts (typisch 30s) werden greifen

**Risiko:**
User erleben Timeout-Fehler bei Discovery-Flows → Produkt wirkt kaputt.

**Empfehlung:**
```
Priorität: HIGH
Aufwand: 1–2 Tage

→ BullMQ oder In-Memory-Queue mit Redis-Backup einplanen
→ Progress-Polling für Scraping-Jobs im Frontend vorsehen
→ Scraping-Batch-Größen limitieren (max 10 Companies pro Job)
```

---

### 3.3 ❌ Resume-Chunking-Strategie zu vage

**Problem:**
`data-model.md` definiert `ResumeChunk`, aber:
- Keine Chunking-Regeln (Größe, Overlap, Semantik)
- Keine Qualitätsmetrik für Chunk-Grenzen
- Embedding-Strategie ist nicht spezifiziert

**Risiko:**
Schlechtes Retrieval → schlechte Matches → Produkt verliert Glaubwürdigkeit.

**Empfehlung:**
```
Priorität: MEDIUM
Aufwand: 1 Tag

→ Chunking-Regel dokumentieren (z.B. 300–500 Tokens, 50 Token Overlap)
→ Embedding-Modell festlegen (z.B. text-embedding-3-small)
→ Retrieval-Top-K und Score-Threshold als konfigurierbare Parameter
```

---

### 3.4 ❌ MCP-Deployment-Strategie unklar

**Problem:**
`interfaces.md` definiert MCP-Tools, aber:
- Kein Hinweis, wie MCP-Clients den Server entdecken
- Keine Auth für MCP-Transport
- Keine Rate-Limiting-Strategie für MCP-Calls

**Risiko:**
MCP-Integration wird im MVP zum Sicherheitsleck oder Performance-Engpass.

**Empfehlung:**
```
Priorität: MEDIUM
Aufwand: 1 Tag

→ MCP-Server als integrierter Teil von apps/api betreiben (kein separater Prozess)
→ MCP-Tool-Calls durch Backend-Auth validieren
→ Rate-Limits pro MCP-Tool dokumentieren
```

---

### 3.5 ❌ Fehlerbehandlung bei ATS-Scraping zu optimistisch

**Problem:**
`module-design.md` listet ATS-Adapter (Greenhouse, Lever, Workday, Ashby), aber:
- Keine Fallback-Strategie bei CAPTCHA oder IP-Blocking
- Keine Retry-Logik mit Backoff
- Keine Mock-Daten für Entwicklung ohne Live-Scraping

**Risiko:**
Entwicklung stoppt, weil Scraping in der Umgebung nicht funktioniert.

**Empfehlung:**
```
Priorität: HIGH
Aufwand: 1–2 Tage

→ Mock-Adapter für alle ATS-Provider im Development-Mode
→ Retry mit Exponential Backoff (max 3 Versuche)
→ Circuit-Breaker für wiederholte Scraping-Fehler
→ Manuelle Job-Entry-Fallbacks (URL + JSON-Upload)
```

---

## 4. MVP-Optimierungen (Quick Wins)

### 4.1 ✅ Zod-Schemas als Single Source of Truth

**Aktuell:** TypeScript-Interfaces in `packages/shared` ohne Validierung.

**Optimierung:**
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

**Vorteile:**
- API-Validierung automatisch generierbar
- MCP-JSON-Schema aus Zod ableitbar
- Frontend-Formularvalidierung teilbar

**Aufwand:** 1 Tag  
**Impact:** HIGH

---

### 4.2 ✅ MCP-Server als integrierter Teil von apps/api

**Aktuell:** MCP als separate Oberfläche angedeutet.

**Optimierung:**
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
// ... weitere Tools

// In apps/api/src/index.ts
if (process.env.ENABLE_MCP) {
  attachMCPServer(server);
}
```

**Vorteile:**
- Kein separater Deploy-Prozess
- Gleiche Auth-Middleware wie HTTP
- Einfacheres Local Development

**Aufwand:** 0.5 Tage  
**Impact:** MEDIUM

---

### 4.3 ✅ Development-Seed-Skripts für Demo-Daten

**Aktuell:** Keine Seed-Strategie dokumentiert.

**Optimierung:**
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

**Vorteile:**
- Sofort testbares MVP ohne manuelle Dateneingabe
- Demo für Stakeholder ohne Live-Scraping
- Konsistente Testdaten für Entwicklung

**Aufwand:** 0.5 Tage  
**Impact:** MEDIUM

---

## 5. Fehlende MVP-Features (Gap-Analyse)

| Feature | Status in Architektur | MVP-Relevanz | Empfehlung |
|---------|----------------------|--------------|------------|
| **Auth (Supabase)** | ❌ Nicht definiert | HIGH | Sofort einplanen |
| **Resume-Upload (S3)** | ⚠️ Erwähnt, nicht spezifiziert | HIGH | Supabase Storage nutzen |
| **Job-Queue** | ⚠️ Konzept, keine Implementierung | HIGH | BullMQ oder In-Memory |
| **Health-Check Endpoint** | ❌ Fehlend | MEDIUM | `/health` für Deployment |
| **Structured Logging** | ⚠️ Erwähnt, nicht spezifiziert | MEDIUM | Pino oder Winston |
| **Error-Tracking** | ❌ Fehlend | LOW | Sentry später nachrüstbar |
| **Rate-Limiting** | ❌ Fehlend | MEDIUM | Express-Rate-Limit |

---

## 6. Konkrete Nächste Schritte

### Phase 1: Foundation (Woche 1)
```
□ Monorepo-Setup mit Turborepo
□ Supabase-Projekt anlegen (Auth + DB + Storage)
□ Zod-Schemas in packages/shared definieren
□ Basis-Ports in packages/domain definieren
```

### Phase 2: Core-Features (Woche 2–3)
```
□ Resume-Upload mit Chunking
□ Company-Discovery mit Mock-Adaptern
□ Scraping-Queue mit BullMQ
□ Match-Engine mit einfachem Scoring
```

### Phase 3: MVP-Polish (Woche 4)
```
□ Frontend-Integration (Next.js)
□ MCP-Server-Anbindung
□ Seed-Skripts für Demo-Daten
□ Deployment auf Railway oder Render
```

---

## 7. Fazit

**Gesamtbewertung:** 🟢 **Gut mit klarem Optimierungsbedarf**

Die Architektur ist **überdurchschnittlich gut durchdacht** für ein frühes Projektstadium. Die hexagonale Struktur und die klare Trennung der Verantwortlichkeiten werden sich bei der Skalierung auszahlen.

**Kritisch für MVP-Lieferfähigkeit:**
1. Auth-Strategie muss früh definiert werden (auch für Single-User)
2. Scraping-Timeouts erfordern Queue-System
3. Resume-Chunking braucht konkrete Regeln
4. ATS-Adapter benötigen Mock-Fallbacks
5. MCP-Deployment muss vereinfacht werden

**Empfohlene Priorisierung:**
1. ✅ Auth + Queue-System (Woche 1)
2. ✅ Zod-Schemas + MCP-Integration (Woche 1)
3. ✅ Mock-Adapter + Seed-Skripts (Woche 2)

**Risikolevel ohne Änderungen:** 🔴 **MVP-Lieferung gefährdet** (Scraping-Timeouts, Auth-Nachrüstung)  
**Risikolevel mit Empfehlungen:** 🟢 **MVP in 4 Wochen lieferbar**

---

## 8. Anhang: Konkrete Code-Empfehlungen

### 8.1 Queue-Setup (BullMQ)
```typescript
// apps/api/src/jobs/queue.ts
import { Queue, Worker } from 'bullmq';
import Redis from 'ioredis';

const redis = new Redis(process.env.REDIS_URL!);

export const scrapingQueue = new Queue('scraping', { connection: redis });
export const scrapingWorker = new Worker('scraping', async (job) => {
  const { companyUrl } = job.data;
  // Scraping-Logik hier
}, { connection: redis });
```

### 8.2 Auth-Port Definition
```typescript
// packages/domain/ports/auth-port.ts
export interface AuthPort {
  validateSession(token: string): Promise<Session>;
  getUserById(userId: string): Promise<User>;
  createSession(userId: string): Promise<Session>;
}
```

### 8.3 Zod zu MCP JSON-Schema
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

**Ende des Reviews**
