# Artistic Agentic Flow System — Updated Technical Specification (Streamlit / Hugging Face Spaces)  
**Version:** 2.1.0 (Design Update)  
**Date:** 2026-04-03  
**Status:** Proposed — *No code, specification only*  
**Deployment:** Hugging Face Spaces (Streamlit) + optional Local Mode  
**Core Config:** `agents.yaml` + multi-provider LLM APIs (Gemini / OpenAI / Anthropic / Grok)

---

## 1. Executive Summary

Artistic Agentic Flow is a themeable, multilingual, multi-agent AI workflow application deployed on Hugging Face Spaces using Streamlit. It orchestrates agent pipelines defined in `agents.yaml`, supports step-by-step execution with user-controlled prompts and model selection, and provides rich artifact editing/hand-off between agents.

This update **keeps all original features** (WOW UI themes, English/Traditional Chinese UI, 20 painter styles with Jackslot, interactive dashboard + status indicators, secure API key management, editable step-by-step agents, AI Note Keeper suite, directory scanning + PDF OCR to Markdown, skill runner + skill tooling) and adds a new end-to-end workflow:

### New Module: Medical Device Regulation Intelligence (MDRI)
Users can paste medical device regulatory information (news/guidance/regulation/standards), optionally paste or choose a default report template, select output language (English / Traditional Chinese), then run a **3-step agent chain** (each step editable for prompt + model; each output editable and downloadable):

1. **Web-researched comprehensive summary** (Markdown, **2000–3000 words**) with citations and evidence mapping  
2. **Comprehensive regulation report** (Markdown, **3000–4000 words**) aligned to the chosen template  
3. **`skill.md` generation**: creates an agent skill description enabling future generation of similar reports for similar inputs, explicitly aligned with the provided *skill creator skill* behavior

All three outputs are editable (text/markdown views), versioned as artifacts, and downloadable.

---

## 2. Goals, Non-Goals, and Product Principles

### 2.1 Goals
- Provide a **WOW-grade UI** that is visually distinctive yet operationally robust (dashboard, statuses, logs, artifacts).
- Enable **human-in-the-loop agent chains**: per-step prompt/model control, and editable outputs passed to subsequent agents.
- Support **secure multi-provider API key handling**: environment-first; UI key input only when missing; never reveal env keys.
- Provide **document transformation and enrichment** via AI Note Keeper, OCR, and skill-based execution.
- Add **medical device regulation summarization and report generation** with web research, citations, and template-driven report structure.

### 2.2 Non-Goals (Explicit)
- The system is **not** a regulated medical/legal advice system. It produces research and drafts for review.
- No guarantee of completeness of web search in restricted environments; must transparently show evidence and limitations.
- No automatic submission to regulators or generation of “final compliance decisions” without human review.

### 2.3 Product Principles
- **Provenance-first:** every generated claim should be traceable to user input or cited sources where feasible.
- **Editable-by-default:** outputs are drafts; users can modify before proceeding.
- **Provider-agnostic orchestration:** consistent UX across OpenAI/Gemini/Anthropic/Grok.
- **Bilingual professionalism:** UI and outputs can be produced in English or Traditional Chinese with consistent terminology.

---

## 3. System Overview & Modules (Information Architecture)

Left navigation rail (persistent) with global WOW status chips and context actions:

1. **WOW Control Center (Dashboard)**
2. **Agents Studio (Step-by-Step Execution)**
3. **Pipeline Runner (Batch Flow from `agents.yaml`)**
4. **Brain (Chat Playground / Reasoning)**
5. **AI Note Keeper**
6. **Directory & OCR → Markdown**
7. **Skill Runner (Skill-on-Doc Execution)**
8. **Medical Device Regulation Intelligence (NEW)**
9. **Officer Tools (Compliance Review Utilities)**
10. **Settings (Keys / Providers / UI / Export)**

---

## 4. Deployment Constraints & Supported Modes

### 4.1 Hugging Face Spaces Mode (Primary)
- Runs remotely; cannot access the user’s local file paths directly.
- Supports:
  - Text paste
  - File uploads (txt/md/pdf)
  - Optional upload of zipped directories for scan/OCR workflows
- Artifacts persist for the session; long-term persistence depends on Space storage strategy.

### 4.2 Local Mode (Optional)
- When run locally (or in an environment with file system access), directory scanning works with real local paths.
- OCR and scan can operate on local directories without uploads.

