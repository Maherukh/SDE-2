All‑in‑one repo regeneration prompt for a 36‑week Study Dashboard + Plan tools, tuned for SDE2 @ Google

You are an expert full‑stack engineer, curriculum designer, and dev‑tools builder. Create a complete repository from scratch that implements:
- A local, no‑backend study dashboard app (HTML/CSS/JS) with per‑task “Hint” notes and “Copy AI Prompt”
- Scripts to generate and enrich a dated 36‑week plan
- A GitHub issues/milestones creator (optional, via gh CLI)
- A robust AI prompt builder that defaults to “today,” attaches real code context from a local git repo, and auto‑names output Markdown files
- A dated, detailed 36‑week curriculum document and a resources catalog, tailored for SDE2 @ Google
- Clear docs and a reusable AI prompt template

Follow this spec exactly. Output every file as a code block with the file name in the header. Do not omit any file.

Parameters (edit these at the top of the files if needed)
- ROLE: SDE2-Google-Full-Stack (default; adapt phases for Google SDE2 expectations)
- INTERVIEW_LANGUAGE: Java (one of: Java | C++ | Go) for coding interviews, examples, and DSA snippets
- START_DATE: 2025-08-14 (ISO YYYY-MM-DD)
- WEEKS: 36
- TASKS_PER_DAY: 2 (fixed)
- GH_REPO: myuser/fs-bootcamp-dashboard (used by the issues script; set to your repo)
- STACK: Next.js 14 + TypeScript, Tailwind, Prisma + Postgres, Auth.js, Playwright/Vitest, Azure preferred (ok: AWS/GCP)

Files to produce (must include all)
- README.md
- index.html
- style.css
- app.js
- package.json
- scripts/generate-plan.mjs
- scripts/enrich-plan.mjs
- scripts/create-issues-from-plan.mjs
- scripts/build-ai-prompt.mjs
- scripts/task-library.json
- docs/ai-prompt-template.md
- curriculum-36-weeks-dated-START_DATE.md
- resources-catalog.md

Global non‑functional requirements
- Node 20+ (ESM only). No external runtime dependencies required.
- Works offline; dashboard uses LocalStorage (auto‑save).
- JSON files must be valid (no comments). The enricher must accept commented JSON by stripping comments before parsing.
- Reasonable a11y in the dashboard (labels, contrast, keyboard focus).
- Keep code readable, with inline comments where helpful.

A) Dashboard app (index.html, style.css, app.js)
- Purpose: Plan, track, and debug your study across 36 weeks.
- Features:
  - Controls: duration selector (4/8/12/36 weeks), start date, daily hours (default 2)
  - Actions: Generate Plan, Save, Reset, Export JSON, Import JSON
  - Summary: Completed/Total, % complete, streak days
  - Today section: list today’s tasks with:
    - Checkbox for done
    - Tag, module
    - Collapsible Details: Steps, Definition of Done (DoD)
    - Hint textarea: freeform notes (errors, stack traces, what you tried). Persist hints.
  - Plan grid: all days with progress bar, per‑day “Copy AI Prompt” button
  - “Copy AI Prompt” buttons:
    - For Today and per‑day cards
    - Prompts include meta, today’s tasks, Steps/DoD, user “Hint” notes
  - Pomodoro timer: 25/5 with start/pause/reset
- Data shape:
  - plan.meta: { weeks:number, dailyHours:number, generatedAt:string, version:number }
  - plan.days: [{ dateISO:"YYYY-MM-DD", tasks:[{ title, tag, module?, done:boolean, steps?:string[], definitionOfDone?:string[], hint?:string }] }]
- UX details:
  - Auto‑add hint fields on import if missing
  - Copy to clipboard using navigator.clipboard with fallback
  - Escape text safely for HTML where needed

B) Plan generator (scripts/generate-plan.mjs)
- Command:
  - node scripts/generate-plan.mjs --start START_DATE --weeks WEEKS --tasks-per-day TASKS_PER_DAY --out study-plan-START_DATE.json
- Behavior:
  - 6‑day study cadence per 7‑day block (skip day 7)
  - Exactly 2 tasks per study day
  - Day 6 is Interview Prep: (1) DSA topic, (2) System Design topic
  - Mon–Fri pull from the week’s “phase pool” appropriate for ROLE=SDE2-Google-Full-Stack
- SDE2 overlays built into the generator:
  - Add a daily DSA practice slot (45–90 min) guidance in meta notes (for schedule overlays; tasks stay at 2/day)
  - Prefer INTERVIEW_LANGUAGE examples in any code‑oriented instructions within steps (Java by default)
- meta.version = 3; meta.dailyHours default 2

