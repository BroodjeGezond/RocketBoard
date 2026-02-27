MASTER PROMPT — RocketBoard (Demo-Ready, 3-Mode, Quizzes + Notes)
SECTION 1: PROJECT FOUNDATIONS
PROJECT: RocketBoard
TAGLINE: “Stop reading code. Start understanding it.”
WHAT IT DOES: Upload a repo zip → AI analyzes codebase → generates 6 onboarding modules → manager reviews & publishes → new hire follows modules, takes quizzes, writes personal notes, and asks questions via AI chat with file-grounded references.

Tech Stack
Frontend: React 18 + Vite + TypeScript + Tailwind CSS + shadcn/ui
Backend: Python 3.11+ + FastAPI
AI: Anthropic Claude API
Model read from env var CLAUDE_MODEL (default: claude-sonnet-4-20250514)
DB: SQLite via SQLAlchemy (WAL mode enabled)
File Storage: Local filesystem under DATA_DIR (default ./data)
extracted repos stored in ./data/repos/{repo_id}/
Non‑Negotiable Demo Reliability Rules
Zip security hardened: prevent Zip Slip, skip symlinks, cap file count/bytes, skip suspicious compression ratios.
Upload streaming: do not await file.read() into RAM for large zips; stream to disk in chunks.
Analysis must not block FastAPI: run analysis pipeline in a separate thread (executor).
SQLite WAL mode + short transactions + separate session per thread/request.
No hallucinated file paths: AI may only reference file paths from the indexed file tree / retrieved file set. Backend must validate and discard invalid references.
Snippets are server-generated: AI returns which files matter; backend attaches real snippets from DB (never model-invented snippets).
DEMO_MODE safety net: if Claude fails, load pre-seeded repo analysis and modules instantly.
Environment Variables (.env)
CLAUDE_API_KEY=...
CLAUDE_MODEL=claude-sonnet-4-20250514
DEMO_MODE=false
DATA_DIR=./data
FRONTEND_URL=http://localhost:5173
MAX_ZIP_SIZE_MB=100
MAX_EXTRACTED_FILES=10000
MAX_EXTRACTED_BYTES=524288000 (500MB)
MAX_TOTAL_LINES=50000
MAX_SINGLE_FILE_BYTES=5242880 (5MB)
REPO_TTL_HOURS=24
Design System (Dark-mode dev tool)
Background: slate-950, surfaces: slate-900, borders: slate-700
Primary: indigo-500, success: emerald-500, warning: amber-500, error: red-500
Fonts: Inter + JetBrains Mono
Minimal animations only (fast fades, subtle pulses)
Visible focus rings (ring-2 ring-indigo-500) on all interactive elements
Mobile touch targets minimum 44×44
SECTION 2: THE 3 MODES (NO AUTH) — LINKS/TOKENS DRIVE UI & PERMISSIONS
Because there is no authentication, we support three experiences using different routes/tokens:

Mode A — Manager (Upload + Progress + Review + Publish)
Manager uploads zip, watches progress, reviews modules, publishes links.
Mode B — Author / Senior Dev (Co-authoring mode)
Author can edit module content (manual markdown edits),
can refine a module with AI via instruction (“make more beginner-friendly”, “add a diagram”, etc.),
can regenerate quiz if needed.
Mode C — Learner / New Hire (Read-only + Notes + Quizzes + Progress + Chat)
Learner cannot edit course content.
Learner can:
mark modules complete (per learner)
take quizzes (per module)
add notes per module and return later
resume where left off
Tokens / Links
share_code → learner link (read-only): /onboard/ROCKET-XXXXXX
manage_code → author link (editable): /manage/MANAGE-XXXXXX
Manager can also use direct /repo/:repoId/review during build/demo.
SECTION 3: ROUTES (FRONTEND)
/ → Manager Upload Page
/repo/:repoId/review → Manager Progress + Review (analyzing/completed)
/manage/:manageCode → Author Dashboard (edit/refine/regenerate quiz)
/onboard/:shareCode → Learner Dashboard (read-only modules + notes + quizzes + chat)
/repo/:repoId → Learner Dashboard (direct access for demo convenience)
No login. Single-tenant demo.