---

## 5. WOW UI System (Preserved + Strengthened)

### 5.1 Global Toggles (Always Visible)
- Theme: **Light / Dark**
- UI Language: **English / Traditional Chinese (繁體中文)**
- Painter Style: **20 curated style packs**
- **Jackslot**: one-click random style selection (animated, non-blocking)

### 5.2 Painter Style Packs (20)
Each style defines design tokens (colors, typography, card chrome, chart palettes, log severity colors). Preserved list:

1. Monet, 2. Van Gogh, 3. Picasso, 4. Dali, 5. Klimt, 6. Hokusai, 7. Frida Kahlo, 8. Rothko, 9. Pollock, 10. Vermeer, 11. Caravaggio, 12. Turner, 13. Matisse, 14. Rembrandt, 15. Magritte, 16. Warhol, 17. Basquiat, 18. Georgia O’Keeffe, 19. Edward Hopper, 20. Cyber-Brush

### 5.3 Accessibility
- Contrast compliance per theme
- Keyboard navigation across editors/buttons
- Reduced-motion support
- Language toggle updates UI strings instantly without resetting user content

---

## 6. WOW Status Indicators & Interactive Dashboard (Preserved)

### 6.1 Global Header Status Bar
- Provider Health: OpenAI / Gemini / Anthropic / Grok (green/yellow/red)
- Key Source: Environment vs Session override (icon only; no key text)
- Rate limit & cooldown indicator
- Token/cost estimates per provider
- Active/pending job count

### 6.2 Agent Step Status Cards
- Idle / Configured / Running / Succeeded / Failed / Skipped
- Latency, tokens, retry count
- “Diff since last run” when prompt/model/input changed
- Artifact links: input snapshot, raw output, approved output, rendered markdown

### 6.3 Dashboard Telemetry
- Timeline of events
- Model/provider usage charts
- Error classification panel (auth/rate/context/OCR/YAML/etc.)
- Exportable session diagnostics (with key redaction)

---

## 7. API Key Management (Preserved Requirements)

### 7.1 Key Sources (Priority)
1. **Environment (HF Secrets)** — preferred  
2. **Session UI input** — fallback only if missing

### 7.2 UX Rules
- If env key exists:
  - Show “Configured via Environment”
  - **Never display** the key (no reveal/copy, no length hints)
- If missing:
  - Show masked input
  - Save to session only
  - “Clear session key” button
- Keys never appear in logs, exports, artifacts, or error traces (must redact).

---

## 8. Agent Execution System (Preserved + Used by MDRI)

### 8.1 `agents.yaml` as Canonical Configuration
Each agent definition includes:
- Name, description
- Default prompt template
- Default model preference list
- Expected output type (text/markdown)
- Optional validations (structure/length/citations)
- Optional post-processing rules (e.g., “ensure Markdown headings”)

### 8.2 Step-by-Step Studio (Human-in-the-loop)
For each agent:
- User can **modify prompt**
- User can **select model** from:
  - `gpt-4o-mini`, `gpt-4.1-mini`
  - `gemini-2.5-flash`, `gemini-3-flash-preview`, `gemini-2.5-flash-lite`
  - Anthropic models (configurable list)
  - `grok-4-fast-reasoning`, `grok-3-mini`
- User chooses output view:
  - Text
  - Markdown source + rendered preview

### 8.3 Editable Handoff
After each agent run:
- Output shown in split view:
  - Editable source (text/markdown)
  - Rendered preview (if markdown)
- User clicks **“Approve as Next Input”**
- Next agent receives:
  - approved output
  - metadata (agent name, model, timestamps, token stats, edit history)

### 8.4 Failure Recovery
- Retry
- Switch model
- Reduce context (auto-summarize prior steps)
- Continue with last approved output

---

## 9. AI Note Keeper (Preserved)

### 9.1 Base Flow
Paste text → transform to Markdown → user edits in:
- Text editor
- Markdown source editor
- Markdown preview

### 9.2 Tools (Preserved)
- AI Formatting
- AI Keywords (user-defined keywords + color selection + legend + density)
- AI Entities (exactly 20 entities with context) in a Markdown table
- AI Chat (note-scoped; model selectable; exportable)
- AI Summary (prompt + model selectable; editable insertion)
- AI Magics:
  - Action Forge (tasks/decision log/open questions tables)
  - Consistency Guardian (ambiguities/contradictions + suggested rewrites)

