# PEAK OPS — MASTER OPERATING CONTEXT
## Unified Cross-Project Standard | Living Document
### Sources: Mission Control (S1–S83) + Fitness Tracker (S1–S9) | Last Updated: 2026-06-29

> This document is the single source of truth for all operating rules, guardrails,
> plugins, and workflow patterns across every Peak Ops project. Paste it into the
> first Claude Chat message of any new project to bootstrap full context in under
> 5 minutes. It is a living document — update it at the close of any session where
> a new cross-project rule is established.

---

## HOW TO USE THIS DOCUMENT

**New project:** Paste this entire document into Claude Chat Session 1, followed by
your project context block. Claude Chat confirms machine-level setup, then work begins.

**Existing project:** Reference specific sections as needed. Do not re-paste the full
document — it lives in Claude Chat memory after Session 1.

**Update process:** At session close, Claude Chat flags any new cross-project rules.
Patrick confirms which go in this document. First 5 minutes of the next session,
the update is committed to the standards repo.

---

## SECTION 1 — MACHINE-LEVEL INSTALLS
*Install once on the Windows machine. Available to all projects automatically.*

### 1.1 Codex Plugin (OpenAI Codex CLI)

**What it is:** Provides `/codex:review`, `/codex:adversarial-review`, and
`/codex:rescue` inside every Claude Code terminal. The pre-commit quality gate.

**Install — run in PowerShell (NOT inside Claude Code terminal):**

```powershell
# Step 1: Open PowerShell on Windows machine

# Step 2: Run setup
/codex:setup

# Step 3: If certificate error appears — let it retry automatically
# When prompted select:
# "Yes, and don't ask again for: NODE_OPTIONS=--use-system-ca npm install *"

# Step 4: Authenticate with patpeak1@gmail.com (ChatGPT subscription covers this)

# Step 5: Confirm all green checkmarks:
# Node.js        checkmark
# npm            checkmark
# Codex CLI      checkmark
# Authentication checkmark
# Review gate: disabled  checkmark  (CRITICAL — do NOT enable review gate)

# Step 6: Close PowerShell — done
# Plugin is now available in ALL Claude Code terminals on this machine
```

**Verify:** Inside any Claude Code terminal run `/codex:review` on any file.
Should return findings or "no issues found" — not "command not found".

**Critical notes:**
- `/plugin marketplace add openai-codex` is blocked by security classifier when
  run autonomously inside Claude Code — this is intentional, always use PowerShell
- `codex:rescue` and `codex:review` are different commands — never substitute one for the other
- `codex:rescue` = delegates a problem to Codex as sub-agent
- `codex:review` = pre-commit quality gate
- Do NOT enable review gate mode — it intercepts every response and causes usage spikes
- After install, available across all projects on the machine — no per-project setup

**Commands after install:**

| Command | Purpose |
|---|---|
| `/codex:review` | Reviews diff before commit — use on every meaningful change |
| `/codex:adversarial-review` | Pressure-tests design decisions — use on major architecture only |
| `/codex:rescue` | Delegates a stuck problem to Codex as sub-agent |
| `/codex:rescue --background` | Same but async |

---

### 1.2 Global CLAUDE.md — Ponytail Skill

**What it is:** Loaded into every Claude Code session automatically.
Contains the YAGNI efficiency ruleset that governs all code decisions.

**Install — run in PowerShell:**

```powershell
# Create directory if needed
mkdir -Force "$env:USERPROFILE\.claude"

# Open and paste the Ponytail ruleset (Section 2.1 below)
notepad "$env:USERPROFILE\.claude\CLAUDE.md"

# Verify
cat "$env:USERPROFILE\.claude\CLAUDE.md"
```

**Verify:** Open a fresh Claude Code terminal. Ponytail should be active in context
without being explicitly told. Note: only active in terminals opened AFTER install.

---

### 1.3 Node.js + npm

**Required for:** Codex CLI, GitNexus, frontend builds.

```powershell
# Install LTS from nodejs.org
node --version   # v20+ required (v26.2.0 confirmed working)
npm --version    # v10+ required
```

---

### 1.4 GitNexus (Mission Control — carry forward if applicable)

**What it is:** Symbol graph indexer. Understands call graphs, symbol relationships,
and change blast radius. Run per-project but CLI is machine-level.

```bash
npm install -g @gitnexus/cli
# or without install:
npx gitnexus analyze
```

**Critical gotcha:** `npx gitnexus analyze` auto-regenerates the `<!-- gitnexus:start/end -->`
managed block in CLAUDE.md on every run, clobbering any hand-edits inside it.
Always place custom rules OUTSIDE the managed block — after `<!-- gitnexus:end -->`.

---

