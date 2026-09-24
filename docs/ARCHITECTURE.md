# System architecture

## System context

```mermaid
C4Context
  title Learning intelligence system context
  Person(student, "Student", "Completes diagnostics and reviews results")
  Person(educator, "Educator", "Builds assessments and reviews content")
  System(platform, "Learning Intelligence Platform", "Assessment, scoring, and study-planning system")
  System_Ext(model, "Model provider", "Generates candidate educational content")

  Rel(student, platform, "Takes assessments")
  Rel(educator, platform, "Authors and reviews content")
  Rel(platform, model, "Requests structured candidates and verification")
```

## Container view

```mermaid
flowchart TB
  subgraph Clients
    ADMIN[Educator React app]
    LEARNER[Student app]
  end

  subgraph Platform
    AUTH[Authentication]
    REST[Row-level data API]
    FN[Trusted edge functions]
    DB[(PostgreSQL)]
    OBJECTS[(Media storage)]
  end

  subgraph Domain
    BANK[Question bank]
    BLUEPRINT[Assessment blueprints]
    SESSION[Attempt snapshots]
    SCORE[Pure scoring engine]
    RESULTS[Result projections]
    AIGEN[AI content pipeline]
  end

  ADMIN --> AUTH
  LEARNER --> AUTH
  AUTH --> REST
  ADMIN --> REST
  ADMIN --> FN
  LEARNER --> FN
  REST --> DB
  FN --> DB
  FN --> OBJECTS
  DB --> BANK
  DB --> BLUEPRINT
  DB --> SESSION
  FN --> SCORE --> RESULTS
  FN --> AIGEN
```

## Assessment lifecycle

```mermaid
sequenceDiagram
  participant E as Educator
  participant B as Blueprint builder
  participant F as Trusted functions
  participant D as Database
  participant S as Student
  participant G as Scoring engine

  E->>B: Define coverage and difficulty mix
  B->>F: Request an assembled assessment
  F->>D: Save reviewed template
  S->>F: Start an attempt
  F->>D: Create an immutable question snapshot
  loop Timed sections
    S->>F: Submit answer / heartbeat / section
    F->>D: Grade and persist trusted state
  end
  F->>G: Finalize graded responses
  G-->>F: Mastery, readiness, fit, recommendations
  F->>D: Persist stable result projections
  S->>F: Read results
  F-->>S: Precomputed result view
```

## Trust boundaries

### Browser-safe responsibilities

- Render allowed questions, passages, mathematics, diagrams, and progress.
- Manage educator drafting state and client-side interaction.
- Send signed requests and display server-authorized results.

### Trusted-function responsibilities

- Authenticate the caller and enforce ownership.
- Read answer keys, grade responses, and finalize attempts.
- Call external models using server-held credentials.
- Validate structured AI output and enforce publication state.

### Database responsibilities

- Row-level access control.
- Referential integrity and attempt ownership.
- Durable snapshots and result projections.
- Audit-friendly content and status transitions.

## Why serverless fit this product

The platform needed trusted operations but not a permanently running custom server. Edge functions provided small, independently deployable security boundaries around grading, finalization, and AI calls while the frontend used authenticated row-level data access for ordinary educator workflows.

The tradeoff is deployment coordination: shared function code must be redeployed anywhere it is bundled, and cross-record operations need deliberate transactional design.
