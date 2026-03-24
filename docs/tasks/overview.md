# Delivery Overview

See also: [index.md](./index.md)

## Architecture Prerequisites

The approved architecture in `docs/architecture/` is treated as complete enough for delivery planning.
No architecture-definition work is represented as a junior implementation task in this delivery package.

## Feature Sequence

### [F1 Foundation Skeleton](./task-briefs/f1-f4.md)

Goal:
Create the monorepo execution skeleton for web, API, shared contracts, and domain placement so later feature teams can work without inventing repository structure.

Tasks:

- `F1-Backend-1`
  Establish `apps/api` skeleton, transport entry shape, and package wiring.

- `F1-Frontend-1`
  Establish `apps/web` shell, route skeleton, and initial app layout scaffold.

- `F1-Cloud-1`
  Establish local workspace scripts, environment shape, and baseline developer startup flow.

- `F1-Test-1`
  Establish baseline test runner structure and repository-wide test folder conventions.

Dependencies:
- none

Can start in parallel:
- all F1 tasks

Detailed brief:
- [task-briefs/f1-f4.md](./task-briefs/f1-f4.md)

User test:
- [user-tests.md](./user-tests.md)

### [F2 Shared Contracts And User Context](./task-briefs/f1-f4.md)

Goal:
Create the shared contract layer and explicit user-context foundation so all later work can rely on stable request, response, and identity semantics.

Tasks:

- `F2-Backend-1`
  Implement shared transport contract scaffolding and backend contract consumption.

- `F2-Backend-2`
  Implement user-context resolution path for single-user MVP semantics.

- `F2-Frontend-1`
  Integrate shared contract usage into frontend request and form boundaries.

- `F2-Test-1`
  Validate contract reuse and user-context behavior.

- `F2-Safety-1`
  Review identity-boundary and request-scoping assumptions.

Dependencies:
- F1

Can start in parallel:
- `F2-Backend-1`, `F2-Backend-2`, `F2-Frontend-1`, `F2-Safety-1`

Must wait for:
- `F2-Test-1` waits for the corresponding implementation tasks

Detailed brief:
- [task-briefs/f1-f4.md](./task-briefs/f1-f4.md)

User test:
- [user-tests.md](./user-tests.md)

### [F3 Resume Management](./task-briefs/f1-f4.md)

Goal:
Support resume upload, storage, version management, and retrieval-ready resume material.

Tasks:

- `F3-Backend-1`
  Implement resume metadata and version persistence.

- `F3-Backend-2`
  Implement file-storage handling and retrieval-ready resume chunk persistence boundary.

- `F3-Frontend-1`
  Implement resume upload and version list UI.

- `F3-Test-1`
  Validate resume upload, version retrieval, and visible state handling.

- `F3-Safety-1`
  Review sensitive-file handling and document-scope assumptions.

Dependencies:
- F2

Can start in parallel:
- `F3-Backend-1`, `F3-Backend-2`, `F3-Frontend-1`, `F3-Safety-1`

Must wait for:
- `F3-Frontend-1` depends on stable contract outputs from `F3-Backend-1`
- `F3-Test-1` waits for implementation tasks

Detailed brief:
- [task-briefs/f1-f4.md](./task-briefs/f1-f4.md)

User test:
- [user-tests.md](./user-tests.md)

### [F4 Company Discovery](./task-briefs/f1-f4.md)

Goal:
Turn user intent into candidate companies that can later feed scraping and opportunity generation.

Tasks:

- `F4-AI-1`
  Implement LLM-backed discovery strategy under the approved architecture constraints.

- `F4-Backend-1`
  Implement company discovery orchestration and persistence boundary.

- `F4-Frontend-1`
  Implement search input and discovery-result presentation.

- `F4-Test-1`
  Validate discovery flow, empty results, and user-visible output shape.

Dependencies:
- F2

Can start in parallel:
- `F4-AI-1`, `F4-Backend-1`, `F4-Frontend-1`