### 1.5 Claude in Chrome MCP Extension

**What it is:** Chrome extension giving Claude Chat control over the browser.
Used for UI verification after every deploy.

**Install:** Chrome Web Store → search "Claude in Chrome" → install → enable on all sites.

**Patrick's laptop device ID:** `1b7e6193-7cf1-42c8-9e4b-3bcca9ffb054`

See Section 5 for full Chrome MCP protocol.

---

## SECTION 2 — WORKFLOW RULES AND OPERATING MODEL

### 2.1 PONYTAIL — Global Efficiency Ladder

```
PONYTAIL — YAGNI EFFICIENCY LADDER
Before writing any code, stop at the first rung that holds:

1. Does this need to exist?              → no: skip it (YAGNI)
2. Does the standard library do it?      → use it
3. Is it a native platform feature?      → use it
4. Is it an installed dependency?        → use it
5. Does it already exist in this repo?   → reuse it
6. Can this be one line?                 → one line
7. Only then: write the minimum that works

Before fixing any bug:
- Understand the problem first — trace the real flow
- Bug fix = find root cause + grep every caller
- Never patch symptoms

NEVER compressed (always full implementation):
- Trust-boundary validation
- Data-loss handling
- Security checks
- Accessibility
- Error handling
- Tests for critical paths
```

---

### 2.2 The 4-Agent Operating Model

```
COORDINATOR: Claude Chat
- Architecture decisions, priority queue, session memory
- Writes terminal prompts, receives completion reports
- Reviews diffs before "Codex cleared, proceed with commit"
- Approves HIGH zone changes
- Runs Chrome MCP verification after every deploy
- Never executes code directly

EXECUTOR: Claude Code terminals (T1-T4)
- Autonomous execution within scoped task
- Self-verify, produce completion report, stop and wait
- Hard-stop at HIGH zone, flag and await coordinator approval
- Never chain autonomously into next task without coordinator assignment
- Questions output as plain numbered text — never pop-up UI widgets

REVIEW PARTNER: Codex (via /codex:review)
- Reviews diff before every commit — no exceptions
- Reports findings at P1/P2/P3 severity
- Falls back to inline self-review if unavailable (document which was used)

RELAY + APPROVER: Patrick (human)
- Pastes session-start prompts to Claude Chat
- Pastes task prompts to Claude Code terminals
- Relays completion reports from Claude Code to Claude Chat
- Approves HIGH zone checkpoints
- Monitors session bar at claude.ai/settings/usage on request
- Final verification sign-off on deploys

Terminal numbering convention:
T1 — Fitness Tracker project
T2 — Mission Control / any new project (first terminal)
T3, T4 — expand only for parallel-safe independent tasks (no shared files)
Max 5 terminals simultaneously
```

---

### 2.3 SESSION_BUDGET_RULE (Corrected — S83 Peak Ops)

```
SESSION_BUDGET_RULE:
The session bar % at claude.ai/settings/usage → "Current session"
is the ONLY authoritative signal for session budget.

Terminal token estimates are informational only.
Do NOT use them to mandate terminal restarts or stop work.

Fresh terminal = only when:
1. Coordinator (Claude Chat) instructs it, OR
2. Session bar exceeds 80% — confirmed by coordinator, not terminal, OR
3. Current task is fully complete and verified

Why terminal math is wrong:
Terminal reports ~80-85k tokens → recommends fresh terminal.
Actual session bar may show only 36% used with 3+ hours remaining.
Token math does not account for caching, context compression, or
the difference between context window and session budget.
Check the bar, not the math.

Note: Session budget is SHARED across Claude Chat AND all active
Claude Code terminals — parallel terminals burn it faster.
```

---

### 2.4 Git Ceremony Rules

```
GIT CEREMONY — REQUIRED FOR EVERY COMMIT:

PRE-WORK:
git fetch origin && git rebase origin/main
(Fitness Tracker uses main. Mission Control uses master.
Always confirm branch — never assume.)

git log --oneline -3
Confirm HEAD matches expected SHA before touching anything.

git status --short
Verify only intended files are dirty. If unexpected files appear —
stop, investigate, exclude via explicit pathspec.

BEFORE COMMIT:
1. Syntax check all modified files
   JavaScript: node --check <file> or npm run build
   Python: python -m py_compile <file>
   Never commit without a clean syntax pass.

2. /codex:review on all modified files
   If unavailable: inline self-review (re-read full diff, check for
   data loss, null cases, logic errors, dead imports)
   Document in commit: "/codex:review: exit 0" or
   "inline self-review: no findings"
   Never fabricate a review result.

3. Output diff in chat and wait for coordinator
   "Codex cleared, proceed with commit" before committing.

COMMIT:
git commit -m "type(scope): description" -- <explicit pathspec>
ALWAYS use explicit pathspec. NEVER git add -A or bare git commit.
Types: feat / fix / docs / chore / refactor / style / test
One terminal at a time on HIGH zone files.

PUSH:
Fitness Tracker: git push origin HEAD:main
Mission Control: git push origin HEAD:master
Never use railway up — it sets commitHash: None and breaks SHA verification.
Always push to GitHub and let the webhook trigger the deploy.

VERIFY:
Railway dashboard ACTIVE card + correct SHA + "Deployment successful"
= ground truth for deploy verification.
Do NOT rely on curl for health checks on Railway SPA — returns 200 for any path.
Poll /health endpoint for backend services.

POST-DEPLOY:
Clear PWA service worker before Chrome MCP UI verification.
(See Section 5 for exact steps.)
```

