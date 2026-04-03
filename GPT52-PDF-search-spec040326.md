Artistic Agentic Flow System — Updated Technical Specification (Streamlit / Hugging Face Spaces)
Version: 2.0.0
Date: 2026-04-03
Status: Proposed (Design Update, No Code)
Author: Senior Frontend Engineer / Architect

1. Executive Summary
1.1 Product Overview
Artistic Agentic Flow is a Streamlit-based, agentic AI workflow application deployed on Hugging Face Spaces. It combines multi-provider LLM orchestration (OpenAI, Google Gemini, Anthropic, Grok) with an artistic, themeable “WOW UI,” an interactive operational dashboard, editable step-by-step agent execution, and advanced note/document transformation tools.

This update preserves all original capabilities (agent chaining, configurable pipelines, real-time logs, multilingual UI, theme switching, and “painter styles”) and adds major new capabilities:

WOW UI Revamp

Light/Dark themes
English / Traditional Chinese localization
20 painter-inspired styles selectable via a Jackpot (“Jackslot”) randomizer
WOW Status Indicators + Interactive Dashboard

Visual health metrics for providers, keys, rate limits, token usage
Job-level progress tracking and agent-level execution state
Rich timeline logs and artifact tracking (inputs/outputs/files)
Secure API Key Entry

Auto-detect keys from environment (HF Secrets)
If missing, allow manual entry in UI (masked), stored only in session
If present in environment, do not display the key
Editable, Step-by-Step Agent Execution

Select model and modify prompts per agent before running
Run agents one-by-one
Edit each agent output (text/markdown) to feed into the next agent
AI Note Keeper (Major Expansion)

Paste text → transform into Markdown
User-editable in text/markdown view
Tools: AI Formatting, AI Keywords highlighting, AI Entities table (20 entities), AI Chat, AI Summary, plus two new “AI Magics”
Directory Scanner + PDF OCR to Markdown

User provides a directory path (with deployment constraints and supported modes described below)
Recursive scan (excluding hidden folders/files, .git, node_modules, etc.)
Export directory.md (editable + downloadable)
For PDFs: OCR first N pages (default 3, user-configurable) using Tesseract with selectable languages (English, Traditional Chinese, plus two additional choices)
Generate per-PDF Markdown artifacts (editable + downloadable), named from the original file + OCR summary
Skill-Based Document Execution

User defines a “Skill” description
User uploads/pastes a doc (txt/markdown/pdf)
Agent executes the skill against the doc (model selectable)
Prompt/skill definitions persist and remain editable
Adds 3 additional WOW AI features (defined in this spec)
2. Deployment & Execution Model
2.1 Primary Deployment: Hugging Face Spaces (Streamlit)
Runs in a containerized environment with restricted file system access.
Reads API keys from HF Secrets (environment variables).
Supports uploading files via Streamlit file uploader.
Supports saving artifacts to a workspace directory in the container session (ephemeral unless explicitly persisted using HF storage patterns).
2.2 Local-Path Directory Scanning: Feasibility & Supported Modes
A web app running on HF Spaces cannot directly access a user’s local machine directory path due to browser sandboxing and remote execution. To keep the requested feature while remaining technically correct, the system supports two operating modes:

Mode A — Local Deployment (Full Support)

When user runs the Streamlit app locally (or on a machine that has access to the target path), “paste directory path” works exactly as requested.
The Python scanner and OCR tools operate on local file system paths.
Mode B — Hugging Face Spaces (Path Substitution)

“Directory Path” input is supported but interpreted as a path inside the container.
To achieve user intent (“scan my local directory”), provide a guided alternative:
Upload a .zip of the directory (recommended), or
Upload a manifest, or
Upload selected files/folders using multi-file uploader
The system then scans the uploaded/extracted workspace directory.
This preserves the “directory scan” feature while making it practical on Spaces.