SECTION 4: API CONTRACT (BACKEND)
BASE URL: http://localhost:8000/api
Response envelope for JSON endpoints:

JSON

{ "success": true, "data": {...}, "error": null }
1) POST /api/repos/upload (Manager)
multipart/form-data:
file (.zip)
repo_name optional
returns 202 Accepted:
JSON

{ "success": true, "data": { "repo_id": "uuid", "status": "processing" }, "error": null }
2) GET /api/repos/{repo_id}/status
returns progress object with current_step + message + counts.
3) GET /api/repos/{repo_id}
Manager/author fetch of repo + modules (no learner progress/notes).
4) POST /api/repos/{repo_id}/publish
Creates or returns existing share codes:
JSON

{
  "success": true,
  "data": {
    "share_code": "ROCKET-a1b2c3",
    "share_url": "/onboard/ROCKET-a1b2c3",
    "manage_code": "MANAGE-x9y8z7",
    "manage_url": "/manage/MANAGE-x9y8z7"
  },
  "error": null
}
5) GET /api/onboard/{share_code}
Learner access to published repo analysis + modules plus learner-specific state.
Includes learner object + progress + notes summary.
6) POST /api/onboard/{share_code}/session
Purpose: create/restore a learner session (no auth).

Request:
JSON

{ "learner_id": "optional-uuid-from-localStorage", "display_name": "optional" }
Response:
JSON

{ "success": true, "data": { "learner_id": "uuid" }, "error": null }
Frontend stores learner_id in localStorage keyed by share_code.

7) PATCH /api/onboard/{share_code}/progress/{module_key}
Request:
JSON

{ "learner_id": "uuid", "is_completed": true, "last_opened": true }
Backend updates per-learner progress (and last opened module pointer).
8) PUT /api/onboard/{share_code}/notes/{module_key}
Request:
JSON

{ "learner_id": "uuid", "notes_markdown": "..." }
Saves notes (learner-only, does not change module content).
9) GET /api/onboard/{share_code}/quiz/{module_key}
Returns quiz JSON; if not generated yet, backend lazy-generates via Claude and caches.
Response:
JSON

{
  "success": true,
  "data": {
    "module_key": "architecture",
    "quiz": {
      "questions": [
        {
          "id": "q1",
          "question": "Where is routing defined in this project?",
          "options": ["...","...","...","..."],
          "correct_index": 1,
          "explanation": "Based on ...",
          "evidence_files": ["src/routes/index.ts"]
        }
      ]
    }
  },
  "error": null
}
10) POST /api/onboard/{share_code}/quiz/{module_key}/attempt
Request:
JSON

{ "learner_id": "uuid", "answers": { "q1": 1, "q2": 3 }, "score": 4, "total": 5 }
Stores attempt & updates progress with last quiz score.
11) POST /api/onboard/{share_code}/chat
Request:
JSON

{
  "learner_id": "uuid",
  "message": "Where is authentication handled?",
  "current_module_key": "architecture",
  "conversation_id": "optional-uuid"
}
Response includes response text + referenced file paths + backend-generated snippets:
JSON

{
  "success": true,
  "data": {
    "response": "Authentication appears to be handled in ...",
    "referenced_files": [
      { "path": "src/auth/middleware.ts", "snippet": "...\n...\n...", "relevance": "JWT middleware" }
    ],
    "evidence_files": ["src/auth/middleware.ts"],
    "conversation_id": "uuid"
  },
  "error": null
}
12) GET /api/manage/{manage_code}
Returns repo + modules for author UI.
13) PATCH /api/repos/{repo_id}/modules/{module_key} (Author manual edit)
Request:
JSON

{ "manage_code": "MANAGE-xxx", "content_markdown": "..." }
14) POST /api/repos/{repo_id}/modules/{module_key}/refine (Author AI refine)
Request:
JSON