---

### 2.5 Codex Review Gate Protocol

```
CODEX_REVIEW_RULE:
Run /codex:review after every meaningful code change, before commit.
This is non-negotiable — it is what catches bugs before production.

MANDATORY GATE IN EVERY TERMINAL PROMPT:
1. Run git diff on all modified files
2. Output the full diff as plain text in the chat window
3. Run /codex:review on modified files
   If unavailable: inline self-review and document which method was used
4. Report findings by severity:
   [P1] = Blocking — fix before commit, re-review after
   [P2] = Should fix — inline if under 5 min, log as task if complex
   [P3] = Informational — note in commit message, no action required
5. Output findings in chat and wait for coordinator confirmation
6. Do not commit until coordinator sends: "Codex cleared, proceed with commit"

Never fabricate a review result. Never stall — use inline self-review fallback.
Never enable review gate mode (/codex:setup --enable-review-gate).
/codex:adversarial-review = major architectural decisions only, not every commit.
```

---

### 2.6 Completion Report Format

```
COMPLETION REPORT FORMAT — required from every Claude Code terminal:

TASK COMPLETE — [Task Name]
Files modified: [list each file + what changed]
Zero regressions: CONFIRMED / [list any issues]
Verification method: [exact steps performed]
Commit SHA: [sha]
Deployed SHA — status: [sha] — ok / pending / failed
Context estimate: LOW / MEDIUM / HIGH (informational only)
Surprises: [anything unexpected encountered]
Ready for next task: YES / NO — [reason if no]

Mission Control extended format also includes:
Zone (ZONE_OVERRIDE_RULE independent verify): [LOW/MEDIUM/HIGH]
Compliance: [list each rule and pass/fail]
Open items for coordinator: [numbered list]
```

---

### 2.7 Zone Classification System

```
ZONE_OVERRIDE_RULE:
A prompt's claimed zone is advisory only. The actual zone is determined
by what files and systems the task actually touches.

Claude Code must independently verify zone classification.
Hard-stop if actual zone is higher than prompt claimed.

Zone definitions (adapt per project):

HIGH (sign-off required before any edit):
- Revenue / billing / payment code
- Authentication and session management
- Database schema changes (migrations, ALTER TABLE, new columns)
- Customer PII or user data
- Production credentials or environment variables
- Any change touching 3+ systems simultaneously
- WorkoutContext.jsx (Fitness Tracker) — one terminal at a time only

MEDIUM (proceed autonomously, report at checkpoint):
- Business logic, route handlers
- Non-critical configuration
- Agent prompt text

LOW (fully autonomous):
- Documentation, comments, skill files
- Frontend CSS, copy, non-auth UI
- New non-auth API endpoints
- Tests, fixtures, log files
- Deploy health checks, browser visual verification

Hard-stop format:
"ZONE ESCALATION: This task touches [file/system] which is HIGH zone.
 Prompt claimed [LOW/MEDIUM]. Stopping for coordinator approval."

AUTONOMOUS_LOOP hard stops (always, regardless of zone):
- DB schema changes or migrations
- User data read/write
- Auth or session changes
- Architectural decisions touching 3+ systems
- Production environment variable changes
```

---

### 2.8 SPEC_FIRST_RULE

```
SPEC_FIRST_RULE:
Any medium+ complexity task or HIGH zone change requires a written
spec document approved by the coordinator BEFORE any code is written.

Spec doc requirements:
- Current behavior (what happens today)
- Gap (what is missing or broken)
- Proposed fix (what changes)
- Files touched (exact paths)
- Risk level and zone classification
- Dependencies on other changes

Spec lives in: docs/[feature]_spec_[session].md
Commit spec first, then implement in a separate commit.
```

---

### 2.9 FORWARD_FIX_RULE