3. Technology Stack (Updated)
3.1 Core Runtime
Python 3.11+
Streamlit as the UI framework
agents.yaml as the canonical agent/pipeline configuration source
3.2 AI Providers
OpenAI API (models: gpt-4o-mini, gpt-4.1-mini)
Google Gemini API (models: gemini-2.5-flash, gemini-2.5-flash-lite, gemini-3-flash-preview)
Anthropic API (multiple Anthropic models; model list is configurable in settings)
Grok API (models: grok-4-fast-reasoning, grok-3-mini)
3.3 Document Processing
Tesseract OCR (with language packs)
PDF page extraction library (implementation detail; must support first N pages extraction reliably)
Markdown generation utilities
3.4 Data & State
Streamlit session_state for:
Theme/language selection
API key session overrides (masked input)
Current agent chain state (inputs/outputs)
Note Keeper content and tool outputs
Dashboard telemetry snapshots
Optional local persistence (when allowed):
Exported Markdown files (directory.md, per-PDF .md, notes exports)
Audit log exports
4. Information Architecture (Navigation)
The app uses a left navigation rail with consistent “WOW” status chips and context actions.

Dashboard (WOW Control Center)
Agents (Step-by-step Execution Studio)
Pipeline (Batch / Multi-agent Flow Runner)
Brain (Interactive Chat / Reasoning Playground)
AI Note Keeper (Transform + Enrich Notes)
Directory & OCR (Scan → Markdown + OCR Artifacts)
Skill Runner (Skill-on-Doc Execution)
Officer Tools (Compliance / Review Workflows)
Settings (Keys, Providers, UI, Language, Export)
5. WOW UI System
5.1 Global Toggles (Always Available)
Theme Mode: Light / Dark
Language: English / Traditional Chinese (繁體中文)
Painter Style: 20 curated styles
Jackslot: One-click random style selection with an animation (non-blocking)
5.2 The 20 Painter-Inspired Styles (Design Tokens)
Each style defines a token pack:

Primary/secondary/accent colors
Background gradients
Typography pairing
Button/hover behavior
Card borders, shadows, noise texture
Chart palette
Log/event colors
Style list (20):

Monet (Impressionist Mist)
Van Gogh (Impasto Night)
Picasso (Cubist Blocks)
Dali (Surreal Melt)
Klimt (Gilded Ornament)
Hokusai (Ukiyo-e Wave)
Frida Kahlo (Folk Vivid)
Rothko (Color Field)
Pollock (Action Splatter)
Vermeer (Candle Contrast)
Caravaggio (Chiaroscuro Noir)
Turner (Luminous Storm)
Matisse (Cutout Bold)
Rembrandt (Warm Depth)
Magritte (Clean Paradox)
Warhol (Pop Neon)
Basquiat (Neo-Expression)
Georgia O’Keeffe (Desert Bloom)
Edward Hopper (Quiet Urban)
Cyber-Brush (Modern Synthwave “AI Painter” hybrid)
5.3 Accessibility & Usability Requirements
Minimum contrast ratios enforced per theme pack
Keyboard navigability across editors and buttons
Clear focus states in all styles
Reduced motion mode (respects OS preference)
Language toggle instantly updates all UI strings and system messages (non-destructive)
6. WOW Status Indicators (Global + Contextual)
6.1 Global Status Bar (Header)
Always visible indicators:

Provider Health: OpenAI / Gemini / Anthropic / Grok (green/yellow/red)
Key Source: Environment vs Session override (icon only; no key display)
Rate Limit Awareness: last error + cooldown timer if applicable
Token Meter: session estimate (input/output tokens per provider)
Cost Meter: approximate cost estimate (configurable pricing table)
Job Queue: number of active/pending agent runs
6.2 Agent-Level Status (Per Step)
For each agent step card:

State: Idle / Configured / Running / Succeeded / Failed / Skipped
Latency, tokens in/out, retry count
“Diff since last run” indicator when prompt/model/input changed
Artifact links: input snapshot, raw output, cleaned output, markdown render
6.3 Dashboard Telemetry (Aggregated)
Execution timeline graph (events: started, completed, failed)
Provider distribution chart (calls per provider/model)
Error panel with “most common causes” classification:
Auth missing/invalid
Rate-limited
Context too large
PDF/OCR failure
YAML validation error
7. Interactive Dashboard (“WOW Control Center”)
7.1 Dashboard Layout
Top Row: KPIs (runs today, success rate, avg latency, tokens, cost)
Second Row: Interactive charts (provider usage, model usage, error types)
Main Panel: Real-time event stream (filterable, searchable)
Right Panel: Current session configuration summary:
selected theme/style/language
active providers
key source status
current pipeline loaded from agents.yaml
7.2 Log & Artifact System
Every run produces:
Input snapshot (text + metadata)
Prompt used (final resolved prompt)
Model selected
Output raw
Output edited (if user changed it)
Output rendered (markdown preview)
Logs are structured, filterable, exportable to .json and .md.
8. Agent Configuration & Execution (Core Upgrade)
8.1 agents.yaml as Source of Truth
The system loads agents.yaml describing:

Agent name, description
Default prompt template
Default model preference list (per provider)
Expected output type (text/markdown/json-ish)
Validation rules (optional)
Post-process steps (optional; e.g., “ensure markdown headings”)
8.2 Pre-Execution Controls (Per Agent)
Before running each agent, user can:

Modify the prompt template (with variable insertion helpers)
Select a model from the allowed list:
gpt-4o-mini, gpt-4.1-mini
gemini-2.5-flash, gemini-3-flash-preview, gemini-2.5-flash-lite
Anthropic models (configurable list)
grok-4-fast-reasoning, grok-3-mini
Select output view preference:
Text
Markdown (rendered preview + source)
8.3 Step-by-Step Execution & Editable Handoff
The agent chain runs one agent at a time.
After each agent:
Output is presented in split view:
Left: editable source (text or markdown source)
Right: rendered preview (if markdown)
User can edit the output and explicitly “Approve as Next Input”
The next agent receives:
Approved text
Metadata (agent name, model, timestamps, tokens, edits applied)
8.4 Error Handling & Recovery
On failure:
Show provider error category + raw message
Provide quick actions:
Retry (same settings)
Switch model
Reduce context (auto-summarize previous outputs)
Continue with last successful output (manual override)
9. API Key Management (Secure UX)
9.1 Key Sources
Environment keys (HF Secrets): preferred
Session keys (UI input): fallback if env is missing
9.2 UI Behavior Rules
If key exists in environment:
Show status: “Configured via Environment”
Do not show the key or reveal length
Do not allow “copy” or “reveal”
If key missing:
Show masked password input
Allow saving in session_state only
Provide “Clear session key” button
9.3 Security Constraints
Keys never written to logs, markdown exports, or artifacts
Any error messages must redact tokens/keys automatically
If user exports diagnostics, keys are excluded
10. AI Note Keeper (Expanded)
10.1 Core Flow
User pastes text (raw notes)
System transforms into Markdown
User edits in:
Plain text editor
Markdown source editor
Markdown preview
10.2 Tools (Existing + New)
A) AI Formatting
Converts rough notes into structured Markdown:
headings, lists, tables, callouts
consistent punctuation and spacing
Options:
“Minimal formatting”
“Meeting notes”
“Technical spec”
“Study notes”
B) AI Keywords (User-Defined Highlighting)
User enters keywords (comma-separated or multi-line)
User selects a color per keyword group
System outputs Markdown with:
inline highlights using consistent syntax (implementation-defined)
a keyword legend table at top
Includes a “keyword density” summary section
C) AI Entities (20 Entities with Context)
Extract exactly 20 entities from the note:
People, orgs, products, acronyms, concepts, locations, dates
Output as a Markdown table with columns:
Entity
Type
Context snippet (quoted)
Why it matters
Suggested follow-up question
D) AI Chat (Note-Scoped)
Chat uses the current note as context
User chooses model and prompt
Chat history can be exported to Markdown
“Cite from note” option enforces quoting
E) AI Summary
User can modify summary prompt and select model
Summary modes:
TL;DR
Executive summary
Action items
Risks / unknowns
Summary output is editable and can be inserted back into the note
F) AI Magics (2 New Features)
AI Action Forge

