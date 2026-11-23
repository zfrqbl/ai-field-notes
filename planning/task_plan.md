# Agent Builder Log — Project Master Plan

**Working title (APPROVED 2026-11-23):** Agent Builder Log — notes numbered
#01, #02, … (Zafar's pick: "Agent builder log #xx")
**Author:** Zafar Iqbal (notes collected while building/delivering/maintaining apps)
**Audience:** Junior and mid-level engineers
**Publish:** zfrqbl.com — NEW section (approved) + LinkedIn, starting February 2027
**Cadence:** 1 note per week + 3 LinkedIn posts from different angles during the week
**Note length:** 700–900 words prose + code blocks (soft — Hermes decides per note)
**Code home:** new public repo zfrqbl/ai-field-notes (CREATED 2026-11-23)
**Research window:** Nov 23–30, 2026 | **Writing starts:** Dec 1, 2026

---

## 1. What this project is

A personal, technically dense log of problems Zafar hit while building,
delivering, and maintaining AI-enabled and AI-powered apps for internal and
external users — each note a self-contained tip with Python code. Not theory:
every note documents a real encounter, a real impact, and the fix that
actually worked (custom component, process addition, or a conversation with a
senior engineer, customer, or peer).

## 2. Note anatomy (every note follows this, in order)

1. **The problem** — the challenge/difficulty/frustration, stated concretely.
2. **The gap** — what's missing in today's agent-development ecosystem
   (tools, docs, frameworks, processes) that caused or amplified this.
3. **The discovery** — how Zafar encountered it (project context, what he was
   building, when it surfaced).
4. **The impact** — how it acted as a blocker/challenge/issue in one of:
   development, delivery, maintenance, requirements gathering, analysis.
5. **The fix** — how it was solved: custom component (code included), a new
   process step, talking to senior engineers, talking to customers, or peers
   suggesting a fix.
6. **References** — links to docs/sources/issues where relevant.

Target: 700–900 words of prose + code blocks; technically dense, no fluff
(soft constraint — Hermes decides the exact length per note).
Python code required where it illustrates the problem and the solution.

## 3. Coverage (the whole agent lifecycle, per the brief)

| Stage | What it covers |
|---|---|
| Env setup | Local dev envs, versions, API keys, .env, dependency pinning, uv/venv |
| Requirements | Scoping agent projects, when an agent is the wrong tool, gathering needs from internal/external users |
| Build | Orchestration (LangGraph/CrewAI/etc.), prompt engineering, tool calling, structured outputs, memory |
| Eval | Evals, golden sets, regression, cost-aware evaluation |
| Delivery | Deploying agents in CUSTOMER environments: Docker, air-gapped, auth, proxies, rate limits, GPU/CPU |
| Maintenance | Observability, logging, drift, model churn, cost control, retries/fallbacks |
| People | Customer expectation setting, internal vs external users, talking to seniors/peers |
| Security & Robustness | Prompt injection (direct, indirect, stored), tool poisoning / MCP trust boundaries, data leakage, guardrails, red-teaming before release (ADDED 2026-11-23 on Zafar's "Add the gaps") |

## 4. Research phase — Nov 23–30 (starts on plan approval)

- Scan LinkedIn (posts, comments, articles) + Google/web for real
  engineer-reported problems and solutions in agent development.
- Sources: LinkedIn posts/articles, engineering blogs, GitHub issues,
  Stack Overflow, Reddit, vendor docs — every item gets a URL.
- Output: an **extended problem list** (target 40–80 items), each entry:
  title, lifecycle stage, problem, evidence links, ecosystem gap, solution
  sketch, difficulty, priority (Jev-scored), and whether Zafar's own
  experience can back it.
- Nov 30: present the list to Zafar for prioritization and first-batch pick.
- The list also seeds Zafar's own notes: items he's personally hit get
  his real encounter written up; items he hasn't, get either dropped or
  research-only versions.

## 5. Writing phase — Dec 1 → Jan 2027

- One note per idea; each drafted → write-like-a-human pass → email for
  review → APPROVE → final. Review-first is locked.
- Every code-bearing note gets a companion Jupyter notebook (interactive
  material embedded in the note: run-in-Colab + GitHub links).
- Diagrams: generate only with Zafar's explicit permission (asked in Dec
  when the first need arises; the SVG/architecture skills cover this free).