```
FORWARD_FIX_RULE:
Never revert a commit. Fix forward.

If a deploy breaks something:
1. Diagnose root cause first
2. Write a targeted fix commit
3. Deploy the fix forward
4. Document what broke and why in the commit message

Exception: coordinator explicitly approves revert after reviewing
full impact of what would be lost.
```

---

### 2.10 LOOP_ORCHESTRATION_RULE

```
LOOP_ORCHESTRATION_RULE:
Repeating tasks with a clear definition of done are loop candidates.

Four-condition test:
1. Does it repeat?
2. Clear verifiable definition of done (pass/fail)?
3. Token cost acceptable for autonomous repetition?
4. All necessary tools available without human steps?

If yes to all four: build a loop skill in docs/skills/loops/

LOOP_TRAINING_MODE_RULE:
Every new loop runs in training mode for first 3 executions.
Checkpoint format before each run:
"LOOP CHECKPOINT [N/3]: About to [action].
 Estimated tokens so far: ~[N]. Confirm or abort?"

Training graduation requires:
- 3 validated runs (both PASS and correct FAIL count)
- Explicit coordinator sign-off
- Update loop skill doc to status: GRADUATED (active)

Never self-exit training mode.
```

---

### 2.11 SUB_AGENT_RULE

```
SUB_AGENT_RULE:
Independent parallel subtasks with no shared file dependencies
get their own scoped terminal session.

Rules:
- Maximum 5 parallel terminals at once
- Each sub-agent gets its own complete scoped prompt
- Sub-agents never share files with each other mid-task
- Each sub-agent independently verifies its own zone
- Sub-agents do not chain autonomously
- Coordinator assigns next task after completion report
```

---

### 2.12 Additional Rules

```
MODEL_RESERVATION_RULE:
Sonnet 4.6 = default for all work
Opus 4.8 = HIGH zone and architectural work only
Never put model directives inside Claude Code prompt bodies.
Model selection is coordinator-level guidance only.

CONTEXT_EFFICIENCY_RULE:
Read files once — never cat the same large file twice.
Use head -50 for discovery passes, not full file reads.
After 8+ tool calls in one task, assess whether remaining
work needs a fresh terminal.

QUESTION_BATCHING_RULE:
All clarifying questions go in one plain-text numbered list.
Never use interactive widgets or multi-turn question sequences.
If the answer can be found by reading the codebase, find it.
Maximum one round of questions before proceeding.

DEAD_CODE_CLEANUP_RULE:
Two triggers, whichever comes first:
1. Calendar: every 3-4 sessions
2. Significance: after any session touching 5+ files or
   adding a major new system

Process: T1 audit pass (ESLint no-unused-vars, unused imports,
TODO/FIXME/console.log grep, unread state/keys) → output finding
list before touching anything → Codex reviews findings and flags
false positives → Coordinator approves removal list →
Single cleanup commit (message: "chore: dead code cleanup").
Build and test must pass before push.
```

---

## SECTION 3 — SESSION START PROTOCOL

### Claude Chat Session Start Sequence

```
STEP 1: Confirm deployed SHA
Navigate to backend /openapi.json or health endpoint via Chrome MCP.
Compare deployed SHA to last session's final SHA.
If mismatch — investigate before any work begins.

STEP 2: Check both services are green
Railway dashboard — both frontend and backend services ACTIVE.

STEP 3: Check session bar
Open claude.ai/settings/usage (manually — Claude MCP cannot navigate
to claude.ai itself due to extension origin restriction).
Confirm current session % before assigning work.
If > 50% — plan accordingly, fewer parallel terminals.

STEP 4: Read priority queue
State what was completed last session (do not re-do).
State what is on deck for this session in priority order.

STEP 5: Assign first task
Single terminal only to start.
Expand to additional terminals only for parallel-safe independent tasks.
```

### Claude Code Terminal Session Start

Every terminal prompt must begin with:

```
git fetch origin && git rebase origin/main
git log --oneline -3
(Confirm HEAD matches expected SHA before touching anything.)
```

### Fitness Tracker Session Start Brief (paste at session open)