{ "manage_code": "MANAGE-xxx", "instruction": "Make this more beginner-friendly and add a diagram." }
Returns updated module markdown and updated evidence list.
15) POST /api/repos/{repo_id}/modules/{module_key}/quiz/regenerate (Author optional)
Request:
JSON

{ "manage_code": "MANAGE-xxx" }
16) GET /api/repos/{repo_id}/export
Downloads all modules as combined markdown.
17) Admin/demo endpoints (only if DEMO_MODE=true)
POST /api/admin/reset-demo { repo_id }
POST /api/admin/cleanup → delete repos older than TTL
DELETE /api/repos/{repo_id} → delete repo + disk
SECTION 5: DATABASE SCHEMA (SQLite + SQLAlchemy)
repos
id (uuid pk)
repo_name
status: processing|completed|failed
current_step, steps_completed, total_steps, progress_message
file_stats_json (text)
summary_json (text)
share_code (unique, nullable)
manage_code (unique, nullable)
error_message
created_at, completed_at, published_at
modules
id (uuid pk)
repo_id (fk)
module_key (unique per repo): tech-stack|architecture|getting-started|key-modules|coding-conventions|quick-reference
title, description, icon, estimated_minutes, order_index
content_markdown (text)
evidence_json (text JSON array of {path, note} OR list of paths)
quiz_json (nullable text) ← cached quiz for module
updated_at
UNIQUE(repo_id, module_key)
repo_files
id (uuid pk)
repo_id (fk)
file_path (relative)
language, file_type, is_key_file
content (truncated)
lines_json (JSON array of lines)
line_count
learner_sessions (per share_code)
learner_id (uuid pk)
repo_id (fk)
share_code (indexed)
display_name (nullable)
last_opened_module_key (nullable)
created_at, last_seen_at
UNIQUE(repo_id, learner_id)
learner_progress
id (uuid pk)
learner_id (fk)
repo_id (fk)
module_key
is_completed (bool)
completed_at (nullable)
last_quiz_score (nullable int)
last_quiz_total (nullable int)
last_quiz_at (nullable)
updated_at
UNIQUE(learner_id, repo_id, module_key)
learner_notes
id (uuid pk)
learner_id (fk)
repo_id (fk)
module_key
notes_markdown (text)
updated_at
UNIQUE(learner_id, repo_id, module_key)
chat_messages
id (uuid pk)
repo_id (fk)
learner_id (nullable fk)
conversation_id
role user|assistant
content
referenced_files_json (nullable)
current_module_key (nullable)
created_at
INDEX(repo_id, conversation_id)
ai_generations (observability)
id (uuid pk)
repo_id (fk nullable)
kind (e.g., module_generation, quiz_generation, chat)
key (module_key, etc.)
model
duration_ms
input_tokens (nullable)
output_tokens (nullable)
success
error_message (nullable)
created_at
SECTION 6: BACKEND SERVICES (CRITICAL IMPLEMENTATION)
Upload handling (streaming)
Stream the UploadFile to disk in chunks
Enforce MAX_ZIP_SIZE_MB during streaming
Do not hold entire zip in memory
Zip processor (security hardened)
Prevent Zip Slip: resolved path must stay inside repo_root
Skip symlinks
Limit extracted file count and extracted bytes
Skip suspicious compression ratios
Flatten single top-level folder carefully (avoid collisions)
File indexer
Deny list directories (node_modules, .git, dist, build, etc.)
Deny list filenames (lockfiles, etc.)
Deny list extensions (binary/media/archives)
IMPORTANT: handle minified patterns by filename:
if filename endswith .min.js or .min.css → skip
Skip files > MAX_SINGLE_FILE_BYTES before reading
Read UTF-8 strict; skip unreadable/binary
Truncate to 500 lines
Enforce total line budget (50k) with priority ordering
Analyzer orchestration
Run in separate thread via executor
Update repo status per step
Sequential Claude calls
If a module fails generation, store fallback content; pipeline continues
Retrieval for chat
Simple keyword/path scoring (MVP)
Select top N relevant files under a character budget
Provide those files to Claude as context
Snippet attachment (IMPORTANT)
Claude returns referenced file paths + relevance.
Backend validates paths exist in retrieved set or in repo_files table.
Backend generates snippet windows from DB lines and returns them.
SECTION 7: FRONTEND SPEC (PAGES + UI BEHAVIOR)
Manager Upload (/)
UploadZone drag/drop .zip
Button “Analyze Repository”
On success redirect to /repo/:repoId/review
Manager Review (/repo/:repoId/review)
Two states:

Analyzing: progress checklist + message; polls status every 2s
Completed: Repo summary + module card previews + Publish & Export
Author Mode (/manage/:manageCode)
Same module list + preview
For each module:
“Edit” (markdown textarea) → Save
“Improve with AI” (instruction input) → refine endpoint
“Regenerate quiz” button (optional)
Evidence chips visible and expanded by default
Learner Dashboard (/onboard/:shareCode)
On first load:

call /session to get/store learner_id (localStorage)
Layout:

Sidebar module nav (desktop), dropdown on mobile
Main module content read-only markdown
Notes panel (learner-only):
Right side tab or collapsible panel: “My Notes”
Autosave via PUT notes endpoint (debounced 500ms)
Notes persist per module; reload when revisiting module
Quiz at end of module:
Button “Take Quiz” shown at bottom of module
Loads quiz via GET quiz endpoint (lazy generated if missing)
Renders 5 MCQ; submit shows score + explanations
POST attempt to store score
Completion:
Button “Mark Module Complete”
PATCH progress endpoint
Resume:
Continue button opens last_opened_module_key or next incomplete
Chat (learner)
Slide-out panel (desktop) / fullscreen modal (mobile)
Suggested question chips
Sends /onboard/{share_code}/chat with learner_id + current_module_key
Shows referenced file chips; chips expand to show snippet (already returned)
SECTION 8: AI PROMPTS (P3)
Global grounding rules (apply to all prompts)
Base analysis only on provided context; do not guess
Only mention file paths from provided file tree / provided file list
Code blocks only if exact code is present in context; otherwise omit code blocks
Evidence must be a subset of provided file paths
Repo Summary prompt
Output JSON {what, tech, entry_point, first_read} (no markdown)
Module generation prompts
Output Markdown content plus ## Evidence section with bullet file paths and notes.
Backend parses evidence into evidence_json and may optionally strip the Evidence section from displayed markdown (or keep it and also show chips—either is fine).
Quiz generation prompt (lazy)
Input context:

module content_markdown
file tree (short)
a few evidence file contents (if available) Output JSON quiz with:
5 MCQ
correct_index + explanation
evidence_files list (only if grounded)
Quiz rule: If evidence is insufficient, quiz questions should test understanding of the onboarding module content itself and avoid claiming precise code facts.

Chat prompt
Input context:

user question
current_module_key
retrieved files (paths + contents) Output JSON:
response (markdown)
referenced_paths (subset of retrieved files, with relevance)
Backend attaches real snippets.

SECTION 9: ACCEPTANCE CRITERIA (DEMO-READY)
End-to-end
 Upload zip → progress steps update → completed modules show
 Publish returns both learner link and author link
 Learner link loads, creates learner session, resumes progress
 Learner can mark modules complete without editing content
 Learner notes save + reload per module
 Quiz loads (lazy), submits, shows score + persists attempt
 Chat answers grounded in retrieved files; referenced files are real; snippets display
 Author mode can edit markdown + save; refine via AI instruction works
 Demo mode loads pre-seeded analysis instantly if Claude fails
 No “database locked” under polling + background analysis
SECTION 10: DEMO SAFETY NETS
Pre-seeded demo repo in ./data/demo/ + DEMO_MODE=true toggle
If upload is slow: start demo from pre-existing /repo/:repoId/review completed state
Pre-plan 3 chat questions that reliably produce good answers:
“Where is authentication handled?”
“How do I run this locally?”
“What’s the request flow for API endpoints?”
If quiz generation fails: show “Quiz unavailable right now—continue to next module” (don’t block demo)