---

## 10. Directory Scanner + PDF OCR → Markdown (Preserved)

### 10.1 Directory Scan → `directory.md`
- Supports local path (Local Mode) or container workspace (Spaces Mode)
- Excludes hidden and common ignored dirs
- Outputs directory tree + extension summary + exclusions list
- Editable + downloadable

### 10.2 PDF OCR
- OCR first N pages (default 3)
- Tesseract languages:
  - English (eng)
  - Traditional Chinese (chi_tra)
  - plus configurable extras (e.g., chi_sim, jpn)
- Per-PDF output: `<filename>_ocr_summary.md` (editable + downloadable)
- Robust failure stubs with remediation hints

---

## 11. Skill Runner (Preserved)

### 11.1 Skill-on-Doc Execution
- User provides skill description/template + a document (paste/upload)
- Model selectable
- Output editable, exportable, can feed into agent chain or note keeper

### 11.2 Additional WOW Skill Features (Preserved)
- Skill Composer (structure a rough skill into a reusable format)
- Skill QA Harness (self-check scoring: completeness, grounding, format)
- Skill Multi-Pass Transformer (fact extraction → synthesis → formatting; editable between passes)

---

# 12. NEW MODULE — Medical Device Regulation Intelligence (MDRI)

## 12.1 User Story (Primary)
A user pastes medical device regulation news or guidance/standards/regulation text (txt/markdown). They optionally paste a report template or use the default template. They choose output language (English / Traditional Chinese). Then, agent-by-agent (with editable prompt and model selection), the system:
1) searches the web for related regulatory context and produces a comprehensive summary (2000–3000 words, markdown, editable + downloadable)  
2) produces a comprehensive regulation report using the chosen template (3000–4000 words, markdown, editable + downloadable)  
3) produces `skill.md` describing a reusable skill for creating similar reports for similar inputs, explicitly aligned to the provided *skill creator skill* intent (editable + downloadable)

## 12.2 MDRI Navigation & Layout
Within MDRI, a 4-pane “mission control” layout:

1. **Inputs Panel**
   - Paste input text (news/guidance/standard/regulation)
   - Optional: paste template (or select default)
   - Output language selector (EN / 繁中)
   - Jurisdiction hints (optional tags): US FDA, EU MDR/IVDR, UK MHRA, IMDRF, ISO/IEC, Taiwan TFDA, Japan PMDA, etc.
   - Time window for web research (e.g., last 12 months / 24 months / all)
2. **Agent Chain Panel (3 steps)**
   - Each step card: prompt editor, model selector, run button
3. **Artifact Editor Panel**
   - Text/Markdown source editor + preview
   - Diff view between runs
4. **Evidence & Citations Panel**
   - Source list with URLs, titles, access time
   - Extracted quotes/snippets used
   - Coverage checklist and “missing evidence” warnings

## 12.3 Inputs: Supported Formats & Preprocessing
### 12.3.1 Accepted Input
- Paste: plain text or Markdown
- Upload: `.txt`, `.md`, `.pdf` (PDF extraction; optional OCR if scanned)

### 12.3.2 Normalization
- Preserve original text as **Input Artifact v0**
- Create a **Normalized Input Artifact v1**:
  - normalize whitespace
  - preserve headings if markdown
  - optionally detect jurisdiction keywords (non-destructive metadata)

### 12.3.3 Safety/Compliance Notice (UI)
- Always display: “Draft for informational purposes; verify against official sources.”

---

## 12.4 Template Handling (Default + User-Provided)

### 12.4.1 Default Template (Provided by User)
The default template is stored as a selectable template named:

**「美國FDA醫療器材網路安全法規」**  
(Uses the content provided in the prompt, including: overview, nine digital health topics, cybersecurity guidance details, key focus areas, summary, references.)

### 12.4.2 User-Provided Template
Users may paste a template in txt/markdown.

**Template parsing requirements:**
- Preserve user headings and section order
- Detect required sections (if headings exist)
- If template is unstructured, agent must:
  - propose a structured outline
  - ask user to approve or auto-apply (configurable)

### 12.4.3 Template Variables (Conceptual)
Even without code, the system standardizes placeholders conceptually:
- `{INPUT_SUMMARY}`
- `{WEB_RESEARCH_FINDINGS}`
- `{REGULATORY_REQUIREMENTS}`
- `{IMPACT_ANALYSIS}`
- `{RECOMMENDATIONS}`
- `{REFERENCES}`