```
# FITNESS TRACKER — SESSION START BRIEF

## Tools Available to Claude Chat
- Chrome extension connected (Claude in Chrome)
- Google Drive connected
- Browser: call list_connected_browsers, then select laptop browser
- Backend schema: navigate to
  https://astonishing-laughter-production-de7d.up.railway.app/openapi.json

## Live URLs
Frontend:  https://fitness-tracker-production-54a4.up.railway.app
Backend:   https://astonishing-laughter-production-de7d.up.railway.app
GitHub:    https://github.com/patpeak1-eng/fitness-tracker
Railway:   peak-ops-q (877335d0-ecc2-4460-9800-291ffcb3f660)

## Architecture
- Frontend: React 18 + Vite PWA, served by Express server.js
- Backend: FastAPI + PostgreSQL on Railway (/backend root dir)
- Auth: Google OAuth, HttpOnly cookie (SameSite=None; Secure)
- WorkoutContext.jsx: central state, HIGHEST RISK — one terminal at a time only
- StorageService.js: localStorage layer
- ApiService.js: all backend API calls (credentials:include on every call)
- bcrypt==4.0.1 pinned — never upgrade
- anthropic>=0.40.0 — never downgrade

## Terminal Workflow
Every terminal prompt must include at top:
  git fetch origin && git rebase origin/main
Every terminal prompt must end with:
  git push origin HEAD:main
  Report SHA + files changed
Codex review required before every commit.
WorkoutContext.jsx = one terminal at a time only.
Never git add -A — always explicit pathspec.
```

---

## SECTION 4 — MANDATORY PROMPT BLOCKS

*These blocks must appear in every Claude Code terminal prompt, every project.*

### Standard Header Block

```
SESSION BUDGET RULE:
The session bar % at claude.ai/settings/usage → "Current session"
is the ONLY authoritative signal. Internal token math is informational
only — do NOT use it to mandate a terminal restart or stop work.

Fresh terminal = only when:
1. Coordinator instructs it, OR
2. Session bar exceeds 80% confirmed by coordinator, OR
3. Current task is fully complete and verified

Report context estimate in completion report as informational only.
```

### Zone Rule Block

```
Before touching any file, independently verify zone classification:
LOW = docs/config/style | MEDIUM = logic/routes | HIGH = revenue/auth/schema/user data
Hard-stop and report to coordinator if actual zone > prompt's claimed zone.
WorkoutContext.jsx (Fitness Tracker) = HIGH zone, one terminal at a time only.
```

### Git Ceremony Block

```
Before every commit:
1. git status --short — confirm only intended files dirty
2. Syntax check all modified files (npm run build or node --check)
3. Run /codex:review on all modified files
   If unavailable: inline self-review, document which method used
4. Output full diff as plain text in chat
5. Wait for coordinator "Codex cleared, proceed with commit"
6. git commit -m "type(scope): description" -- [explicit pathspec]
   NEVER git add -A or bare git commit without pathspec
7. git push origin HEAD:main (Fitness Tracker) or HEAD:master (Mission Control)
8. Railway dashboard = ground truth for deploy confirmation
```

### Completion Report Block

```
End every task with:
TASK COMPLETE — [Name]
Files modified: [list]
Zero regressions: CONFIRMED / [issues]
Verification method: [steps]
Commit SHA: [sha]
Deployed SHA — status: [sha] — ok/pending/failed
Context estimate: LOW/MEDIUM/HIGH (informational only)
Surprises: [anything unexpected]
Ready for next task: YES / NO — [reason if no]
```

### Clarifying Questions Block

```
If you have clarifying questions before proceeding, output them as
plain numbered text in the chat. Do not use pop-up dialogs or
interactive UI elements. Find answers in the codebase before asking.
```

---

## SECTION 5 — CHROME MCP PROTOCOL

### Session Start Connection

```
1. Call: Claude in Chrome:list_connected_browsers
2. Select laptop browser (device ID: 1b7e6193-7cf1-42c8-9e4b-3bcca9ffb054)
3. Call: Claude in Chrome:tabs_context_mcp (createIfEmpty: true)
4. Note current tab ID — changes every session, never hardcode
5. Navigate to backend /openapi.json to read current schema
   before writing any API code
```

### PWA Service Worker Clear (Before Every UI Verification)

```
After any deploy, before Chrome MCP UI verification:

1. Run this JavaScript in the connected tab:
   const regs = await navigator.serviceWorker.getRegistrations();
   for (let r of regs) await r.unregister();
   const keys = await caches.keys();
   for (let k of keys) await caches.delete(k);

2. Navigate to the app URL fresh
3. Then proceed with verification

Why: PWA service workers aggressively cache old bundles.
A fresh deploy may show old UI if the service worker is not cleared.
This is required before every live verification — never skip it.
```

### Dual-Verification Model

```
After every Claude Code completion report:
1. T1 self-verification (Claude Code ran it live, confirmed visually)
2. Chrome MCP coordinator check (Claude Chat independently navigates
   and confirms the same surface)

Both must pass before the task is marked complete.
Discrepancy between the two = investigate before closing.

Standard Chrome MCP checks:
- localStorage keys present and correct values
- Zero console errors (window.__errors or DevTools console)
- Page renders with expected content
- No emojis (Fitness Tracker: /[\u{1F300}-\u{1FFFF}]/gu test)
- lucide-react SVG icons rendering (not emoji fallbacks)
```