Turns notes into:
task list (owner, due date, priority)
decision log
open questions list
Outputs as Markdown sections and tables
AI Consistency Guardian

Finds contradictions, ambiguous pronouns, missing definitions, inconsistent numbering
Produces a “Fix List” with:
issue
location (quote)
suggested rewrite
Offers a “apply rewrite patches” mode (user-approved)
11. Directory Scanner + PDF OCR → Markdown
11.1 Directory Scan Feature (Markdown Export)
Goal: Convert a directory tree into an editable Markdown inventory.

Input:

Directory path (local path in Local Mode, container path / uploaded workspace in HF Mode)
Exclusions (must enforce):

Hidden files/folders (prefix .)
.git/, node_modules/, __pycache__/
Build artifacts: dist/, build/, .venv/ (configurable)
Large binary types (optional ignore list; configurable)
Output:

directory.md containing:
Scanned root path (display-only)
Timestamp
Directory tree (indented Markdown list)
Summary table:
counts by file extension
total files scanned
total folders scanned
“Not scanned” section listing excluded paths and reasons
User actions:

Edit directory.md in text/markdown view
Download directory.md
11.2 PDF OCR Pipeline (First N Pages)
Trigger: After scan, user selects PDF files (or all PDFs) for OCR.

Configuration:

Page count N (default 3; user adjustable)
OCR language choices (select one or multiple):
English (eng)
Traditional Chinese (chi_tra)
Additional choice #1: Simplified Chinese (chi_sim)
Additional choice #2: Japanese (jpn) (or configurable alternative)
OCR mode:
“Speed” (faster, lower accuracy)
“Accuracy” (slower, better layout heuristics)
OCR Output Per PDF:

Markdown file name:
<original_filename>_ocr_summary.md
Content structure:
Document header (file name, pages processed, OCR languages, timestamp)
Extracted text per page (with ## Page 1, etc.)
“OCR Confidence Notes” (if available)
Short AI-generated summary (optional, model selectable)
“Potential Issues” (e.g., low confidence sections, missing glyphs)
User actions:

View + edit each generated Markdown
Download individual .md or download all as a zip bundle (export behavior requirement)
Failure handling:

If OCR fails, output an .md stub with error description and remediation steps (e.g., “language pack missing”)
12. Skill Runner (Skill-on-Doc Execution)
12.1 Core Concept
A Skill is a reusable instruction set that can be applied to a document. Users provide:

A Skill description (e.g., “Extract compliance risks and propose mitigations in a table.”)
An optional Skill prompt template (advanced mode)
A target document (paste or upload: .txt, .md, .pdf)
The system then runs an agent using the Skill against the provided document with a user-selected model.

12.2 Document Input Support
Text input (paste)
Upload:
.txt / .md → direct read
.pdf → extract text; optionally run OCR if the PDF is scanned
Preprocessing options:
Strip headers/footers
Normalize whitespace
Keep page markers
12.3 Skill Persistence & Editing
Skill is saved in session and optionally exportable as:
skill.md (human-readable)
skill.json (structured fields; optional)
Users can:
Duplicate skill
Version skill (“v1”, “v2”, etc.)
Compare skill versions (diff view)
12.4 Execution Output
Output supports:
text view
markdown view with preview
Output can be:
inserted into Note Keeper
attached as an artifact to dashboard logs
fed into Agents chain as starting input
12.5 Three Additional WOW AI Features (New)
Skill Composer (Auto-Structuring)

User writes a rough skill idea; system refines it into a structured template:
objective
constraints
output format schema
quality checklist
Produces a “Skill Card” in Markdown for easy reuse.
Skill QA Harness (Self-Check & Scoring)

After producing output, system runs an evaluator pass (user-selectable model) that scores:
completeness
factual grounding (with quotes from doc)
formatting adherence
missing sections
Produces a “Quality Report” with suggested revisions.
Skill Multi-Pass Transformer

Applies the same skill in staged passes:
Pass 1: extract key facts
Pass 2: synthesize
Pass 3: format output precisely
User can edit intermediate pass outputs (similar to agent step handoff).
13. Pipeline, Brain, Officer Tools (Compatibility)
13.1 Pipeline Runner (Batch/Flow)
Still supports multi-step pipelines from agents.yaml
Adds:
per-step model selector
per-step prompt editor
per-step editable output gating (optional toggle; can be strict/manual or automatic)
13.2 Brain (Chat Playground)
Multi-provider chat with:
system prompt editor
model selection
optional injection of:
last agent output
selected note
selected OCR markdown
Chat exports to markdown
13.3 Officer Tools (Compliance / Review)
Uses the same agent execution engine
Adds structured checklists and standardized report exports
Supports bilingual output (EN / 繁中)
14. Data Flow & Artifacts
14.1 Artifact Types
Agent run artifacts: input, prompt, model, output raw/edited, rendering
Note artifacts: original paste, transformed markdown, tool outputs
Directory scan artifacts: directory.md
OCR artifacts: per-PDF markdown outputs
Skill artifacts: skill versions, outputs, QA reports
14.2 Export & Download
Every artifact is downloadable individually
“Export Session Bundle” downloads:
all .md artifacts
logs in .json and optionally a markdown report
15. Non-Functional Requirements
15.1 Performance
Streamlit UI must remain responsive during long operations:
OCR and large-doc processing must show progress indicators
Use background task pattern (implementation-defined) with periodic UI updates
15.2 Reliability
Retries with exponential backoff for provider calls
Deterministic snapshotting:
each run stores the exact prompt and inputs used
Graceful degradation:
missing provider key disables that provider in model lists
15.3 Privacy & Security
Keys never logged or exported
Redaction of secrets from errors
Clear user messaging on HF-mode filesystem constraints
15.4 Internationalization (i18n)
All UI labels, tooltips, and system notifications available in:
English
Traditional Chinese
User-generated content is not auto-translated unless explicitly requested
16. Model Selection Policy (Unified UX)
16.1 Model Registry
A centralized model registry defines:

provider
model id
capabilities tags (fast, reasoning, cheap, long-context)
max input guidance (soft limit)
recommended use cases
16.2 User Experience Requirements
Default recommended model shown per tool
Advanced dropdown lists all supported models
Provider disabled states explained (e.g., “Key not configured”)
17. Acceptance Criteria (High-Level)
User can switch Light/Dark, EN/繁中, and 20 painter styles, and use Jackslot to randomize style.
Dashboard displays provider health, job timeline, tokens/cost estimates, and structured logs.
API keys:
Use environment keys if present without displaying them
Allow masked UI input only when missing
Agents:
User can modify prompt and select model per step
Run one-by-one
Edit outputs and pass to next step
Note Keeper:
Paste → Markdown transform
Editable text/markdown views
AI Formatting, AI Keywords + color, AI Entities table (20), AI Chat, AI Summary
Two AI Magics: Action Forge, Consistency Guardian
Directory scan:
Excludes hidden files/folders and common ignored dirs
Generates editable/downloadable directory.md
PDF OCR:
User chooses pages (default 3)
Tesseract languages include English, Traditional Chinese, and two more
Produces editable/downloadable per-PDF Markdown outputs
Skill Runner:
Skill description + doc upload/paste
Model selection
Persistent editable skill
Includes Skill Composer, Skill QA Harness, Skill Multi-Pass Transformer
Works on Hugging Face Spaces with clear limitation messaging for local directory paths and a supported upload-based workaround.
