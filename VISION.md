# CeeVee — Vision Document

## What is CeeVee?

CeeVee is a smart job opportunity agent. You describe what you're looking for in plain language — the kind of company, the location, the industry focus, your priorities. It finds those companies, checks their career pages, and surfaces the most relevant open roles matched against your resume. Over time, it learns from your application history.

---

## The Happy Path

1. Upload your resume (PDF) — you can store multiple versions
2. Describe what you're looking for in plain language:
   > "Software startups in Berlin, ideally in healthcare or fintech. I prefer early-stage companies with small engineering teams."
3. CeeVee discovers relevant companies and scrapes their career pages
4. You see a ranked list of opportunities, each showing:
   - Company, role, location, link
   - % match to your resume + reasoning
   - Which resume version to send and what to change
5. Mark an opportunity as "Applied" — it enters your application tracker
6. Record outcomes (interview / rejection / no response)
7. CeeVee surfaces patterns from your history: what's working, what's not

---

## Core Features (MVP Scope)

### Must Have
- Resume upload and storage (one user, multiple versions supported)
- Company discovery: free-form natural language prompt → LLM-generated list of companies
- Career page scraping: detect ATS provider (Greenhouse, Lever, Workday, Ashby) and extract job listings in a normalized format
- Match scoring: % match per job per resume, with reasoning
- Opportunity feed: ranked list of matched jobs
- Application tracker: mark jobs as applied, record outcome
- A Skill Section where a user can mention all of their skills and the Agent can update the resume with what is relevant for a specific application
- Cover letter builder (scrape what the company does, what the core values are, and what can actually be of interest for the applicants) - v0 does not have to be the full cover letter, but bullet points of the important parts (scaffolding). 

### Should Have
- Resume recommendation: which version to send + what to tweak for a specific role
- Learning system: analyze past applications using RAG, surface patterns

---

## Insights

- By comparing the CV to relevant applications, the Agent creates a backlog of relevant skills to learn next (i.g., if the agent sees the Docker is on 90% of application but not on the CV, it will prioritize learning it).

---

### Out of Scope (for now)
- Auto-apply
- Multi-user / sharing
- LinkedIn / Xing scraping

---

## Tech Stack

| Layer | Choice |
|---|---|
| Frontend | Next.js (App Router) |
| Backend | TypeScript — Next.js API routes or separate Node service |
| Database | Supabase (Postgres + pgvector) |
| Auth | Supabase Auth |
| Language | TypeScript (primary) |
| Monorepo | Yes — one repo, shared types |

---

## Architecture Principles

### Port-Adapter (Hexagonal Architecture)

The core business logic must not know about HTTP, databases, or LLM providers. Every external dependency is hidden behind a port (interface) and implemented by an adapter.

**Example ports:**
- `CompanyDiscoveryPort` — given a natural language prompt, returns a list of companies
- `CareerPageScraperPort` — given a URL, returns structured job listings
- `MatchEnginePort` — given a job + resume, returns score + recommendations
- `ApplicationRepositoryPort` — CRUD for application history

Each port has at least one adapter. The LLM adapter for `CompanyDiscoveryPort` can be swapped for a Crunchbase adapter later without touching core logic. That is the point.

### Agent Architecture via MCP

The backend exposes its capabilities as an MCP server. Core tools:

| Tool | Description |
|---|---|
| `discover_companies(prompt)` | Returns a list of companies from a natural language prompt |
| `scrape_career_page(url)` | Returns normalized job listings |
| `match_resume(job_id, resume_id)` | Returns score + recommendation |
| `log_application(job_id, resume_id)` | Saves to application history |
| `get_application_insights()` | RAG-powered pattern analysis |

### RAG — Two Use Cases

Embeddings live in Supabase pgvector.

1. **Application history** — embed each application (role, company, job description, outcome). When evaluating a new opportunity, retrieve semantically similar past applications as context for match scoring and insights.
2. **Resume chunks** — embed resume sections. Retrieve relevant chunks when generating a tailored cover letter or explaining a match score.

---

## What "Done" Looks Like

Not a fully polished product. "Done" means every contributor can point to real technical decisions they made and explain the tradeoffs.

Concretely:
1. An architecture decision record (ADR) or design doc was written before major implementation began
2. Port-Adapter pattern is visible and enforced — domain logic does not import HTTP clients or DB drivers directly
3. At least one MCP server is built and running
4. RAG is implemented for at least one feature
5. At least one genuinely hard problem was encountered, debugged, and solved

---

## The Challenges Worth Talking About in Interviews

These are the stories you want to be able to tell:

> "We needed to scrape career pages from dozens of different companies. Each was structured differently. Here's how we solved that..."

> "We used RAG to build a learning system. Here's what we embedded, why, and what we retrieved at query time..."

> "We designed a hexagonal architecture. Here's how it let us swap the LLM adapter without touching the core..."

> "We built an MCP server from scratch. Here's what it exposed and how the agent called it..."