### Usage Bar Monitoring

```
Claude Chat cannot autonomously read claude.ai/settings/usage —
the extension blocks navigation to its own origin.

Protocol:
- Patrick monitors session bar at natural pause points
- Claude Chat asks for a check when T1 reports MEDIUM or HIGH
  context estimate, or when session has run 2+ hours
- Patrick reports: percentage and reset time
- Claude Chat decides: proceed, hold, or wrap session cleanly

If session bar > 80% and reset is within 30 minutes:
Hold — no new prompts sent. Resume after reset.
All work is safe — every iteration ends with a push to main.

If session bar > 80% and reset is more than 30 minutes:
Close session cleanly — T1 commits whatever is complete,
document stopping point, open new session with session brief.
```

### Known Limitations

```
- claude.ai cannot be navigated via Chrome MCP (own origin restriction)
- Login-gated pages inaccessible if session expired in Chrome
- Closing the MCP tab breaks the entire session connection
  Fix: click extension icon → reconnect, or reload extension
- After Chrome updates, tab IDs change — always call tabs_context_mcp fresh
- claude.ai intercepts navigation and redirects to chat interface
  Settings/usage page cannot be read programmatically
- Railway SPA fallback returns HTTP 200 for any path —
  curl health checks are unreliable for frontend deploy verification
  Use Railway dashboard ACTIVE card as ground truth instead
```

---

## SECTION 6 — LOOP AND ITERATION STRUCTURE

### Iteration Structure (Current Model)

```
One iteration = one logical unit of work:
- Clear verifiable start state
- Clear verifiable done state
- Self-contained (no dependency on previous iteration's uncommitted state)
- SHA-verified after any deploy it triggers
- Coordinator sign-off required before next iteration

Sequence:
1. Coordinator writes and sends iteration prompt to T1
2. T1 executes, self-verifies, produces completion report, stops
3. Patrick relays completion report to Claude Chat
4. Claude Chat reviews report, runs Chrome MCP verification
5. Claude Chat gives next instruction or flags issues
6. Repeat

T1 never chains autonomously into next iteration.
Exception: coordinator explicitly writes "on completion proceed to [X]"
Even then: completion report required at each step.
```

### Loop Training Mode

```
Every new loop runs in training mode for 3 executions.
Checkpoint required before each run — coordinator confirms or aborts.
A correct FAIL (catching a real problem) counts toward the 3 runs.
Never self-exit training mode regardless of results.
```

---

## SECTION 7 — SKILL CAPTURE SYSTEM

### When to Create a Skill

```
Create a skill when:
- A pattern was used successfully more than once
- The pattern involves multiple steps that could be forgotten
- The pattern is reusable across different tasks or projects
- A loop was graduated and needs permanent documentation

Do NOT create a skill when:
- It is a one-off specific to one task
- It can be described in one sentence in a commit message
- PONYTAIL rung 1 applies: does this skill need to exist?
```

### Skill File Format

```markdown
# [Skill Name]

## Trigger
When to use this skill. What conditions invoke it.

## Execution Skills
Dependencies and tools required.

## Goal + Verification
What success looks like. Verifiable pass/fail criteria.

## Output + Memory
What this skill produces. Where output is stored.

## Training-Mode Status
TRAINING (N/3) or GRADUATED (active)
```

### Skill Directory Structure

```
docs/skills/
  ├── loops/          # Loop orchestration skills
  │   ├── README.md
  │   └── [loop_name].md + [loop_name].sh
  ├── logs/           # Append-only run logs
  │   └── [log_name].md
  └── [skill_name].md # Individual execution skills
```

---

## SECTION 8 — NEW PROJECT BOOTSTRAP CHECKLIST

### A. One-Time Machine Setup (Do Once Ever)

```
[ ] Install Node.js LTS from nodejs.org
    Verify: node --version (v20+), npm --version (v10+)

[ ] Install Codex plugin via PowerShell
    Run /codex:setup in interactive session
    Authenticate: patpeak1@gmail.com
    Confirm review gate DISABLED
    Verify: /codex:review works in any Claude Code terminal

[ ] Create global ~/.claude/CLAUDE.md
    Paste Ponytail ruleset (Section 2.1)
    Verify: open fresh terminal, Ponytail active

[ ] Install Claude in Chrome extension
    Chrome Web Store → enable on all sites
    Note device ID in Claude settings → connected devices

[ ] Configure Git
    git config --global user.name "[name]"
    git config --global user.email "[email]"
```

### B. One-Time Project Setup (Do Once Per New Project)