- Batch of ~25–40 notes targeted before Feb.

## 6. Publishing phase — Feb 2027 onward

- zfrqbl.com: new "Agent Builder Log" series section (APPROVED — new
  section, not the notes pipeline), OG images, feed/archive/RSS updates,
  prose gate pass.
- LinkedIn: 3 companion posts per note from different angles (review-first,
  existing PostPeer/Composio pipeline).
- Cadence: 1 note per week, each with 3 LinkedIn posts during the week.

## 7. Tool requirements (free-first)

| Need | Tool | Cost | Already have? |
|---|---|---|---|
| Web research | web_search / web_extract / Exa MCP | Free | Yes |
| GitHub repos + notebooks | GitHub (gh CLI, GH_TOKEN) | Free | Yes |
| Interactive notebooks | .ipynb in repo + Colab links + nbviewer | Free | Yes (local Jupyter) |
| Diagrams | architecture-diagram / image_generate skills | Free | Yes (permission-gated) |
| LinkedIn publishing | PostPeer / Composio LinkedIn | Free (credits) | Yes |
| Tracking | Todoist | Free | Yes |

No paid tools required today. If LinkedIn's own search blocks scraping
(403/bot walls), the blocked-page-recovery or camofox browser paths are used
— still free. Any paid need will be flagged with justification before
anything is purchased.

## 8. GitHub repo (APPROVED + CREATED 2026-11-23)

1. **zfrqbl/ai-field-notes** (public, created Nov 23) — home repo:
   - `notes/<NN>-<slug>.md` — the note sources
   - `notebooks/<NN>-<slug>.ipynb` — interactive companions (embedded in notes)
   - `research/problem-list.md` — the living extended list
   - `assets/` — diagrams/images | README with series index + status
2. **zfrqbl/ai-field-notes-notebooks** (optional later) — only if notebooks
   deserve standalone hosting instead of a folder in repo 1.
3. Existing **zfrqbl/zfrqbl-website** — final published pages (Feb 2027).

## 9. Decisions locked by Zafar's reply (2026-11-23, thread 75003503)

1. Series name: **"Agent builder log #xx"** (notes numbered).
2. Note length: **700–900 words**, no fluff, not hard — Hermes decides.
3. Site home: **new section** on zfrqbl.com.
4. Cadence: **1 note/week + 3 LinkedIn posts** from different angles.
5. Coverage: include the ideas list in the series; **standing rule — ask
   Zafar before adding any idea** (whenever an article/topic comes up in
   research or discussion, ask rather than auto-add).
6. Repos: **code + notebooks in a new GitHub repo** — zfrqbl/ai-field-notes
   created 2026-11-23, note-wise entries.
7. **Plan APPROVED 2026-11-23 (thread 0e40ce56): "Approved. Add the gaps"** —
   the Q5 coverage-gap item is resolved: **Security & Robustness added as an
   8th lifecycle stage** (prompt injection, MCP tool poisoning, data leakage,
   guardrails, red-teaming), with its own problem list
   (`research/problem-list-stageD-security.md`, 4 items, series total now 50).

## 10. Timeline summary

| Date | Milestone |
|---|---|
| Nov 23 | Plan emailed for approval |
| Nov 23–30 | Research: LinkedIn + Google scan → extended list |
| Nov 30 | Extended list presented for prioritization |
| Dec 1+ | Writing: note 1 onward, notebooks, review emails |
| Dec (on need) | Diagram permission requests |
| Jan | Buffer, polish, prep publish pipeline |
| Feb 2027 | Go live: 1–2 notes/week on zfrqbl + LinkedIn |
