SECTION 1: SHARED FOUNDATIONS

PROJECT: RocketBoard
TAGLINE: "Stop reading code. Start understanding it."

TECH STACK:
- Frontend: React 18 + Vite + TypeScript + Tailwind CSS + shadcn/ui
- Backend: Python (FastAPI) — chosen for superior zip/file handling 
  and Python ecosystem for text processing
- AI: Anthropic Claude 3.5 Sonnet API (code analysis + Q&A)
- Storage: SQLite via SQLAlchemy (simple, file-based, zero setup)
- File Storage: Local filesystem (extracted repos stored in 
  /tmp or /data directory)

DESIGN SYSTEM:
- Style: Developer-focused. Clean, dark-mode-first, monospace 
  accents. Think "GitHub meets Linear."
- Primary: Indigo (#6366F1)
- Secondary: Emerald (#10B981) for success/complete states
- Background: Slate-950 (#020617) — dark mode default
- Surface: Slate-900 (#0F172A) for cards
- Border: Slate-700 (#334155)
- Text: Slate-50 (#F8FAFC) primary, Slate-400 (#94A3B8) muted
- Accent: Amber (#F59E0B) for highlights/warnings
- Code font: JetBrains Mono or Fira Code
- Body font: Inter
- Border radius: rounded-lg throughout
- Animations: minimal — fast fade-ins, no playfulness. 
  Professional dev tool feel.

PROJECT STRUCTURE:
rocketboard/
├── frontend/           (P1 owns)
│   ├── src/
│   │   ├── components/
│   │   ├── pages/
│   │   ├── hooks/
│   │   ├── types/
│   │   └── api/        (API client functions)
│   ├── package.json
│   └── vite.config.ts
├── backend/            (P2 owns)
│   ├── app/
│   │   ├── main.py     (FastAPI app)
│   │   ├── routers/
│   │   ├── services/
│   │   │   ├── zip_processor.py
│   │   │   ├── file_indexer.py
│   │   │   └── ai_analyzer.py  (P3 co-owns)
│   │   ├── models/
│   │   ├── database.py
│   │   └── prompts/    (P3 owns)
│   │       ├── tech_stack.py
│   │       ├── architecture.py
│   │       ├── patterns.py
│   │       ├── modules.py
│   │       ├── getting_started.py
│   │       └── qa.py
│   ├── data/           (SQLite DB + extracted repos)
│   └── requirements.txt
└── README.md

SECTION 2: API CONTRACT (READ FIRST — ALL 3 PEOPLE)

BASE URL: http://localhost:8000/api

All responses follow: { success: bool, data?: any, error?: string }

===================================================
ENDPOINT 1: POST /api/repos/upload
===================================================
Purpose: Manager uploads a zip file
Request: multipart/form-data
  - file: .zip file (max 100MB)
  - repo_name: string (optional — extracted from zip if not provided)
Response (202 Accepted):
  {
    success: true,
    data: {
      repo_id: "uuid-string",
      status: "processing",
      message: "Upload received. Analysis starting..."
    }
  }
Errors:
  - 400: "No file uploaded" / "File must be a .zip" / "File too large (max 100MB)"
  - 500: "Failed to process upload"

Notes: This kicks off async analysis. Frontend polls status.

===================================================
ENDPOINT 2: GET /api/repos/{repo_id}/status
===================================================
Purpose: Poll analysis progress
Response (200):
  {
    success: true,
    data: {
      repo_id: "uuid",
      status: "processing" | "completed" | "failed",
      progress: {
        current_step: "extracting" | "indexing" | "analyzing_tech_stack" 
          | "analyzing_architecture" | "analyzing_patterns" 
          | "analyzing_modules" | "generating_getting_started" 
          | "generating_quick_ref" | "completed",
        steps_completed: 3,
        total_steps: 8,
        message: "Analyzing architecture patterns..."
      },
      error?: "Error description if failed"
    }
  }

===================================================
ENDPOINT 3: GET /api/repos/{repo_id}
===================================================
Purpose: Get full repo analysis (after completion)
Response (200):
  {
    success: true,
    data: {
      repo_id: "uuid",
      repo_name: "my-project",
      status: "completed",
      created_at: "ISO timestamp",
      file_stats: {
        total_files: 142,
        analyzed_files: 87,
        skipped_files: 55,
        languages: { "TypeScript": 45, "Python": 12, ... },
        total_lines: 15000
      },
      modules: [
        {
          id: "tech-stack",
          title: "Tech Stack Overview",
          description: "Languages, frameworks, and dependencies",
          icon: "layers",
          estimated_minutes: 5,
          content_markdown: "## Tech Stack\n...",
          order: 1,
          is_completed: false
        },
        {
          id: "architecture",
          title: "Architecture Guide",
          description: "How the codebase is organized",
          icon: "git-branch",
          estimated_minutes: 8,
          content_markdown: "## Architecture\n...",
          order: 2,
          is_completed: false
        },
        {
          id: "getting-started",
          title: "Getting Started",
          description: "How to set up and run the project",
          icon: "play",
          estimated_minutes: 5,
          content_markdown: "## Getting Started\n...",
          order: 3,
          is_completed: false
        },
        {
          id: "key-modules",
          title: "Key Modules Deep Dive",
          description: "What each major module does",
          icon: "box",
          estimated_minutes: 12,
          content_markdown: "## Key Modules\n...",
          order: 4,
          is_completed: false
        },
        {
          id: "coding-conventions",
          title: "Coding Conventions",
          description: "Patterns and standards in this codebase",
          icon: "file-code",
          estimated_minutes: 5,
          content_markdown: "## Coding Conventions\n...",
          order: 5,
          is_completed: false
        },
        {
          id: "quick-reference",
          title: "Quick Reference",
          description: "Key files, commands, and environment setup",
          icon: "bookmark",
          estimated_minutes: 3,
          content_markdown: "## Quick Reference\n...",
          order: 6,
          is_completed: false
        }
      ]
    }
  }
Errors:
  - 404: "Repo not found"
  - 202: If still processing, return status (same as endpoint 2)

===================================================
ENDPOINT 4: POST /api/repos/{repo_id}/chat
===================================================
Purpose: Employee asks a question about the codebase
Request:
  {
    message: "Where is authentication handled?",
    current_module_id?: "architecture",  // optional context
    conversation_id?: "uuid"  // for multi-turn, optional for MVP
  }
Response (200):
  {
    success: true,
    data: {
      response: "Authentication is handled in the `src/auth/` 
        directory. The main middleware is in...",
      referenced_files: [
        { path: "src/auth/middleware.ts", lines: "24-45" },
        { path: "src/config/auth.config.ts", lines: "1-15" }
      ],
      conversation_id: "uuid"
    }
  }

===================================================
ENDPOINT 5: PATCH /api/repos/{repo_id}/modules/{module_id}/complete
===================================================
Purpose: Toggle module completion
Request: { is_completed: true }
Response: { success: true }

===================================================
ENDPOINT 6: GET /api/repos/{repo_id}/export
===================================================
Purpose: Export all modules as a single markdown file
Response: text/markdown file download

===================================================
ENDPOINT 7: POST /api/repos/{repo_id}/publish
===================================================
Purpose: Manager "publishes" — generates a share code
Response:
  {
    success: true,
    data: {
      share_code: "ROCKET-a1b2c3",
      share_url: "/onboard/ROCKET-a1b2c3"
    }
  }

===================================================
ENDPOINT 8: GET /api/onboard/{share_code}
===================================================
Purpose: Employee accesses published repo via share code
Response: Same as GET /api/repos/{repo_id} (the full analysis)

SECTION 3: BACKEND SPECIFICATION (P2)

===================================================
DATABASE SCHEMA (SQLite):
===================================================

Table: repos
  - id: TEXT PRIMARY KEY (uuid4)
  - repo_name: TEXT
  - status: TEXT ('processing' | 'completed' | 'failed')
  - current_step: TEXT
  - steps_completed: INTEGER (default 0)
  - total_steps: INTEGER (default 8)
  - progress_message: TEXT
  - file_stats_json: TEXT (JSON string of file_stats object)
  - share_code: TEXT (nullable, unique)
  - error_message: TEXT (nullable)
  - created_at: TEXT (ISO timestamp)
  - completed_at: TEXT (nullable)

Table: modules
  - id: TEXT PRIMARY KEY (e.g., "{repo_id}-tech-stack")
  - repo_id: TEXT (FK to repos.id)
  - module_key: TEXT ('tech-stack' | 'architecture' | 
    'getting-started' | 'key-modules' | 'coding-conventions' | 
    'quick-reference')
  - title: TEXT
  - description: TEXT
  - icon: TEXT
  - estimated_minutes: INTEGER
  - content_markdown: TEXT
  - order_index: INTEGER
  - is_completed: BOOLEAN (default false)

Table: chat_messages
  - id: TEXT PRIMARY KEY (uuid4)
  - repo_id: TEXT (FK)
  - conversation_id: TEXT
  - role: TEXT ('user' | 'assistant')
  - content: TEXT
  - referenced_files_json: TEXT (nullable, JSON)
  - created_at: TEXT

Table: repo_files
  - id: TEXT PRIMARY KEY (uuid4)
  - repo_id: TEXT (FK)
  - file_path: TEXT
  - language: TEXT
  - content: TEXT
  - line_count: INTEGER
  - file_type: TEXT ('source' | 'config' | 'docs' | 'test' | 'other')
  - is_key_file: BOOLEAN
  PURPOSE: Indexed repo contents for Q&A retrieval

===================================================
ZIP PROCESSOR SERVICE (zip_processor.py):
===================================================

def process_zip(file_path: str, repo_id: str) -> dict:

STEP 1: Extract zip to /data/repos/{repo_id}/

STEP 2: Walk the file tree. SKIP these paths entirely:
  DENY_LIST (directories):
    node_modules/, .git/, __pycache__/, .next/, .nuxt/,
    dist/, build/, out/, .cache/, .vscode/, .idea/,
    vendor/, .tox/, .mypy_cache/, .pytest_cache/,
    coverage/, .nyc_output/, .terraform/, 
    venv/, env/, .env/, .venv/

  DENY_LIST (file patterns):
    *.lock (package-lock.json, yarn.lock, poetry.lock, etc.)
    *.min.js, *.min.css (minified files)
    *.map (source maps)
    *.png, *.jpg, *.jpeg, *.gif, *.svg, *.ico, *.webp (images)
    *.woff, *.woff2, *.ttf, *.eot (fonts)
    *.pdf, *.zip, *.tar, *.gz (binaries/archives)
    *.pyc, *.pyo, *.class, *.o, *.so, *.dll (compiled)
    .DS_Store, Thumbs.db
    *.log

  ALLOW_LIST (config files — always include even if small):
    package.json, tsconfig.json, vite.config.*, webpack.config.*,
    requirements.txt, setup.py, pyproject.toml, Cargo.toml,
    Dockerfile, docker-compose.yml, .env.example,
    Makefile, Procfile, Gemfile,
    README.md, README.rst, CONTRIBUTING.md,
    .eslintrc.*, .prettierrc.*, jest.config.*,
    next.config.*, nuxt.config.*

STEP 3: For each included file:
  - Read content (UTF-8, skip if binary/decode fails)
  - Truncate individual files at 500 lines (keep first 500)
  - Classify: source / config / docs / test / other
  - Detect language from extension
  - Flag as is_key_file if it's a config, entry point, or README
  - Store in repo_files table

STEP 4: Compute file_stats:
  - total_files, analyzed_files, skipped_files
  - languages breakdown (count per language)
  - total_lines

STEP 5: Return:
  {
    repo_root: "/data/repos/{repo_id}",
    file_tree: [list of relative paths],
    file_stats: {...},
    key_files: [list of config/entry/readme file contents],
    source_files: [list of source file contents with paths]
  }

FILE SIZE GUARD:
- If zip is > 100MB: reject immediately
- If extracted content exceeds 50,000 total lines after filtering: 
  keep only key_files + top 100 source files sorted by:
  1. is_key_file (configs first)
  2. file_type = 'source' (prioritize source over test)
  3. shorter file paths (closer to root = more important)
  4. smaller files first (more likely to be focused/important)
- Log how many files were skipped in file_stats

===================================================
ANALYSIS ORCHESTRATOR (ai_analyzer.py):
===================================================

This is the core pipeline. Called async after upload.
P2 owns the orchestration. P3 owns the prompts.

async def analyze_repo(repo_id: str):
  
  # 1. Extract + index
  update_status(repo_id, "extracting", 0, "Extracting files...")
  processed = process_zip(zip_path, repo_id)
  
  update_status(repo_id, "indexing", 1, "Indexing file structure...")
  store_files_in_db(repo_id, processed)
  
  # 2. Build context payloads for AI
  # KEY PRINCIPLE: Don't send ALL files to every prompt.
  # Send targeted context per analysis type.
  
  context = build_analysis_context(repo_id, processed)
  # context.config_files → for tech stack analysis
  # context.file_tree → for architecture analysis  
  # context.source_samples → for patterns/conventions
  # context.module_dirs → for module deep dive
  # context.setup_files → for getting started
  
  # 3. Run analysis prompts (SEQUENTIALLY to manage rate limits)
  
  update_status(repo_id, "analyzing_tech_stack", 2, 
    "Identifying technologies...")
  tech_stack = await call_claude(TECH_STACK_PROMPT, context.config_files)
  store_module(repo_id, "tech-stack", tech_stack)
  
  update_status(repo_id, "analyzing_architecture", 3, 
    "Mapping architecture...")
  architecture = await call_claude(ARCHITECTURE_PROMPT, 
    context.file_tree + context.config_files)
  store_module(repo_id, "architecture", architecture)
  
  update_status(repo_id, "generating_getting_started", 4, 
    "Building setup guide...")
  getting_started = await call_claude(GETTING_STARTED_PROMPT, 
    context.setup_files)
  store_module(repo_id, "getting-started", getting_started)
  
  update_status(repo_id, "analyzing_modules", 5, 
    "Analyzing key modules...")
  modules = await call_claude(MODULES_PROMPT, context.module_dirs)
  store_module(repo_id, "key-modules", modules)
  
  update_status(repo_id, "analyzing_patterns", 6, 
    "Detecting coding patterns...")
  patterns = await call_claude(PATTERNS_PROMPT, context.source_samples)
  store_module(repo_id, "coding-conventions", patterns)
  
  update_status(repo_id, "generating_quick_ref", 7, 
    "Generating quick reference...")
  quick_ref = await call_claude(QUICK_REF_PROMPT, 
    context.config_files + context.setup_files)
  store_module(repo_id, "quick-reference", quick_ref)
  
  # 4. Mark complete
  update_status(repo_id, "completed", 8, "Analysis complete!")

TOKEN BUDGET STRATEGY:
  Each Claude call should use targeted context, not everything.
  Approximate budget per call (Claude 3.5 Sonnet 200K context):
  
  | Analysis | Context to Send | Target Tokens |
  |---|---|---|
  | Tech Stack | config files + top of README | ~5K |
  | Architecture | file tree + config + README | ~10K |
  | Getting Started | README + Dockerfile + configs + scripts | ~8K |
  | Key Modules | top-level dirs + index files + first 50 lines per | ~30K |
  | Conventions | 10-15 representative source files | ~20K |
  | Quick Reference | configs + scripts + env files | ~5K |
  | Q&A (per question) | relevant files (retrieved) + question | ~15K |

===================================================
Q&A RETRIEVAL SERVICE:
===================================================

For MVP, use SIMPLE KEYWORD + PATH MATCHING (not embeddings):

def retrieve_relevant_files(repo_id, question, current_module=None, 
                            max_files=10, max_tokens=15000):
  
  # 1. Extract keywords from question
  #    Simple: split on spaces, remove stopwords, lowercase
  keywords = extract_keywords(question)
  
  # 2. Score each file in repo_files by relevance:
  for file in all_repo_files:
    score = 0
    # Path match: if any keyword appears in file path
    score += sum(3 for kw in keywords if kw in file.path.lower())
    # Content match: if any keyword appears in file content
    score += sum(1 for kw in keywords if kw in file.content.lower())
    # Key file bonus
    if file.is_key_file: score += 2
    # Current module context bonus
    if current_module and module_relates_to_file(current_module, file):
      score += 2
  
  # 3. Sort by score descending, take top max_files
  # 4. Truncate total content to max_tokens (character estimate)
  # 5. Return file contents with paths
  
  return relevant_files

This is crude but effective for a 4-hour MVP. Works surprisingly 
well for direct questions like "Where is auth handled?" because 
"auth" will match file paths like src/auth/*.

===================================================
FASTAPI SETUP:
===================================================

main.py:
- CORS middleware (allow frontend origin)
- Mount routers
- Background tasks for async analysis
- Static file serving for extracted content (if needed)

Run analysis as a BackgroundTask:
  @router.post("/repos/upload")
  async def upload(file: UploadFile, background_tasks: BackgroundTasks):
      repo_id = str(uuid4())
      save_zip(file, repo_id)
      create_repo_record(repo_id, status="processing")
      background_tasks.add_task(analyze_repo, repo_id)
      return {"repo_id": repo_id, "status": "processing"}

===================================================
BACKEND ACCEPTANCE CRITERIA:
===================================================

[ ] Zip upload saves file and starts background analysis
[ ] File filtering correctly skips node_modules, .git, binaries, etc.
[ ] Large repos (50K+ lines) are truncated gracefully
[ ] Status endpoint returns real-time progress
[ ] All 6 modules generated and stored
[ ] Chat endpoint returns contextual answers with file references
[ ] Module completion toggle works
[ ] Export endpoint returns combined markdown
[ ] Publish creates share code, onboard endpoint resolves it
[ ] Errors return proper HTTP codes + messages
[ ] CORS configured for frontend

SECTION 4: AI PROMPTS (P3)

===================================================
CLAUDE API CONFIGURATION:
===================================================

Model: claude-sonnet-4-20250514 (or claude-3-5-sonnet-20241022)
All calls use:
  - temperature: 0.2 (low creativity, high accuracy)
  - max_tokens: 4096 per module generation
  - max_tokens: 2048 per Q&A response

Wrapper function:
async def call_claude(system_prompt: str, context: str, 
                       user_prompt: str = "") -> str:
    start = time.time()
    try:
        response = client.messages.create(
            model="claude-sonnet-4-20250514",
            max_tokens=4096,
            system=system_prompt,
            messages=[{"role": "user", "content": context + 
                       ("\n\n" + user_prompt if user_prompt else "")}]
        )
        duration = int((time.time() - start) * 1000)
        log_generation(function_name, duration, success=True)
        return response.content[0].text
    except Exception as e:
        log_generation(function_name, 0, success=False, error=str(e))
        raise

===================================================
PROMPT 1: TECH STACK ANALYSIS
===================================================

SYSTEM PROMPT:
"You are a senior software engineer analyzing a codebase to help 
new team members onboard. Generate a clear, structured Tech Stack 
Overview in Markdown.

RULES:
- Base ALL analysis on the actual files provided. Do NOT guess or 
  assume technologies not evidenced in the code.
- Be specific: include version numbers when visible in configs.
- Explain WHY each technology is likely used (its role in the project).
- Group into categories: Language, Framework, Database, Testing, 
  DevOps/Infra, Key Libraries.
- For each technology, write 1-2 sentences explaining its role.
- Use ## headers for categories, bullet points for technologies.
- Include a 'Summary' section at the top (3 sentences max).
- Tone: friendly expert explaining to a competent developer 
  who's new to THIS project."

USER CONTEXT: [Inject config file contents: package.json, 
requirements.txt, Cargo.toml, go.mod, Gemfile, docker files, 
README top section, etc.]

===================================================
PROMPT 2: ARCHITECTURE ANALYSIS
===================================================

SYSTEM PROMPT:
"You are a senior software architect analyzing a codebase for 
onboarding documentation. Generate a clear Architecture Guide 
in Markdown.

ANALYZE AND INCLUDE:
1. **High-Level Overview** (3-5 sentences: what this project is 
   and how it's structured)
2. **Directory Structure** — show the top-level and key nested 
   directories as a tree, with 1-line descriptions:


   SECTION 5: FRONTEND SPECIFICATION (P1)

   ===================================================
ROUTES:
===================================================

/                     → Manager Upload Page
/repo/:repoId/review → Manager Review Page (analysis results)
/onboard/:shareCode  → Employee Dashboard (shared link)
/repo/:repoId        → Employee Dashboard (direct access for demo)

No auth. No login. Single-tenant demo.

===================================================
PAGE 1: UPLOAD PAGE (/)
===================================================

LAYOUT: Centered, single-column, max-w-2xl

- Header: "🚀 RocketBoard" logo + tagline "Stop reading code. 
  Start understanding it."
  
- Hero section:
  "Onboard developers into any codebase — in minutes, not weeks."
  "Upload a GitHub repo zip. Get an AI-generated interactive 
  onboarding course."

- Upload zone (large, prominent):
  - Drag-and-drop area (dashed border, slate-700, rounded-xl)
  - Icon: Upload cloud
  - Text: "Drop your repo .zip here"
  - Subtext: "or click to browse · Max 100MB · .zip files only"
  - Browse button embedded
  - On file select: show filename + file size + ✓ icon
  - Optional: "Repo name" text input (auto-filled from zip name)
  
- "Analyze Repository →" button (indigo, full-width, large)
  - Disabled until file selected
  - On click: uploads file, shows progress page

- Below upload: "How it works" mini-steps:
  1. Upload → 2. AI Analyzes → 3. Share with team

- Footer: "Powered by Claude AI · Built at [Hackathon Name]"

ON UPLOAD:
- POST /api/repos/upload
- On 202: redirect to /repo/{repoId}/review

===================================================
PAGE 2: ANALYSIS PROGRESS + REVIEW (/repo/:repoId/review)
===================================================

This page has TWO STATES: analyzing and completed.

STATE 1: ANALYZING (status !== 'completed')
- Header: "🚀 RocketBoard" + repo name
- Centered card:
  - "Analyzing [repo_name]..."
  - Animated progress bar (steps_completed / total_steps)
  - Current step label: e.g., "Mapping architecture..."
  - Step checklist below progress bar:
    ☑ Extracting files
    ☑ Indexing structure
    ☐ Analyzing tech stack ← (current, pulsing)
    ☐ Mapping architecture
    ☐ Generating setup guide
    ☐ Analyzing key modules
    ☐ Detecting patterns
    ☐ Building quick reference
  - Subtle animation: spinning icon or pulsing dots on current step
  
- Poll GET /api/repos/{repoId}/status every 2 seconds
- On status === 'completed': transition to STATE 2
- On status === 'failed': show error + "Try Again" button

STATE 2: COMPLETED (Manager Review)
- Header: "🚀 RocketBoard" + repo name + "Analysis Complete ✅"
- Top bar: file stats summary:
  "[87] files analyzed · [15,000] lines · [5] languages detected"
  
- Module cards grid (2 columns on desktop, 1 on mobile):
  For each module:
  ┌───────────────────────────────┐
  │ [icon]  [Title]               │
  │ [description]                 │
  │ ⏱️ [X] min read               │
  │                               │
  │ [Preview →]                   │
  └───────────────────────────────┘
  
  Click "Preview" → expands card to show content_markdown rendered 
  (accordion or modal). Manager can read through to verify quality.

- Action buttons at bottom:
  [📤 Publish & Share]  [📥 Download as Markdown]
  
- "Publish & Share" flow:
  - POST /api/repos/{repoId}/publish
  - Shows modal with share code + share URL:
    "Share this link with your new team member:"
    [https://rocketboard.app/onboard/ROCKET-a1b2c3]
    [Copy Link] button
  
- "Download as Markdown":
  - GET /api/repos/{repoId}/export
  - Triggers .md file download

===================================================
PAGE 3: EMPLOYEE DASHBOARD (/onboard/:shareCode or /repo/:repoId)
===================================================

LAYOUT: Two-panel (sidebar + main content) on desktop. 
Single panel with nav on mobile.

LEFT SIDEBAR (280px, desktop):
- "🚀 RocketBoard" mini-logo
- Repo name (bold)
- File stats: "[87] files · [5] languages"
- Divider
- Module navigation list:
  For each module:
    [icon] [Title]
    [completion checkbox ☐/☑] · [X min]
  Highlight current module (indigo bg)
  Completed: green check
- Divider
- Progress summary: "[2] of [6] complete"
- Mini progress bar
- Bottom: "💬 Ask about this code" button (opens chat)

MAIN CONTENT AREA:

SUB-STATE A: Module View (default)
- Top: Module title (H1) + description + estimated time + 
  type icon
- Content: Render content_markdown as rich Markdown:
  - ## headers with good spacing
  - Code blocks with syntax highlighting (use a syntax 
    highlighting library like react-syntax-highlighter 
    or shiki — IMPORTANT for a dev tool)
  - ```tree``` blocks rendered as monospace
  - Tables rendered properly
  - Inline code with distinct background
  - Blockquotes styled as callout cards
- Bottom: 
  [Mark as Complete ✓] button (toggles is_completed)
  [← Previous Module] [Next Module →]
- On "Mark Complete": PATCH /api/repos/.../complete
  - Button turns green: "Completed ✓"
  - Sidebar updates checkbox
  - If all modules complete: show celebration banner:
    "🎉 Onboarding Complete! You're ready to contribute."

SUB-STATE B: Chat Panel
- Triggered by "Ask about this code" button or a persistent 
  floating button (bottom-right)
- IMPLEMENTATION: Slide-out panel from right (desktop) or 
  full-screen modal (mobile)
- Chat interface:
  - Message list (scrollable)
  - Bot messages: left-aligned, slate-800 bg
  - User messages: right-aligned, indigo-600 bg
  - Referenced files shown as clickable chips below bot message:
    📄 src/auth/middleware.ts:24-45
    (clicking a chip could scroll to that file reference or 
    just be informational for MVP)
  - Input: text input + send button (Enter to send)
  - Placeholder: "Ask anything about this codebase..."
  - Suggested question chips (shown when chat is empty):
    "What does this project do?"
    "How is the code organized?"
    "Where is [specific feature] handled?"
    "How do I run the tests?"
  - Auto-include current_module_id in API call for context
  
  POST /api/repos/{repoId}/chat with each message

===================================================
COMPONENT LIBRARY:
===================================================

Build these reusable components:

1. UploadZone — drag & drop file upload
2. ProgressTracker — animated step list with progress bar
3. ModuleCard — module preview card (icon, title, desc, time)
4. MarkdownRenderer — renders AI markdown with syntax highlighting
5. ChatPanel — slide-out chat interface
6. FileChip — clickable file reference badge
7. ModuleSidebar — navigation sidebar with completion state
8. ShareModal — shows share code + copy button

===================================================
API CLIENT (src/api/client.ts):
===================================================

Typed functions matching the API contract:

export async function uploadRepo(file: File, repoName?: string): 
  Promise<{ repo_id: string }>

export async function getRepoStatus(repoId: string): 
  Promise<RepoStatus>

export async function getRepo(repoId: string): 
  Promise<RepoData>

export async function getRepoByShareCode(code: string): 
  Promise<RepoData>

export async function sendChatMessage(
  repoId: string, message: string, moduleId?: string
): Promise<ChatResponse>

export async function toggleModuleComplete(
  repoId: string, moduleId: string, completed: boolean
): Promise<void>

export async function publishRepo(repoId: string): 
  Promise<{ share_code: string, share_url: string }>

export async function exportRepo(repoId: string): 
  Promise<Blob>

===================================================
STATE MANAGEMENT:
===================================================

Keep it simple (no Redux, no complex state):

- useRepo(repoId) hook — fetches + caches repo data via 
  Tanstack Query
- useRepoStatus(repoId) hook — polls status every 2s, stops 
  on complete/fail
- useChat(repoId) hook — manages chat messages array + send 
  function
- Simple React state for: current module, sidebar open/closed, 
  chat open/closed

===================================================
RESPONSIVE DESIGN:
===================================================

Desktop (1024px+): sidebar + main content side by side
Tablet (768-1023px): collapsible sidebar (hamburger toggle)
Mobile (<768px): no sidebar — top dropdown for module nav, 
  full-screen chat modal

===================================================
FRONTEND ACCEPTANCE CRITERIA:
===================================================

[ ] Upload page: drag-drop works, file validation (.zip, <100MB)
[ ] Upload triggers API call and redirects to review page
[ ] Progress page: polls status, shows animated steps
[ ] Progress page: transitions smoothly to review on completion
[ ] Review page: shows file stats + module cards
[ ] Review page: module preview (expand/modal) renders markdown
[ ] Publish generates share code + copy-to-clipboard works
[ ] Export downloads .md file
[ ] Employee dashboard: sidebar navigation works
[ ] Module content renders with syntax-highlighted code blocks
[ ] Mark Complete toggles correctly, sidebar updates
[ ] All modules complete: celebration message shows
[ ] Chat panel opens/closes, sends messages, shows responses
[ ] Chat shows file reference chips
[ ] Chat suggested questions shown on empty state
[ ] Mobile responsive: all pages work at 375px
[ ] Loading states: skeleton/spinner on all async operations
[ ] Error states: friendly messages + retry for failures


SECTION 6: INTEGRATION + DEMO PREP

===================================================
HOUR-BY-HOUR EXECUTION PLAN (REVISED):
===================================================

HOUR 1: FOUNDATIONS (parallel work)
  P1: Vite + React project setup, routing, UploadZone component, 
      basic layout/design system, API client stubs
  P2: FastAPI project setup, SQLite models, zip upload endpoint, 
      zip_processor service (extract + filter + index), status 
      endpoint
  P3: Set up Claude API client, write + test TECH_STACK_PROMPT 
      and ARCHITECTURE_PROMPT against a sample repo, build 
      call_claude wrapper with logging

  SYNC at :50 — P2 should have upload endpoint returning repo_id. 
  P1 should have upload page calling it.

HOUR 2: CORE PIPELINE (integration begins)
  P1: Progress/review page (polls status, renders module cards, 
      markdown renderer with syntax highlighting)
  P2: Analysis orchestrator (calls P3's prompts sequentially, 
      updates status, stores modules), GET /repo endpoint
  P3: Write remaining prompts (GETTING_STARTED, MODULES, PATTERNS, 
      QUICK_REF), test each, tune for quality. Start Q&A prompt + 
      retrieval function.

  SYNC at :50 — Full pipeline should work: upload → analyze → 
  view modules. Even if prompts need tuning, the flow works.

HOUR 3: EMPLOYEE EXPERIENCE + CHAT
  P1: Employee dashboard (sidebar, module viewer, completion 
      tracking), chat panel UI
  P2: Chat endpoint (with retrieval), completion endpoint, 
      publish endpoint, export endpoint
  P3: Tune Q&A prompt, test with real questions, improve 
      retrieval relevance. Pre-generate demo data as backup.

  SYNC at :50 — Full employee flow works. Chat returns answers. 
  Publish generates share code.

HOUR 4: POLISH + DEMO PREP
  P1: Polish UI, fix responsive issues, add loading/error states, 
      add celebration on completion
  P2: Bug fixes, error handling, test edge cases (large files, 
      weird repos), set up DEMO_MODE
  P3: Final prompt tuning, pre-seed backup demo data, test full 
      flow end-to-end

  LAST 30 MIN: All three together — run the demo flow 3 times. 
  Fix any breaks. Prep talking points.

===================================================
DEMO SCRIPT (60-90 seconds):
===================================================

1. HOOK (10s): "Every engineering team wastes 2-4 WEEKS onboarding 
   new developers. They say 'just read the code.' We built something 
   better."

2. PROBLEM (10s): "New hires stare at unfamiliar codebases with no 
   guide, no structure, no idea where to start. Leads are too busy 
   to hand-hold."

3. DEMO — Manager Flow (20s):
   - "A lead developer drops in their repo zip..."
   [upload real repo]
   - "In under a minute, RocketBoard AI analyzes the entire codebase..."
   [show progress animation]
   - "...and generates a complete onboarding course."
   [show review page with 6 modules]
   - "They publish it with one click and share a link."
   [show share code]

4. DEMO — Employee Flow (25s):
   - "The new hire opens their personalized onboarding link..."
   [show dashboard]
   - "They get structured modules: architecture, setup guide, 
   key modules — all generated from the actual code."
   [click through a module, show code snippets]
   - "And when they have questions..."
   [open chat, ask "Where is authentication handled?"]
   - "...the AI answers with specific file references from 
   THEIR codebase."
   [show response with file chips]

5. TRACTION/VALIDATION (10s): 
   "We tested this on [X] repos today. [Y] developers said they'd 
   use this for their next hire."

6. CLOSE (10s): "RocketBoard. Stop reading code. Start understanding 
   it."

===================================================
DEMO SAFETY NETS:
===================================================

[ ] Pre-analyzed repo ready (Express.js or similar recognizable repo)
[ ] DEMO_MODE flag tested — loads pre-seeded data if Claude fails
[ ] Demo flow rehearsed 3+ times
[ ] Backup: if upload is slow, start demo from pre-loaded review page
[ ] Have 2-3 prepared Q&A questions that give impressive answers
[ ] Test on the actual presentation machine/network

===================================================
INTEGRATION TESTING CHECKLIST:
===================================================

[ ] Upload .zip → backend receives and starts processing
[ ] Frontend polls status → shows real-time progress
[ ] Analysis completes → frontend transitions to review
[ ] All 6 modules have content (no empty/error modules)
[ ] Publish → share code generated → share URL works
[ ] Employee dashboard loads via share code
[ ] Module navigation works (sidebar click → content changes)
[ ] Mark complete → persists → sidebar updates
[ ] Chat → sends message → receives contextual response
[ ] Chat references actual files from the repo
[ ] Export → downloads valid markdown file
[ ] Error case: invalid zip → friendly error message
[ ] Error case: Claude fails → fallback content shown

===================================================
POTENTIAL MOLLIE INTEGRATION (if needed for hackathon):
===================================================

If this is a Mollie hackathon and you need payment integration:

PRICING MODEL: Pay-per-analysis
  - Free: 1 repo analysis (demo/trial)
  - Pro: €9.99/month — unlimited repos
  - Team: €29.99/month — unlimited repos + team sharing

MINIMAL INTEGRATION:
  - Before upload, check if user has a valid session/payment
  - If free tier used: show "Upgrade to analyze more repos"
  - Mollie checkout for subscription
  - This can be added as a thin layer WITHOUT auth — use a 
    browser session/cookie with an email input

But honestly: if this is a 4-hour build, skip payments and focus 
on the core experience being flawless. Mention pricing in the pitch 
as "planned monetization."