```
[ ] Create repo and confirm branch name
    (Mission Control = master, Fitness Tracker = main — always confirm)

[ ] Create project CLAUDE.md at repo root
    Include: project stack, deploy method, standing rules from Section 2,
    project-specific zone definitions, any project-specific rules
    Include SESSION_BUDGET_RULE (corrected version)
    Include CODEX_REVIEW_RULE

[ ] Create docs/skills/ directory structure
    mkdir -p docs/skills/loops docs/skills/logs

[ ] Set up health check endpoint (backend projects)
    Must return: {"status":"ok","sha":"[7-char SHA]"}
    SHA read from environment variable — not git rev-parse

[ ] Create SESSION_START.md in repo root
    Reusable brief for starting each session

[ ] Set up Railway project and environment variables
    Never paste credentials in Claude Chat

[ ] Pin all critical dependencies immediately after first successful build
    Document pinned versions and reasons in requirements.txt / package.json comments
```

### C. Every Session Start (Every Session, Every Project)

```
[ ] Paste SESSION_START.md brief into Claude Chat
[ ] Claude Chat confirms deployed SHA matches last session's final SHA
[ ] Both services green on Railway dashboard
[ ] Chrome MCP connected — get current tab ID
[ ] Navigate Chrome to backend /openapi.json (read current schema)
[ ] Check session bar at claude.ai/settings/usage (manually)
[ ] State priority queue — completed last session, on deck this session
[ ] Open T1 with standard header prompt (single terminal to start)
[ ] Verify /codex:review available in terminal before any commit work
```

### D. Every Terminal Prompt (Required Elements)

```
1. SESSION BUDGET RULE block (Section 4)
2. Project repo path and branch (confirmed — never assumed)
3. Critical rules block (git ceremony, zone rules)
4. Task description with exact scope and file paths
5. CODEX REVIEW GATE block (before commit step)
6. COMPLETION REPORT FORMAT block
7. CLARIFYING QUESTIONS instruction
8. Confirmation that Codex review is required before committing
```

---

## SECTION 9 — KNOWN GOTCHAS AND LESSONS LEARNED

### Railway

```
DEPLOY VERIFICATION:
Railway dashboard ACTIVE card + correct SHA + "Deployment successful"
= ground truth. Never rely on curl for frontend (SPA 200 on every path).

SHA SOURCE:
Read from RAILWAY_GIT_COMMIT_SHA environment variable baked at build time.
Never from git rev-parse (no .git in container).
If SHA reads "unknown" — health endpoint misconfigured.

TRANSIENT ERROR ON POLL 2 IS NORMAL:
During container swap, /health may return error briefly.
Continue polling. Self-resolves by poll 3-4.
Only escalate if persists past poll 5.

INTERNAL DATABASE URL:
Railway DATABASE_URL uses postgres.railway.internal — only resolves
inside Railway's private network. Cannot query directly from local terminal.
```

### Git Safety

```
ALWAYS USE PATHSPEC:
git commit without pathspec bundles every dirty file.
In multi-terminal sessions, other terminals may have touched unrelated files.
Always: git commit -m "..." -- specific/files/only

BRANCH CONFUSION:
Mission Control = master. Fitness Tracker = main.
Never assume — always confirm with git branch.

GIT ADD -A IS BANNED:
Never use git add -A on any project. Explicit pathspec only.

ISOLATED WORKTREES:
Use isolated detached worktrees for every commit.
git add in shared checkout is unsafe — sibling terminals can sweep each others' staged files.
```

### PWA and Browser

```
SERVICE WORKER STALENESS:
Even hard reloads do not bypass Workbox precaching after deploys.
Correct sequence: unregister all service workers → delete all cache entries
→ navigate → reload. Do this before every live verification.

CLAUDE.AI INTERCEPTS CHROME MCP:
chrome.ai intercepts Chrome MCP navigation and redirects to chat interface.
Session bar must be read manually by Patrick and reported on request.
```

### Dependency Pins

```
bcrypt==4.0.1 pinned in Fitness Tracker backend/requirements.txt
Reason: passlib incompatibility with bcrypt 5.0+. Never upgrade.

anthropic>=0.40.0 — never downgrade (Fitness Tracker).

Alembic revision IDs must be under 32 characters (VARCHAR(32) column limit).
Example: "0004_uq_stats" not "0004_add_unique_constraint_user_stats"

After any successful build that resolves a dependency conflict:
pin the working versions immediately. Never leave versions floating
after a conflict resolution — the conflict will reappear.
```

### Agent Reliability