If placeholders are absent, the report generator agent aligns content to sections by semantic matching.

---

## 12.5 Web Research (Search) Capability — Design Requirements

### 12.5.1 Research Connector Abstraction
MDRI requires a **Web Research Connector** that can operate in constrained environments:

- **Primary approach:** curated-source retrieval
  - user-specified URLs (optional)
  - predefined authoritative domains list (configurable allowlist)
- **Secondary approach:** search API integration (optional)
  - If available, use a configured search service (requires separate key)
- **Fallback approach:** “No-search mode”
  - the system produces a summary strictly from user input and warns that web research is unavailable.

### 12.5.2 Authoritative Domain Allowlist (Default Suggestions)
- FDA (fda.gov), Federal Register (federalregister.gov)
- EU (eur-lex.europa.eu), European Commission pages
- ISO/IEC (if accessible), IMDRF
- MHRA (gov.uk), PMDA (pmda.go.jp)
- NIST (nist.gov) for cybersecurity frameworks

### 12.5.3 Citation and Evidence Rules
Every major claim in the web-researched summary must have:
- at least one citation URL, and ideally
- a short quoted snippet (or paraphrase attribution) stored in the Evidence Panel

The output must include:
- “Sources consulted” section (bulleted list with access date)
- “Evidence map” table: claim → source(s) → confidence level (High/Med/Low)

### 12.5.4 Rate Limits & Caching
- Cache fetched pages by URL + timestamp (session cache)
- Enforce per-domain rate limits
- Show status: “research in progress” with counts (pages fetched, failures, retries)

---

# 13. MDRI Agent Chain Specification (3 Agents)

All three agents use the common Agent Studio framework:
- prompt editable
- model selectable
- output editable and can be approved to feed next agent
- artifacts versioned with diffs

## 13.1 Agent 1 — Web-Researched Comprehensive Summary (2000–3000 words)

### Purpose
Create a comprehensive, well-cited summary of regulations/guidance/standards related to the user’s pasted content, **including web research** where available.

### Inputs
- Normalized user input text (and optional uploaded docs)
- Jurisdiction hints (optional)
- Output language (EN/繁中)
- Research connector results (URLs + snippets)
- User constraints:
  - word count target: 2000–3000
  - formatting: Markdown

### Output (Markdown) — Required Sections
1. **Title**
2. **Executive Summary**
3. **What the user provided (context)**
4. **Regulatory landscape overview**
5. **Key requirements / expectations**
6. **Recent updates & trends (from web research)**
7. **Implications for manufacturers / QMS / submissions**
8. **Open questions / items to verify**
9. **Evidence Map (table)**
10. **Sources consulted (with access date)**

### Quality Controls
- Must avoid overclaiming; label uncertainty clearly.
- Must include citations for non-trivial assertions.
- Must be readable and structured (headings, lists, tables).

### User Actions
- Edit in text/markdown
- Download as `.md`
- “Approve as Input to Agent 2”

---

## 13.2 Agent 2 — Comprehensive Medical Device Regulation Report (3000–4000 words)

### Purpose
Generate a full report aligned to the selected template (default or user-provided), incorporating:
- the original user text
- Agent 1 summary (edited/approved)
- evidence and citations

### Inputs
- Template content (default or user-provided)
- Approved output of Agent 1 (or user-edited)
- Output language selection
- Optional: desired tone (academic / executive / regulatory memo)

### Output — Required Behaviors
- Must follow the template’s section ordering as closely as possible.
- Where template has references, preserve/expand them with updated citations.
- Must include:
  - regulatory scope and applicability
  - definitions and acronyms
  - mapping between cybersecurity (if relevant) and QMS/TPR/TPLC expectations (when applicable to the topic)
  - recommended actions for compliance (as recommendations, not legal advice)
  - explicit “assumptions and limitations” section

### Output Structure Handling
- If the template is the provided FDA cybersecurity template:
  - expand the “指引文件” discussion with latest updates and cross-links
  - keep the “小結” and “參考資料” sections, enhancing citations
- If user template differs:
  - follow user headings
  - if a section is missing but important, add an appendix (“Appendix: Additional Considerations”)

### User Actions
- Edit in text/markdown
- Download as `.md`
- Approve as input to Agent 3 (optional)

---

## 13.3 Agent 3 — Generate `skill.md` (Reusable Skill Description)