C) Task library (scripts/task-library.json)
- Provide steps[] and definitionOfDone[] for recurring tasks used in phases:
  - Foundations/Next/TS tasks (TS types/narrowing; Next App Router; React; Tailwind; Git; HTTP; A11y; SEO; Env & scripts; RHF+Zod; Server vs Client; State patterns)
  - CRUD/Auth/DB/Testing tasks (Postgres; Prisma; Route Handlers+Zod; Auth.js; pagination/sorting; uploads; unit/component/E2E; error handling; validation; seeds)
  - Files/CI/CD/Perf tasks (CI; docs; perf/CWV; bundling; a11y deep dive; caching/CDN; monitoring/logging/security/rate limiting; SEO perf)
  - Realtime/Observability/Security tasks (websockets; presence; metrics/tracing; auth hardening; OWASP; audit/activity; backpressure/idempotency)
  - Kanban/Background Jobs tasks (DnD a11y; kanban model; jobs/queues; cron; multitenancy; feature flags; optimistic UI; attachments; error recovery; indexing; security headers)
  - Payments/Advanced Testing/Docker tasks
  - AWS+IaC tasks
  - GCP+Perf/Reliability tasks
  - Capstone/Interview polish tasks
- Must include byTag templates at minimum: DSA, Design (system design), Next.js, React
- JSON must be valid (no // comments)

D) Plan enrichment (scripts/enrich-plan.mjs)
- Command:
  - node scripts/enrich-plan.mjs --in study-plan-START_DATE.json --library scripts/task-library.json --out study-plan-START_DATE-detailed.json
- Behavior:
  - Attach steps and DoD by exact title match; fallback to tag‑level templates
  - Special cases: titles that begin with “DSA practice:” and “System design study:” use dedicated templates
  - Preserve existing task.hint if present
  - Accept commented JSON in the library by stripping // and /* */ comments safely (don’t strip inside strings)
  - meta.version += 0.1, add enrichedAt ISO timestamp

E) GitHub issues/milestones creator (scripts/create-issues-from-plan.mjs)
- Optional. Requires GitHub CLI (gh) authenticated.
- Command:
  - node scripts/create-issues-from-plan.mjs --repo GH_REPO --plan study-plan-START_DATE-detailed.json
- Behavior:
  - Create milestones “Week 1” … “Week N” with due_on = last study day of each week
  - Create one issue per study day:
    - Title includes week/day and date
    - Body contains task checklist with nested Steps and DoD and a “Notes / Hints” section
    - Label “study-plan”
  - Attach each issue to the correct milestone
- Safety: idempotent guards (skip if already exists by title), or “--force” to recreate

F) AI prompt builder (scripts/build-ai-prompt.mjs)
- Purpose: Build a high‑signal debugging prompt for a selected day, optionally appending repo code context via git, and auto‑write Markdown files.
- Day selection (priority):
  - --day N (1..plan.days.length)
  - --week W --weekday D (D in 1..6, study days only)
  - --date YYYY-MM-DD
  - Default: local “today”. If no exact match, pick latest past day; else first future; else last day. Log resolution.
- Code context collection:
  - If --repo provided, include branch and last commit
  - Gather relevant files (src/, app/, pages/, prisma/, key configs like package.json, tsconfig, next.config, schema.prisma, .env.example)
  - Truncate big files (head 200 lines + tail 50), enforce size budget --max-bytes (default ~100 KB)
- Output behavior:
  - If --out path provided: write exactly there (create dirs)
  - Else write to outdir (default ./prompts) as prompt-YYYY-MM-DD-wW-dD.md and also write/overwrite prompts/prompt-latest.md (unless --no-latest)
  - If dated file exists and not --overwrite, add -1, -2 suffix to avoid clobbering
  - Options: --outdir, --overwrite, --no-latest, --error "free text"
- Prompt content:
  - Title with Week/Day/date
  - Context: duration, hours, stack, and SDE2 interview language (INTERVIEW_LANGUAGE)
  - Today’s tasks with Steps, DoD, and user “Hint” notes
  - Additional error context from --error
  - Appended code context block
  - Explicit ask: ranked root‑cause hypotheses with evidence; minimal, correct diffs; tests/validation; risks/alternatives

G) Dated curriculum document (curriculum-36-weeks-dated-START_DATE.md)
- Include the entire 36‑week schedule with actual dates from START_DATE
- Format:
  - Weekly header with module
  - Day 1..Day 6 lines like: Day N (YYYY‑MM‑DD): Task A; Task B
  - Day 6 always shows: DSA topic + System Design topic
- Ensure the selection matches the generator’s rotation so it can be imported back if desired