Must wait for:
- `F4-Test-1` waits for implementation tasks
- `F4-Frontend-1` depends on stable discovery contract from `F4-Backend-1`

Detailed brief:
- [task-briefs/f1-f4.md](./task-briefs/f1-f4.md)

User test:
- [user-tests.md](./user-tests.md)

### [F5 ATS Scraping Pipeline](./task-briefs/f5-f8.md)

Goal:
Detect supported ATS families, scrape career pages, and produce normalized raw extraction results suitable for opportunity creation.

Tasks:

- `F5-Backend-1`
  Implement ATS detection and scraper orchestration boundary.

- `F5-Backend-2`
  Implement provider-specific adapter set for Greenhouse and Lever.

- `F5-Backend-3`
  Implement provider-specific adapter set for Workday and Ashby.

- `F5-Test-1`
  Validate detection, extraction behavior, and failure classes.

- `F5-Safety-1`
  Review outbound scraping constraints and failure handling assumptions.

Dependencies:
- F4

Can start in parallel:
- `F5-Backend-2` and `F5-Backend-3` after `F5-Backend-1` defines the stable adapter boundary
- `F5-Safety-1` can start with `F5-Backend-1`

Must wait for:
- `F5-Test-1` waits for scraper implementations

Detailed brief:
- [task-briefs/f5-f8.md](./task-briefs/f5-f8.md)

User test:
- [user-tests.md](./user-tests.md)

### [F6 Opportunity Feed](./task-briefs/f5-f8.md)

Goal:
Normalize scraped listings into internal opportunities and show a visible ranked opportunity feed.

Tasks:

- `F6-Backend-1`
  Implement normalization and opportunity persistence.

- `F6-Backend-2`
  Implement open-opportunity retrieval and freshness-aware query behavior.

- `F6-Frontend-1`
  Implement opportunity feed rendering and visible opportunity cards.

- `F6-Designer-1`
  Define visible card hierarchy and feed presentation details consistent with current product direction.

- `F6-Test-1`
  Validate feed behavior, empty states, and normalization visibility.

Dependencies:
- F5

Can start in parallel:
- `F6-Backend-1` and `F6-Designer-1`
- `F6-Frontend-1` after minimal feed contract from `F6-Backend-2`

Must wait for:
- `F6-Backend-2` depends on `F6-Backend-1`
- `F6-Test-1` waits for implementation tasks

Detailed brief:
- [task-briefs/f5-f8.md](./task-briefs/f5-f8.md)

User test:
- [user-tests.md](./user-tests.md)

### [F7 Matching And Recommendation](./task-briefs/f5-f8.md)

Goal:
Generate match score, explanation, and resume recommendation for opportunity-resume pairs.

Tasks:

- `F7-AI-1`
  Implement matching strategy with retrieval-aware reasoning under approved AI boundaries.

- `F7-Backend-1`
  Implement match orchestration and `MatchResult` persistence or retrieval path.

- `F7-Frontend-1`
  Implement match explanation, score, and resume recommendation display.

- `F7-Test-1`
  Validate score visibility, explanation output, and failure handling.

Dependencies:
- F3
- F6

Can start in parallel:
- `F7-AI-1` and `F7-Backend-1` after stable matching contract agreement from existing architecture

Must wait for:
- `F7-Frontend-1` depends on stable match result contract from `F7-Backend-1`
- `F7-Test-1` waits for implementation tasks

Detailed brief:
- [task-briefs/f5-f8.md](./task-briefs/f5-f8.md)

User test:
- [user-tests.md](./user-tests.md)

### [F8 Application Tracking](./task-briefs/f5-f8.md)

Goal:
Allow the user to record application actions and visible outcomes tied to concrete resume versions and opportunities.

Tasks:

- `F8-Backend-1`
  Implement application persistence and lifecycle transitions.

- `F8-Frontend-1`
  Implement apply-state action and outcome update UI.

- `F8-Test-1`
  Validate application state changes and visible tracking behavior.

- `F8-Safety-1`
  Review application-history integrity and scoped user-context behavior.