### Purpose
Create a `skill.md` document that describes how an agent should reliably produce a comprehensive medical device regulation report for similar inputs in the future.

### Alignment with “skill creator skill”
The generated `skill.md` must explicitly encode:
- when to trigger the skill (user intent patterns)
- required inputs (pasted text, optional template, output language)
- step-by-step method (research → summary → report)
- quality checklist (citations, evidence map, disclaimers, word counts, bilingual handling)
- output format requirements (Markdown sections, tables)
- evaluation suggestions (what “good” looks like, how to verify)

### Inputs
- User’s original input
- Template used
- Agent 1 and Agent 2 approved outputs (or their outlines)
- User language preference

### Output: `skill.md` Required Sections
1. **Frontmatter-like identification** (name, description, compatibility) — conceptually consistent with skill systems  
2. **When to use this skill (trigger guidance)**
3. **Inputs required & optional**
4. **Workflow steps**
5. **Output format templates** (for summary + report)
6. **Citation & evidence rules**
7. **Bilingual output rules**
8. **Quality checklist**
9. **Failure modes & recovery** (no web access, insufficient sources, ambiguous jurisdiction)
10. **Test prompts (2–3)** for future validation

### User Actions
- Edit `skill.md`
- Download `skill.md`

---

# 14. MDRI Artifacts, Versioning, and Downloads

## 14.1 Artifact Types
- `mdri_input_original.md/txt`
- `mdri_input_normalized.md`
- `mdri_summary_vN.md` (Agent 1)
- `mdri_report_vN.md` (Agent 2)
- `skill_vN.md` (Agent 3)
- `sources.json/md` (URLs, titles, timestamps, snippets)
- `evidence_map.md` (claim-to-source mapping)

## 14.2 Editing Model
- Each artifact supports:
  - text view
  - markdown source + preview
  - diff against previous version
- “Approve as next input” creates a locked snapshot to maintain provenance.

## 14.3 Downloads
- Per-artifact download
- “Download MDRI Bundle”:
  - all `.md` outputs
  - evidence + sources export
  - optional dashboard run log (redacted)

---

# 15. Localization Requirements (MDRI-Specific)

## 15.1 Output Language Toggle
User selects output language:
- English
- Traditional Chinese (繁體中文)

The agent must:
- maintain consistent terminology
- translate section headers appropriately (or preserve user template language)
- preserve citations and source titles in original language with optional translated annotation

## 15.2 Mixed-Language Templates
If user template is Traditional Chinese but user wants English output (or vice versa):
- Default behavior: **follow the template language for headings**, and generate body text in the selected output language unless user overrides.
- Provide a toggle:
  - “Follow template language strictly”
  - “Translate headings to output language”

---

# 16. Non-Functional Requirements (Expanded)

## 16.1 Performance
- Web research and long outputs must show progress indicators:
  - research fetch progress
  - synthesis progress
- Support chunking strategy for long inputs:
  - extract key points → synthesize → expand to target word count

## 16.2 Reliability
- Retry policy per provider
- Transparent failure messages:
  - “Web research unavailable; generating from provided text only”
- Validation checks:
  - word count range enforcement (soft enforcement with warnings)
  - presence of citations + sources section
  - presence of evidence map table

## 16.3 Security & Privacy
- API keys redacted everywhere
- Web fetching respects:
  - allowlist (default) + user override
  - basic safeguards against fetching arbitrary internal addresses
- Exports exclude keys and sensitive session metadata.

---

# 17. Updates to `agents.yaml` (Conceptual Only)

This release adds a new pipeline definition conceptually named:
- `mdri_web_summary`
- `mdri_template_report`
- `mdri_skill_md_generator`

They are runnable:
- individually (Agent Studio)
- sequentially (Pipeline Runner)
with consistent UI controls for prompt/model per step and editable output gating.

---

# 18. Acceptance Criteria (Delta + Full Coverage)

## 18.1 Must Keep (Regression-Proof)
- WOW UI: Light/Dark, EN/繁中, 20 painter styles + Jackslot
- WOW status indicators + interactive dashboard with telemetry
- API key UX:
  - env-first, never displayed
  - UI entry only if missing, masked, session-only
- Agents:
  - per-step prompt editing
  - model selection among listed models
  - run one-by-one
  - edit agent output and feed to next