H) Resources catalog (resources-catalog.md)
- For each topic family, provide:
  - Learning outcomes (what you’ll be able to do)
  - 1–3 curated resources, prioritizing official docs and a single high‑quality deep dive
- Include sections for: Foundations/Next/TS, CRUD/Auth/DB/Testing, Files/CI/CD/Perf, Realtime/Observability/Security, Kanban/Background Jobs, Payments/Advanced Testing/Docker, AWS+IaC, GCP+Perf/Reliability, Capstone/Interview polish, DSA, System Design, CS Fundamentals (OS/Networking/DB), and Mocking/interview practice

I) SDE2 @ Google overlays (baked into phases and docs)
- INTERVIEW_LANGUAGE focus (default Java); include snippets/tips in docs where relevant
- Algorithms depth/volume targets (document in README):
  - Week 12: ~75 mediums + 15 hards
  - Week 24: ~150 mediums + 40 hards
  - Week 36: 220–300 total, 60–80 hards across topics
  - Speed: easy ≤10m, medium ≤20m, hard ≤35m (timed drills)
- System design depth:
  - 12+ full designs across: URL shortener, news feed, chat, autocomplete, logging pipeline, object storage, rate limiter, payments flow, analytics counter, video streaming, notifications, collaborative editing
  - Each with requirements, APIs, data model, capacity estimates, partition/replication, consistency/failures, trade‑offs
- Behavioral/GCA guidance:
  - STAR stories (12–16) emphasizing ownership, debugging ambiguity, impact, mentorship
- 8‑week pre‑interview sprint overlay:
  - Mon–Fri: 2 mediums + 1 hard timed; flashcards 15 min
  - 3×/week: 30‑min design micro‑drill
  - Weekend: 1 full design mock + 2 hards
  - Weekly: 1 live coding mock + 1 live design mock

J) README.md (must cover)
- What this repo is, why it exists
- Quick start (open index.html), dashboard features (Hint, Copy AI Prompt, Pomodoro)
- Generate/enrich plan commands and expected outputs
- Optional GitHub issues script with gh CLI prerequisites
- AI prompt builder usage:
  - Defaults to today
  - Examples for --day, --week/--weekday, --date
  - Auto‑naming behavior and flags (--outdir/--overwrite/--no-latest)
- SDE2 overlays (language, DSA targets, system design practice, 8‑week sprint)
- Links to curriculum-36-weeks-dated-START_DATE.md and resources-catalog.md

K) AI prompt template (docs/ai-prompt-template.md)
- High‑signal structure:
  - Context, Today’s Tasks (with Steps/DoD/Hints), Error/Logs, Code Context, What I tried/ruled out, Ask, Output format
- Short tips for sharing focused code snippets or running the prompt builder to attach repo context

Acceptance criteria checklist (must satisfy all)
- Dashboard:
  - Import enriched plan (with Steps/DoD/Hint), show collapsible Details, editable Hint persists
  - “Copy AI Prompt” works for Today and per‑day
  - Pomodoro works
- Generators:
  - generate-plan.mjs: 6 study days/week, 2 tasks/day, Day‑6 DSA+System Design; phases aligned with SDE2 Google focus
  - enrich-plan.mjs: attaches Steps/DoD; preserves Hints; strips comments from library JSON safely
  - create-issues-from-plan.mjs: creates milestones and per‑day issues with nested checklists (when gh CLI is available)
  - build-ai-prompt.mjs:
    - Day selection via --day | --week/--weekday | --date | default “today” with fallback
    - Appends repo code context bounded by size
    - Auto‑writes prompts/prompt-YYYY-MM-DD-wW-dD.md and prompts/prompt-latest.md by default; supports --outdir/--overwrite/--no-latest and --out
- Curriculum and resources docs are present and coherent with generator output
- No external network calls needed at runtime (except gh for issues)

Output format requirements (strict)
- Use file blocks for every file. Put the file name in the header:
  - For code files: javascript name=scripts/file.mjs (or appropriate language)
  - For JSON: json name=scripts/task-library.json
  - For HTML/CSS/TS/JS: corresponding languages
  - For Markdown files: use FOUR backticks to open/close, with language “markdown” and name, so inner code blocks render correctly, e.g.:
    `markdown name=README.md
    …content that includes code blocks…
    `
- Do not omit any file. Do not collapse files. No placeholders; include complete implementations.

After you generate all files, include short “How to run” notes in README and ensure the package.json scripts work:
- npm run generate:plan
- npm run enrich:plan
- npm run issues:create
- npm run prompt:build

Now generate the full repository exactly per this spec, including all files listed above.