Dependencies:
- F3
- F6

Can start in parallel:
- `F8-Backend-1`, `F8-Safety-1`

Must wait for:
- `F8-Frontend-1` depends on stable application contract
- `F8-Test-1` waits for implementation tasks

Detailed brief:
- [task-briefs/f5-f8.md](./task-briefs/f5-f8.md)

User test:
- [user-tests.md](./user-tests.md)

### [F9 Historical Retrieval And Insights](./task-briefs/f9-f12.md)

Goal:
Generate and use history-based learning signals from application outcomes.

Tasks:

- `F9-AI-1`
  Implement insight extraction logic and learning-signal rules.

- `F9-Backend-1`
  Implement historical retrieval and insight persistence path.

- `F9-Backend-2`
  Implement retrospective insight refresh execution path.

- `F9-Test-1`
  Validate insight trigger behavior and historical retrieval usage.

Dependencies:
- F7
- F8

Can start in parallel:
- `F9-AI-1`, `F9-Backend-1`, `F9-Backend-2`

Must wait for:
- `F9-Test-1` waits for implementation tasks

Detailed brief:
- [task-briefs/f9-f12.md](./task-briefs/f9-f12.md)

User test:
- [user-tests.md](./user-tests.md)

### [F10 Skill-Gap And Resume Guidance](./task-briefs/f9-f12.md)

Goal:
Show skill-gap backlog and resume-guidance outputs grounded in history and matching context.

Tasks:

- `F10-AI-1`
  Implement skill-gap generation rules and guidance semantics.

- `F10-Backend-1`
  Implement guidance retrieval and delivery path.

- `F10-Frontend-1`
  Implement visible skill-gap and resume-guidance presentation.

- `F10-Test-1`
  Validate guidance visibility and recommendation behavior.

Dependencies:
- F9

Can start in parallel:
- `F10-AI-1`, `F10-Backend-1`

Must wait for:
- `F10-Frontend-1` depends on stable guidance contract
- `F10-Test-1` waits for implementation tasks

Detailed brief:
- [task-briefs/f9-f12.md](./task-briefs/f9-f12.md)

User test:
- [user-tests.md](./user-tests.md)

### [F11 Cover-Letter Scaffolding](./task-briefs/f9-f12.md)

Goal:
Generate grounded cover-letter scaffolding using company, opportunity, and resume context.

Tasks:

- `F11-AI-1`
  Implement scaffolding-generation behavior under current product scope.

- `F11-Backend-1`
  Implement cover-letter request orchestration and output delivery.

- `F11-Frontend-1`
  Implement visible scaffolding display and user flow entry.

- `F11-Test-1`
  Validate grounded output presence and visible user flow.

Dependencies:
- F3
- F6
- F10

Can start in parallel:
- `F11-AI-1`, `F11-Backend-1`

Must wait for:
- `F11-Frontend-1` depends on stable cover-letter contract
- `F11-Test-1` waits for implementation tasks

Detailed brief:
- [task-briefs/f9-f12.md](./task-briefs/f9-f12.md)

User test:
- [user-tests.md](./user-tests.md)

### [F12 Runtime Hardening And Release Readiness](./task-briefs/f9-f12.md)

Goal:
Make the MVP operationally robust enough for repeatable development and release confidence.

Tasks:

- `F12-Cloud-1`
  Implement environment handling, local runtime wiring, and baseline deployment/readiness support.

- `F12-Backend-1`
  Implement job progress visibility, health checks, and operational support endpoints where needed.

- `F12-Test-1`
  Strengthen automated verification for critical flows.

- `F12-Browser-1`
  Validate visible critical user flows in a real browser.

- `F12-Safety-1`
  Review sensitive runtime paths, logs, and request boundaries before release.

Dependencies:
- F8
- F9
- F10
- F11

Can start in parallel:
- all F12 tasks after the dependent features are stable

Detailed brief:
- [task-briefs/f9-f12.md](./task-briefs/f9-f12.md)

User test:
- [user-tests.md](./user-tests.md)
