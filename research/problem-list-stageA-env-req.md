# Agent Builder Log — Problem List: Stage A (Environment Setup + Requirements)

Research window: 2026-11-23 → 2026-11-30. Every item is URL-backed from live web
searches (GitHub issues, vendor docs, engineering blogs, Reddit/LinkedIn-linked
write-ups). Nothing invented; items without real evidence were dropped.

---

## ENVIRONMENT SETUP

### 01. LLM-stack dependency hell: the pydantic v1↔v2 split broke every agent scaffold
[LLM-stack dependency hell: the pydantic v1↔v2 split broke every agent scaffold](https://github.com/langchain-ai/langchain/issues/6841)

- **Lifecycle stage:** Environment setup (dependency pinning / version conflicts)
- **Problem:** For months in 2023–24, the hottest packages in the LLM stack were on opposite sides of the pydantic v1/v2 line — LangChain was `pydantic<2`, while FastAPI, Supabase, Gradio, llama-cpp-python and the OpenAI SDK all needed v2. Any project combining them hit `pip` conflicts that pip's own resolver "does not currently take into account", forcing impossible choices ("Which version of Pydantic do I use?"). Users describe being "stuck in dependency conflict hell". Maintainers in langchain-ai/langchain#29347 show the failure persists: installing "latest" langchain packages at different times produces `langchain-core`/`langchain`/`langchain-text-splitters` conflicts that are only resolvable by upgrading the whole set at once.
- **Evidence:**
  - https://github.com/langchain-ai/langchain/issues/6841 (pydantic v2 support; user cries of "dependency conflict hell")
  - https://github.com/langchain-ai/langchain/issues/8724 (`langchain ... requires pydantic<2,>=1, but you'll have pydantic 2.1.1`)
  - https://github.com/pydantic/pydantic/issues/8458 (pydantic 2.5.3 vs langchain 0.0.148 vs gradio vs openai — no compatible answer)
  - https://github.com/langchain-ai/langchain/issues/29347 (maintainers: "Debugging dependency conflicts is difficult"; fix = upgrade whole package family)
- **Ecosystem gap:** Framework majors land faster than transitive deps catch up; there is no coordinated migration bus for the AI package graph, and pip's "does not take into account all installed packages" behavior makes partial upgrades silently broken.
- **Solution sketch:** Pin the entire framework family (`langchain-core`+`langchain-*`) to one release train in a lockfile; isolate each agent runtime in its own venv; treat `pip check` in CI as a gate; when conflicts appear, upgrade the family wholesale (`pip install -U langchain-core langchain-anthropic ...`), never surgically.
- **Difficulty:** Medium · **Priority:** 1
- **Zafar-backing:** Unknown — highly plausible (near-universal for anyone who scaffolded an agent app in 2023–2025, but not confirmed from Zafar's notes).

### 02. Secrets in `.env` — and now the AI agent commits them itself
[Secrets in `.env` — and now the AI agent commits them itself](https://dev.to/ulnit/my-ai-agent-pushed-a-live-api-key-to-a-public-github-repo-i-didnt-notice-for-4-days-307)

- **Lifecycle stage:** Environment setup (API keys / .env handling)
- **Problem:** A one-person business running a fleet of AI agents on Raspberry Pis had an agent scaffold a service, read the Postmark API key **value** out of `.env`, embed it in a `config.yaml`, commit both (including the `.env` itself) to a **public** repo created by the agent — and nobody noticed for four days until the email provider's leak-scanner revoked the key. Root causes echo across the ecosystem: `.gitignore` only protects forward, `COPY . .` into Docker images leaks `.env`, `NEXT_PUBLIC_`/`VITE_` prefixed variables ship keys into browser bundles, and git history keeps leaked strings retrievable after deletion. Public scanners index `sk-`/`sk-proj-` patterns from git history within minutes.
- **Evidence:**
  - https://dev.to/ulnit/my-ai-agent-pushed-a-live-api-key-to-a-public-github-repo-i-didnt-notice-for-4-days-307 (full post-mortem; five guardrails)
  - https://rafter.so/blog/secrets/openai-api-key-exposure (scanner behavior, revocation order, `git filter-repo` history scrub)
  - https://zev.averba.ch/oops (OpenAI invalidated an exposed token; automated encryption of secret files)
  - https://github.com/Banbury-inc/Banbury/issues/263 ("[Security] Your openai API key was committed to this repo")
- **Ecosystem gap:** Secret management guidance predates agentic coding: agents treat instructions literally ("write the config, push it"), get raw secret **values** instead of names, and no tooling defaults new repos to private or blocks secret-bearing pushes at agent speed.
- **Solution sketch:** Give agents env-var **names**, never values, with a hard "never write secret values into files/logs/output" prompt rule; run `gitleaks` as a pre-commit hook on every machine an agent can push from; enable GitHub secret scanning + push protection org-wide; scope keys per-project with least privilege (send-only keys); revoke first, then `git filter-repo`, then usage audit.
- **Difficulty:** Hard · **Priority:** 1
- **Zafar-backing:** Unknown — the agent-commits-the-key variant is 2026-era and novel; Zafar ships code and notebooks to a public repo, making a near-miss plausible, but nothing confirms it.

### 03. NumPy 2.0's ABI break silently disabled every compiled AI package in existing environments
[NumPy 2.0's ABI break silently disabled every compiled AI package in existing environments](https://github.com/vllm-project/vllm/issues/5587)

- **Lifecycle stage:** Environment setup (version churn / dependency pinning)
- **Problem:** NumPy 2.0 (June 2024) broke binary compatibility: any package compiled against the NumPy 1.x C-API dies at import with `A module that was compiled using NumPy 1.x cannot be run in NumPy 2.0.0` / `_ARRAY_API not found`, and vLLM users hit `ModuleNotFoundError: No module named 'numpy.lib.function_base'` because `outlines` and `tensorizer` didn't cap `numpy<2` on the day of release. The numpy tracking issue (#26191) lists dozens of downstream packages racing to ship compatible releases; vLLM support only landed months later.
- **Evidence:**
  - https://github.com/vllm-project/vllm/issues/5587 (install-time breakage; workaround `pip install numpy==1.26.4`)
  - https://numpy.org/doc/stable/user/troubleshooting-importerror.html (official error catalog: ABI, `_ARRAY_API`, dtype size)
  - https://github.com/numpy/numpy/issues/26191 (ecosystem compatibility tracking)
  - https://pythonspeed.com/articles/numpy-2/ ("why you need to pin dependencies" — direct vs transitive dep lists + lockfile)
- **Ecosystem gap:** A 0.x→2.0 major with hard ABI break in the single most foundational package, partially absorbed because the ecosystem's solver only sees declared constraints — and most packages declared `numpy>=1.20` instead of `<2`.
- **Solution sketch:** App-level **lockfile** pins the whole graph (transitive included), not just direct deps; keep a CI job that resolves with latest to catch the "rebuild broke my env" day; where a dependency lags, temporarily pin `numpy<2` with a tracked ticket; treat import-time ABI errors as "upgrade the failing package, not numpy".
- **Difficulty:** Medium · **Priority:** 2
- **Zafar-backing:** Unknown — plausible for anyone maintaining an ML/vector env through mid-2024; no confirmation.

### 04. uv's resolver says "no solution found" on real AI projects — and can pin the wrong Python
[uv's resolver says "no solution found" on real AI projects — and can pin the wrong Python](https://github.com/astral-sh/uv/issues/17649)

- **Lifecycle stage:** Environment setup (dependency pinning / Python version conflicts)
- **Problem:** The lockfile tooling that was supposed to end dependency hell has its own sharp edges. LLaMA-Factory's install fails resolution because vLLM pins exact `torch==2.4.0` while sglang pins `torch==2.6.0` — an unsatisfiable set (hiyouga/LLaMA-Factory#8309). OLMo hit the same class of failure (#798). Open issues document the resolver "pinning a direct dependency instead of backtracking, causing Python-incompatible resolution" (#17649), and monorepo workspaces choke when one member needs Python 3.12 and another 3.13 (#17309, "Conflicting Python Requirements"). uv docs confirm: `requires-python` must be a subset of every dependency's range, and un-bounded deps can backtrack to ancient broken versions.
- **Evidence:**
  - https://github.com/astral-sh/uv/issues/17649 (resolver pins direct dep, produces Python-incompatible resolution)
  - https://github.com/hiyouga/LLaMA-Factory/issues/8309 (vLLM vs sglang torch pins → "No solution found")
  - https://github.com/allenai/OLMo/issues/798 (uv resolution failure on a research repo)
  - https://docs.astral.sh/uv/concepts/resolution/ (universal resolution constraints, lower bounds guidance)
- **Ecosystem gap:** Heavy GPU packages ship exact-pinned torch versions, making shared resolution objectively impossible; uv's universal-resolution default turns one machine's install into an all-platforms problem; error output is solver-speak, not actionable.
- **Solution sketch:** Declare `requires-python` explicitly and keep it tight; split GPU-heavy deps into extras (`uv sync --extra torch`); move cross-cutting conflicted packages into path-sourced workspace members; add lower bounds everywhere (`uv add` warns when missing); test with `--resolution lowest-direct` to catch overly-tight bounds.
- **Difficulty:** Medium · **Priority:** 2
- **Zafar-backing:** Unknown — plausible if Zafar's repos mix ML libs; no evidence either way.

### 05. PEP 668 "externally-managed-environment" — distro Python refuses pip, AI tutorials still say `sudo pip install`
[PEP 668 "externally-managed-environment" — distro Python refuses pip, AI tutorials still say `sudo pip install`](https://www.kali.org/blog/python-externally-managed/)

- **Lifecycle stage:** Environment setup (local dev envs, venv usage)
- **Problem:** Recent Debian/Ubuntu/Kali ships of Python are marked "externally managed": any `pip install` outside a venv dies with `error: externally-managed-environment`, including `pip install --user`. Kali shipped it by default and then reverted temporarily because of the flood of user reports; pip issue #13249 shows even `pip install --user` trips it ("not a bug" — by design). Meanwhile countless AI/ML tutorials still advise `sudo pip install`, so beginners — exactly the audience installing llama-cpp, transformers, notebook stacks — hit a wall they can't interpret, or worse, reach for `--break-system-packages`.
- **Evidence:**
  - https://www.kali.org/blog/python-externally-managed/ (shipment story, "we reverted it — but not for long")
  - https://pythonspeed.com/articles/externally-managed-environment-pep-668/ (why the guard exists, venv guidance)
  - https://github.com/pypa/pip/issues/13249 (`pip install --user` trips it; closed as not-a-bug)
  - https://discuss.python.org/t/error-externally-managed-environment/24237
- **Ecosystem gap:** The official guidance (venvs) still isn't the default in instructions; the error message itself is the only education most users get, and `--break-system-packages` is one Stack Overflow answer away from breaking their OS package manager (dnf/apt).
- **Solution sketch:** Standardize team onboarding on `uv venv`/`uv` (or `python3 -m venv`) with a documented `python -m pip` path; use `pipx` for installing Python *applications*; treat `--break-system-packages` as forbidden in docs; add an env-setup check to the repo README funnel.
- **Difficulty:** Easy · **Priority:** 3
- **Zafar-backing:** Unknown — extremely common friction for Linux-based AI devs; plausible but unconfirmed.

### 06. Fresh Python versions strand AI stacks: "No matching distribution found for torch" on 3.13
[Fresh Python versions strand AI stacks: "No matching distribution found for torch" on 3.13](https://github.com/pytorch/pytorch/issues/130249)

- **Lifecycle stage:** Environment setup (Python version conflicts)
- **Problem:** New CPython releases ship with no wheels for the heavy AI libs. PyTorch's own tracking issue for Python 3.13 ran for months; `torchvision` asked "Support for python 3.13" and stayed open until 2025; torch.compile support was only declared complete in early 2025. Meanwhile a fresh macOS with Python 3.13.2 gets `ERROR: Could not find a version that satisfies the requirement torch (from versions: none)` — a message that reads like a network failure rather than "your interpreter is too new".
- **Evidence:**
  - https://github.com/pytorch/pytorch/issues/130249 (PyTorch Python 3.13 support tracking)
  - https://github.com/pytorch/vision/issues/8747 (torchvision 3.13 support)
  - https://stackoverflow.com/questions/79512485/why-isnt-pip-finding-torch-when-installing-on-mac (3.13.2, `from versions: none`)
  - https://dev-discuss.pytorch.org/t/torch-compile-support-for-python-3-13-completed/2738
- **Ecosystem gap:** Compiled-wheel ecosystem mires behind the CPython release calendar; neither pip nor the framework docs give a clear "supported Python versions" gate at install time, and "no matching distribution" hides the real cause.
- **Solution sketch:** Check the framework's supported-Python matrix *before* provisioning the project interpreter; let `uv` manage per-project Python versions (`uv python install 3.12 && uv python pin`); treat "latest Python" as a deliberate migration with its own ticket, not the default for new agent projects.
- **Difficulty:** Easy · **Priority:** 2
- **Zafar-backing:** Unknown — plausible, standard trap; unconfirmed.

### 07. MCP server bring-up dies on environment plumbing: `npx` resolution, path configs, Windows
[MCP server bring-up dies on environment plumbing: `npx` resolution, path configs, Windows](https://github.com/modelcontextprotocol/servers/issues/76)

- **Lifecycle stage:** Environment setup (local dev envs, tool/server config)
- **Problem:** The single most common MCP setup failure is not MCP itself but the `stdio` transport's dependency on the host resolving `npx`/`uvx`: `spawn npx enoent`, `MCP error -32000: Connection closed`, servers failing under NVM, pnpm-wrapper (`pnpx`) failures on Windows, and GitHub MCP failing to start with an `npx` command error. Fixes are workarounds: absolute paths to the node executable, global installs instead of npx, hand-editing `~/.claude.json`/client configs. It is the first-hour experience of every new MCP user.
- **Evidence:**
  - https://github.com/modelcontextprotocol/servers/issues/76 (npx-based servers fail under NVM; global install + absolute paths fix)
  - https://github.com/modelcontextprotocol/servers/issues/1097 (GitHub MCP Server "npx Command Error and Connection Closed (-32000)")
  - https://github.com/cline/cline/issues/1948 (`spawn npx enoent`)
  - https://github.com/anthropics/claude-code/issues/3910 (pnpx on Windows → `McpError -32000`)
- **Ecosystem gap:** MCP config is a JSON blob of raw command strings in per-client files with no standard diagnostics, no `mcp doctor` equivalent, and no install-time validation; each client re-implements resolution differently.
- **Solution sketch:** Install servers globally or via `uvx` with pinned versions and **absolute** runtime paths in the config; add a `mcp list`/`mcp get` health check step to onboarding docs; keep a smoke-test script that spawns the server and lists tools before the first agent run.
- **Difficulty:** Easy · **Priority:** 2
- **Zafar-backing:** Unknown — plausible for anyone running MCP-connected agents locally; unconfirmed.

### 08. OpenAI SDK/API shape-churn: Chat Completions → Responses migration taxes every agent integration
[OpenAI SDK/API shape-churn: Chat Completions → Responses migration taxes every agent integration](https://developers.openai.com/api/docs/guides/migrate-to-responses)

- **Lifecycle stage:** Environment setup (SDK versions / API surface churn)
- **Problem:** The API surface agent code is written against keeps moving: `functions`/`function_call` are deprecated in favor of `tools`/`tool_choice` in the SDK types; the Responses API replaces `messages[]` with typed `input`/`output` items, moves `response_format` to `text.format`, restructures function calls around `call_id`, drops `n`, and — per the function-calling guide — new models (e.g. GPT-6 family) *require* the Responses API for tool calling, forcing migration on a deadline. The migration guide is now a checklist every integration has to re-run; per recallrun.dev, most simple code paths rename `messages`→`input` and `choices[0].message.content`→`output_text`, but tool-call plumbing genuinely restructures.
- **Evidence:**
  - https://developers.openai.com/api/docs/guides/migrate-to-responses ("Map Messages to Items", function definitions "externally tagged" → "internally tagged", strict mode changes)
  - https://recallrun.dev/posts/openai-responses-vs-chat-completions (code-level diff of what changes)
  - https://github.com/openai/openai-python/blob/v2.41.0/src/openai/types/chat/completion_create_params.py (`function_call`/`functions` documented "Deprecated in favor of `tool_choice`/`tools`")
  - https://developers.openai.com/api/docs/guides/function-calling
- **Ecosystem gap:** Model capability is increasingly gated behind the newest API shape, so SDK-version pinning alone doesn't protect you — the *protocol* moves under you, and deprecation notices in SDK docstrings are the only early warning.
- **Solution sketch:** Wrap the provider SDK behind a thin internal client adapter (one module owns all request/response mapping); pin `openai>=X` knowingly and subscribe to deprecation signals; allocate migration time as a standing item whenever a new API generation ships; make the adapter the enforcement point for outputs (typed Pydantic models), not the SDK.
- **Difficulty:** Medium · **Priority:** 2
- **Zafar-backing:** Unknown — Zafar ships agents on OpenAI-compatible APIs where this churn is inescapable; plausible, unconfirmed.

---

## REQUIREMENTS

### 09. "Add AI" scope vagueness: one phrase, five different stakeholder expectations
["Add AI" scope vagueness: one phrase, five different stakeholder expectations](https://ones.com/blog/requirements-gathering-practices-to-clarify-ai-project-scope)

- **Lifecycle stage:** Requirements (gathering needs from internal/external users)
- **Problem:** "Add AI" arrives as a one-liner and can mean summarizing support cases, predicting demand, or answering from internal knowledge — stakeholders each expect a different outcome, technical teams discover constraints late (data access, privacy, ops limits), and nobody agrees what success looks like. The ONES practice guide's central thesis: an AI project's scope can only be clarified by linking a specific user need to a bounded use case, known constraints, and measurable acceptance criteria *before* choosing a model — and by involving the people who use, operate, govern, or are affected by the system.
- **Evidence:**
  - https://ones.com/blog/requirements-gathering-practices-to-clarify-ai-project-scope (use-case-first practice list, stakeholder/owner assignment, constraint-vs-assumption discipline)
  - https://big-agile.com/blog/ai-makes-bad-requirements-more-expensive ("working export button... with tests... wrong feature" — AI accelerates delivery of the wrong thing)
- **Ecosystem gap:** No standard "AI feature brief" shape: problem statement, out-of-scope list, human-decision boundaries, and failure behavior ("what happens when the AI can't answer") are not part of the generic ticket template, and AI velocity turns vague tickets into confident wrong features within hours.
- **Solution sketch:** One-sentence use case + explicit in-scope/out-of-scope boundaries; named owners per requirement and open question; check data/privacy/technical constraints early and label confirmed-vs-assumed; define the "AI cannot help" path and human review responsibilities before building.
- **Difficulty:** Medium · **Priority:** 1
- **Zafar-backing:** Unknown — plausible; Zafar's series is aimed at exactly this gap, so resonance is likely but not documentable.

### 10. Missing agent requirements is the #1 cited cause of agent failure (hallucinated pricing, ignored business rules)
[Missing agent requirements is the #1 cited cause of agent failure (hallucinated pricing, ignored business rules)](https://botsify.com/blog/ai-agent-requirements)

- **Lifecycle stage:** Requirements (scoping agent projects)
- **Problem:** Canonical failure story: leadership approved budget, vendor demos impressed, six months and $200K later the agent "could technically answer customer questions" but hallucinated pricing, ignored business rules, and the support team abandoned it after two weeks. The vendors' own teardown: "poor requirements are the #1 cause of AI agent failure... most failed AI projects fail before any code is written", and agent projects need *more* stakeholders than traditional software (business owner, end users, ops, security/governance, compliance) with success criteria defined before features.
- **Evidence:**
  - https://botsify.com/blog/ai-agent-requirements (five requirement categories; success-criteria-first; stakeholder checklist; living spec)
- **Ecosystem gap:** Framework/docs address *how* to build agents, not *what* an agent project must specify: knowledge boundaries, tool permissions, failure behavior, escalation rules, and acceptance criteria have no standard template, so teams ship behavior instead of requirements.
- **Solution sketch:** Write a requirements doc as a living product spec with explicit sections: outcome + success metrics, users, functions, knowledge sources, tools + permissions, integration points, failure behavior, acceptance criteria; review it before any agent code; treat "agent works but metric didn't move" as failure.
- **Difficulty:** Medium · **Priority:** 1
- **Zafar-backing:** Unknown — vendor-authored, directionally consistent with Zafar's thesis ("when an agent is the wrong tool"); unconfirmed.

### 11. When an agent is the wrong tool: engineers who shipped agents and went back to scripts/workflows
[When an agent is the wrong tool: engineers who shipped agents and went back to scripts/workflows](https://kescoda.com/blog/when-not-to-use-ai-agents-seven-tasks-back-to-scripts)

- **Lifecycle stage:** Requirements (when an agent is the wrong tool)
- **Problem:** Practitioners report reverting agentic systems to plain scripts: "seven tasks where I went back to scripts" after a year of practice; a rule engineers converge on independently — if you can draw the full flowchart, you need a workflow, not an agent. The consistent frame (matching Anthropic's workflow-vs-agent split): use a script when rules are fixed, a workflow when steps are known, an agent only when the route genuinely depends on what's discovered, and keep a human when judgment/accountability is the product (approving payments, legal commitments).
- **Evidence:**
  - https://kescoda.com/blog/when-not-to-use-ai-agents-seven-tasks-back-to-scripts (first-hand reversions)
  - https://www.abhishekpatoliya.com/blog/ai-agent-vs-workflow-when-not-to-use-an-agent (decision checklist + veto questions)
  - https://lenvanderhof.com/en/blog/when-not-to-use-an-agent/ (5 signs; "draft-only" assistant still has a route past its instructions)
  - https://nyk.dev/blog/when-not-to-use-an-agent (7-step roadmap; "First Agent Scope Card")
- **Ecosystem gap:** The word "agent" now attaches to anything with an LLM inside, so teams reach for the most complex option first; there is no disciplined, shareable pre-build decision gate (flowchart test, enumerable steps, bounded damage, evaluability).
- **Solution sketch:** Run a scoping gate before any agent build: can the team draw the flowchart? is the wrong-action cost bounded or gated by human approval? is "done" checkable? If no to the first or yes to irreversible actions without approval, ship the workflow/script instead; use a scope card (job, trigger, inputs, tools, required output, approval-before, escalate-when, success measure) before choosing a framework.
- **Difficulty:** Easy · **Priority:** 1
- **Zafar-backing:** Unknown — this is one of Zafar's own planned coverage themes; the practice is well documented, but his personal reversions are unconfirmed.

### 12. Vague requests + able agents = the agent decides instead of asking: word-of-mouth requirements accelerate misalignment
[Vague requests + able agents = the agent decides instead of asking: word-of-mouth requirements accelerate misalignment](https://adservio.fr/en/insights/articles/ai-dlc-episode-1-besoin-oral)

- **Lifecycle stage:** Requirements (gathering needs; scoping agent projects)
- **Problem:** "An AI agent handed a vague request does not ask questions, it decides." Requirements arrive by word of mouth from a manager who knows the business but not how an agent will interpret it; a poorly scoped requirement used to slow a project down, but with AI it "accelerates misalignment" and takes effect within hours. The field report (energy sector, spoken brief) hides four question families — who, what comes first, what record, under what conditions — that never get asked.
- **Evidence:**
  - https://adservio.fr/en/insights/articles/ai-dlc-episode-1-besoin-oral (AI-DLC staged elicitation: agent asks, writes down, waits for approval at each stage; versioned documents; business in the loop)
  - https://ones.com/blog/how-to-use-ai-to-gather-software-requirements (controlled AI-assisted elicitation workflow; "careless automation creates polished ambiguity")
- **Ecosystem gap:** No enforcement of requirement elicitation as a *process*: agents default to executing hypotheses instead of surfacing ambiguity, and there's no standard "ask before building" protocol for agentic tools.
- **Solution sketch:** Gate agent projects behind a Q&A stage with written records and human approval per stage (who, precedence, source-of-record, conditions); convert user stories → acceptance criteria → tests; version the produced requirement documents; keep the business owner on a fast response loop.
- **Difficulty:** Medium · **Priority:** 2
- **Zafar-backing:** Unknown — plausible; unconfirmed.

### 13. MCP tool sprawl degrades agent accuracy: 167 tools ≈ 60K tokens before the first question
[MCP tool sprawl degrades agent accuracy: 167 tools ≈ 60K tokens before the first question](https://redis.io/blog/from-reasoning-to-retrieval-solving-the-mcp-tool-overload-problem/)

- **Lifecycle stage:** Requirements (scoping the tool surface / agent capability)
- **Problem:** Connecting the usual MCP servers (Redis 44 tools, GitHub 40, Jira 27, Grafana 56) yields 167 tools and ~60K tokens consumed before the user even asks — production setups hit 150K+. Consequences measured by Redis: accuracy dropped (models pick wrong tools, data gets overwritten, requests go to wrong places), costs and latency climbed; their trial showed 98% token reduction, 8× latency improvement, and 2× selection accuracy by fetching 3–5 relevant tools instead of 50+. Microsoft Research's survey of 1,470 MCP servers found tool name collisions, servers with up to 256 tools, and tool outputs of 128K+ tokens that swamp model context; OpenAI recommends fewer than 20 functions, and large tool spaces can lower performance by up to 85% for some models.
- **Evidence:**
  - https://redis.io/blog/from-reasoning-to-retrieval-solving-the-mcp-tool-overload-problem/ (numbers, tool-filtering solution)
  - https://www.microsoft.com/en-us/research/blog/tool-space-interference-in-the-mcp-era-designing-for-agent-compatibility-at-scale/ (survey, collisions, tool-count + response-length findings, MCP Interviewer)
  - https://meshes.io/blog/mcp-tool-sprawl (token cost of idle tools, description-quality competition, CRUD-per-destination compounding)
  - https://docs.aws.amazon.com/prescriptive-guidance/latest/mcp-strategies/mcp-tool-strategy-organization.html (domain-noun-verb naming; split servers >50 tools)
- **Ecosystem gap:** The MCP spec standardizes tool *delivery*, not tool *hygiene*: no namespacing guidance (collisions), no tool-count limits, no response-length limits, no model-aware descriptions — so "more capabilities" becomes the trap.
- **Solution sketch:** Treat the tool surface as a scoped requirement, not an integration list; cap tools per agent (<20 when possible); implement tool filtering/retrieval (vector/hybrid search over tool defs) for large surfaces; use domain-noun-verb naming and split any server over ~50 tools; keep tool responses small; validate servers with MCP Interviewer before wiring them in.
- **Difficulty:** Medium · **Priority:** 2
- **Zafar-backing:** Unknown — squarely inside Zafar's coverage interests; unconfirmed.

### 14. Evaluability as a requirements gate: if you can't define "done", you can't run an agent
[Evaluability as a requirements gate: if you can't define "done", you can't run an agent](https://wolinski.com/blog/agent-vs-deterministic-workflow.html)

- **Lifecycle stage:** Requirements (scoping; acceptance criteria)
- **Problem:** The recurring hard gate across practitioner guidance: if you cannot say what "done" looks like or test it, you cannot evaluate an agent — and deploying one you can't evaluate produces "a wrong medical summary... wrong financial report... wrong security classification" with no programmatic floor. Practice converges on two floors: a **code verifier** that approves agent output before it's accepted (the verifier-driven agent pattern — "in production it has held up as one of the more reliable ways to constrain non-determinism"), or a **human in the loop** as the legitimate floor for low-stakes work (drafts, summaries-for-review).
- **Evidence:**
  - https://wolinski.com/blog/agent-vs-deterministic-workflow.html (6-question matrix, hard/soft gates, verifier-driven pattern, operational floor)
  - https://promptalmanac.com/academy/agents-and-sub-agents/ ("If you cannot define success, you cannot evaluate an agent"; complexity ladder; test set first)
  - https://nyk.dev/blog/when-not-to-use-an-agent (scope card field: "Success measure / Baseline today")
- **Ecosystem gap:** Success criteria and test sets are treated as post-hoc evaluation chores instead of entry requirements; no tooling treats "you can't measure quality" as a build-blocker, so un-measurable agents ship and degrade silently.
- **Solution sketch:** Require, in the brief: a success measure, a baseline measured today, and a checkable failure condition before build approval; where a deterministic check exists, wire it into the loop as a verifier gate; otherwise name the human reviewer and the cost ceiling per wrong answer; refuse to build agents for tasks with no defined "done".
- **Difficulty:** Easy · **Priority:** 2
- **Zafar-backing:** Unknown — plausible; consistent with the series' evaluation-stage interests; unconfirmed.

---

### Summary counts
- Total items: 14 (8 environment setup, 6 requirements)
- Priority 1: #1 (pydantic dependency hell), #2 (secrets/.env), #9 ("Add AI" vagueness), #10 (missing agent requirements), #11 (when not to use an agent)
- All URLs verified in live search results during research window 2026-11-23.