- AI Note Keeper features and AI Magics
- Directory scan + OCR to Markdown with language options
- Skill runner + skill composer/QA/multipass transformer

## 18.2 New MDRI Must-Haves
- User can paste medical device regulation content (txt/md; optional pdf upload)
- User can paste a report template or choose default template (provided FDA cybersecurity template)
- User can select output language (EN/繁中)
- Agent 1:
  - performs web research (or transparently degrades)
  - produces 2000–3000 word Markdown summary
  - includes sources section + evidence map
  - result editable + downloadable
- Agent 2:
  - produces 3000–4000 word Markdown report aligned to template
  - includes assumptions/limitations and citations
  - editable + downloadable
- Agent 3:
  - generates `skill.md` describing how to produce similar reports
  - aligned to “skill creator skill” intent (triggering guidance, workflow, QC, test prompts)
  - editable + downloadable
- MDRI outputs integrate with dashboard logs and export bundles.

---

## 19. Proposed Future Enhancements (Optional, Not Required Now)
- Jurisdiction-specific report modes (FDA / EU MDR / UKCA / TFDA)
- Regulatory change monitoring subscriptions (watchlists)
- Structured extraction of “regulatory obligations” into a compliance checklist table
- Reviewer workflow: assign sections to reviewers and track approvals

---

# 20. Follow-up Questions (20 Comprehensive)

1. **Web search mechanism:** Do you want a dedicated search API (e.g., SerpAPI/Bing) integrated, or should MDRI rely on an authoritative-domain allowlist + user-provided URLs only?  
2. **Citation strictness:** Should the system *block* completion if citations/evidence map are missing, or only warn and allow download?  
3. **Source policy:** Do you want a default allowlist limited to regulator/standards bodies only (FDA/EU/NIST/IMDRF/ISO), or include reputable secondary sources (law firms, consultancies) with labeling?  
4. **Jurisdiction detection:** Should the system auto-detect jurisdiction (FDA/EU/etc.) from text, or require the user to confirm before web research/report generation?  
5. **Output language behavior with templates:** When template language conflicts with output language, should headings be preserved as-is, translated, or user-selectable per run?  
6. **Word count enforcement:** If the model cannot reliably hit 2000–3000 / 3000–4000 words due to token limits, should the system split into multi-part outputs (Part 1/2), or compress content with an explicit warning?  
7. **Evidence map design:** Do you prefer evidence mapping at the **claim level** (more granular, longer) or **section level** (shorter, easier to read)?  
8. **Quotation policy:** Should web research snippets be quoted verbatim (risk: copyrighted text) or summarized with short quoted fragments only?  
9. **PDF handling in MDRI:** Should MDRI accept PDFs as core inputs (with OCR option) similarly to Directory/OCR module, or keep MDRI text-first?  
10. **Template library:** Besides the provided FDA cybersecurity template, do you want a built-in library of additional templates (EU MDR PMS, risk management, clinical evaluation, etc.)?  
11. **Report “tone” presets:** Should Agent 2 support presets like “Executive memo,” “Regulatory affairs briefing,” and “Audit-ready report,” each with distinct formatting expectations?  
12. **Compliance disclaimers:** Do you want a standardized disclaimer section inserted into every MDRI output (with bilingual variants), or only shown in UI?  
13. **Acronyms/definitions:** Should reports always include a glossary (SaMD, MDDS, TPLC, SBOM, etc.), even if not requested by the template?  
14. **Change tracking:** Do you want the system to generate a “What’s new since last report” delta if the user uploads a previous report version?  
15. **Reviewer workflow:** Should there be an approval/sign-off mechanism (per section) with audit logs for regulated environments?  
16. **Model defaults:** For each agent (summary/report/skill.md), which default models should be recommended (speed vs quality), and should the system auto-switch when rate-limited?  
17. **Anthropic model list:** Which specific Anthropic models do you want exposed in UI (since “anthropic models” is broad), and should the list be configurable via Settings?  
18. **Grok usage:** Do you want Grok models available everywhere, or only in certain tools (e.g., reasoning-heavy evidence mapping) due to cost/latency considerations?  
19. **Export formats:** Besides Markdown downloads, do you need DOCX/PDF export (even if implemented later), or is `.md` sufficient for your workflow?  
20. **Skill.md expectations:** Should `skill.md` be optimized for a specific skill framework/consumer (your internal agent system, a known “skills” loader), or remain generic and human-readable with clear trigger guidance?