```
VERIFY THEN CONFIRM:
After any agent write operation, ask the agent to read back what it
just wrote before trusting the confirmation. If it cannot read it
back, the write did not land.

T1 HARD STOP PATTERN:
When T1 encounters something outside its scope or an ambiguity in the
spec, it outputs clarifying questions as plain numbered text and stops.
This is correct behavior — do not ask T1 to self-authorize.
Coordinator resolves, then sends "proceed" confirmation.
```

### Competitive Research Process (Established Fitness Tracker S9)

```
RESEARCH_BEFORE_ARCHITECTURE:
For any significant feature, run competitive research across multiple
AI platforms before writing a spec.

Assignment model:
- Gemini: [App 1] + [App 2]
- ChatGPT: [App 3] + [App 4]
- Codex Desktop: [App 5] + [App 6]
- Claude Code T1 (browser): user sentiment mining (App Store, Reddit)
- Claude Chat: synthesis → decision document → spec

No terminal opens until decision document is locked.
Research sprint = 30-60 minutes. Return = every major decision
for the feature answered with evidence, not guesswork.
```

---

## SECTION 10 — LIVING UPDATE PROCESS

### When a New Rule Is Developed

```
At session close, Claude Chat flags any new cross-project rules.
Patrick confirms which go in MASTER_CONTEXT.md.
First 5 minutes of the next session:
Update the standards repo and commit.

Commit message format:
"docs: add [rule name] from [project] S[number]"

This is not optional — it is part of the session close ritual.
```

### Where This Document Lives

```
RECOMMENDED: Dedicated GitHub repository
Name: peak-ops-standards
File: MASTER_CONTEXT.md
Access: raw GitHub URL — readable by any Claude Code terminal via curl

Why GitHub over Google Drive:
- Version history built in
- Claude Code terminals can read it directly via curl
- Works offline
- Raw URL accessible from any project's first prompt

Raw URL format:
https://raw.githubusercontent.com/patpeak1-eng/peak-ops-standards/main/MASTER_CONTEXT.md
```

### How New Projects Reference This Document

```
First Claude Chat message for any new project:

"Before anything else, read the master operating context at:
 [raw GitHub URL to MASTER_CONTEXT.md]

 Project context:
 PROJECT: [name]
 REPO: [path]
 BRANCH: [branch — confirmed]
 STACK: [stack]
 HEALTH CHECK: [endpoint or command]

 Session 1 goals:
 [3-5 specific items]

 Confirm machine-level setup is in place before writing any code."
```

### Update Cadence

```
Mandatory update triggers:
- Any new named rule developed in any project session
- Any new tool or plugin added to the stack
- Any gotcha discovered that cost time to learn
- Any correction to an existing rule (e.g. SESSION_BUDGET_RULE correction S83)

This document should never be more than one session out of date.
```

---

## APPENDIX — PROJECT-SPECIFIC REFERENCES

### Fitness Tracker Project

```
Frontend: https://fitness-tracker-production-54a4.up.railway.app
Backend:  https://astonishing-laughter-production-de7d.up.railway.app
GitHub:   patpeak1-eng/fitness-tracker
Railway:  peak-ops-q (877335d0-ecc2-4460-9800-291ffcb3f660)
Branch:   main

Stack: React 18 + Vite PWA | FastAPI + PostgreSQL | Railway
Auth: Google OAuth (HttpOnly cookie, SameSite=None; Secure)
AI: claude-sonnet-4-6 (AI Coach, SSE streaming via /api/coach/chat)
Voice: ElevenLabs (Jarvis voice FxZjRiAEBESrb7srpme7)

Pinned dependencies:
- bcrypt==4.0.1 (never upgrade — passlib incompatibility)
- anthropic>=0.40.0 (never downgrade)
- Alembic revision IDs max 32 chars

No emojis anywhere in the app — use lucide-react icons or nothing.
WorkoutContext.jsx = highest risk file, one terminal at a time only.

Session 9 state (last completed session):
Commits: 232fc4f, 620ae0d, c417171, 8abea2a, e969a15
Equipment profile system fully built and deployed.
Template picker live with equipment filtering and chip filters.
Fire Station pause button working with reload persistence.
Dead code cleanup scheduled: Session 10 (significance threshold met).
Codex plugin install: PENDING (Patrick runs PowerShell setup at Session 10 start).
```

### Mission Control Project

```
Branch: master (not main)
Stack: Python/FastAPI backend + static HTML frontend
Deploy: GitHub webhook → Railway (never railway up)
GitNexus: installed, gitnexus_rename permanently waived (see 2.12)
DUAL_PLATFORM_RULE: Claude Code owns backend Python, Codex Desktop owns frontend HTML
```

---

*Last updated: Session 9 (Fitness Tracker) / S83 (Mission Control) | 2026-06-29*
*Next update due: Session 10 (Fitness Tracker) — after Codex install and cleanup pass*
