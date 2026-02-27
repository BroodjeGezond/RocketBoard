# RocketBoard — MVP Product Requirements Document

## Vision

RocketBoard helps lead developers onboard new engineers into unfamiliar codebases. Upload a GitHub repo zip, get an AI-generated interactive onboarding course — no more weeks of "just read the code."

---

## Constraints

- **Timeline:** 4 hours
- **Team:** 3 people
- **Goal:** Working demo, not production-ready

---

## Target Users

### 1. Hiring Manager / Lead Developer
The person responsible for onboarding. They know the codebase and want to accelerate ramp-up time for new hires.

### 2. New Employee
A developer joining the team who needs to understand an unfamiliar codebase quickly and independently.

---

## MVP Features

### Hiring Manager Flow

| Feature | Description | Priority |
|---------|-------------|----------|
| **Zip Upload** | Drag-and-drop a GitHub repo zip file | Must-have |
| **Auto Analysis** | AI extracts: tech stack, architecture, key modules, coding conventions, module dependencies | Must-have |
| **Review & Publish** | Manager reviews the generated analysis and publishes the onboarding course | Must-have |

#### What the analysis engine extracts:
1. **Tech Stack** — languages, frameworks, dependencies (from package.json, requirements.txt, etc.)
2. **Architecture Overview** — folder structure, main modules, entry points
3. **Key Patterns** — naming conventions, file organization, design patterns detected
4. **Module Map** — simplified dependency relationships between major modules/directories

### New Employee Flow

| Feature | Description | Priority |
|---------|-------------|----------|
| **Dashboard** | Overview of the repo they're onboarding into | Must-have |
| **Structured Modules** | Auto-generated learning sections (see below) | Must-have |
| **Interactive Q&A Chat** | Ask questions about the codebase with AI that has full repo context | Must-have |
| **Progress Tracking** | Simple completion checkboxes per module | Nice-to-have |

#### Generated Learning Modules:
1. **Tech Stack Overview** — what technologies are used and what role each plays
2. **Architecture Guide** — how the codebase is organized, key directories, data flow
3. **Key Modules Deep Dive** — what each major module does, its responsibilities, dependencies
4. **Coding Conventions** — patterns, naming, file structure conventions found in the repo

---

## Out of Scope (for MVP)

- Gamification / points / badges
- Week-by-week pacing / learning psychology scheduling
- Full interactive dependency graph visualization
- Multi-repo support per organization
- User authentication / accounts (single-tenant demo)
- Manager editing/customizing generated content beyond review
- Team analytics / onboarding metrics

---

## Technical Architecture

### Tech Stack (for building RocketBoard itself)

| Layer | Technology |
|-------|------------|
| **Frontend** | React + Vite |
| **Backend** | Python (FastAPI) or Node.js (Express) |
| **AI Engine** | Claude API (code analysis + Q&A) |
| **Storage** | SQLite or JSON files (MVP-level persistence) |

### High-Level Flow

```
[Manager uploads zip]
        |
        v
[Backend extracts & indexes files]
        |
        v
[AI analyzes: tech stack, architecture, patterns, dependencies]
        |
        v
[Structured course content generated & stored]
        |
        v
[Employee views dashboard + modules + asks questions via chat]
```

### AI Analysis Pipeline

1. **Extract zip** — walk the file tree
2. **Identify key files** — package.json, README, config files, entry points, main source dirs
3. **Structured prompts** — feed file contents to Claude with specific extraction prompts per category
4. **Generate modules** — Claude outputs structured markdown for each learning module
5. **Q&A context** — store repo content for retrieval when employee asks questions in chat

---

## Team Split (Suggested)

| Person | Focus Area | Key Deliverables |
|--------|-----------|-----------------|
| **P1 — Frontend** | UI/UX for both user flows | Upload page, dashboard, module viewer, chat UI |
| **P2 — Backend** | API + file processing | Zip extraction, file indexing, API endpoints, storage |
| **P3 — AI/Prompts** | Analysis engine + Q&A | Analysis prompt engineering, chat/RAG system, content generation |

---

## Milestone Plan (4 hours)

| Time | Milestone |
|------|-----------|
| **Hour 1** | Project setup, API scaffolding, basic upload UI, first analysis prompts working |
| **Hour 2** | End-to-end: upload → analysis → stored results. Frontend shows generated modules |
| **Hour 3** | Q&A chat functional. Polish module viewer. Progress tracking |
| **Hour 4** | Integration testing, bug fixes, demo prep |

---

## Success Criteria

A successful MVP demo shows:
1. Manager uploads a zip file of a real repo
2. System analyzes it and generates structured onboarding content within ~1 minute
3. New employee views a clean dashboard with learning modules
4. Employee can ask natural language questions about the codebase and get accurate answers
5. The whole flow works end-to-end without manual intervention
