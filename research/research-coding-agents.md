# AI Software Factory — Framework-Neutral Coding Agent & Skill-Portability Research Report

> **最終決策備註（2026-11-07）**：本報告為原始調研紀錄，撰寫時以「Azure OpenAI Private Endpoint 或本地模型」與「Azure DevOps Server 地端」為假設前提。後續規劃已定案為：**完全地端 Kubernetes**、**唯一允許的雲端服務為 Azure DevOps Services**、**不使用任何雲端 LLM**（含 Azure OpenAI），模型一律為叢集內 vLLM 或公司內部 endpoint，經 LiteLLM Proxy 路由。閱讀時請以 `AI-Software-Factory-Plan.md` 為準，本文中的 Azure OpenAI / ADO Server 相關內容僅供參考。


*Research date: 2026-11-07. All URLs verified live during this session.*

---

## Part A — Coding Agents (headless/CLI/API, model neutrality, self-host, CI fit)

### GitHub Copilot CLI & Copilot coding agent
- **Description**: `copilot` CLI (npm `@github/copilot`, also WinGet/Homebrew) is an interactive + scriptable terminal agent. **Copilot coding agent** is GitHub's cloud/Actions-based autonomous agent assigned to issues/PRs.
- **Headless/CLI**: Copilot CLI supports non-interactive automation via slash-commands and scripting; auth via `COPILOT_GITHUB_TOKEN`/`GH_TOKEN` fine-grained PAT with "Copilot Requests" scope — scriptable in pipelines. ([docs](https://docs.github.com/en/copilot/how-tos/copilot-cli/set-up-copilot-cli/install-copilot-cli))
- **Model neutrality**: Locked to GitHub Copilot's hosted model menu (OpenAI/Anthropic/Google models via Copilot subscription) — **no Azure OpenAI private endpoint / local model support**.
- **Self-host**: No — SaaS only (cloud sandbox, GitHub Actions runner, or "remote" custom environment, but orchestration is GitHub-hosted). ([docs](https://docs.github.com/en/copilot/concepts/copilot-surfaces/copilot-on-github))
- **Azure DevOps fit**: **Copilot coding agent is GitHub-native only** — it operates on GitHub issues/PRs/Actions/cloud sandboxes; there is no first-party Azure Repos/Pipelines integration mentioned anywhere in current docs. Copilot CLI itself (as a terminal tool) can technically be invoked from any shell including an Azure Pipelines agent, but coding-agent-the-autonomous-service cannot be assigned Azure DevOps work items. **Verdict: not viable as the Azure DevOps autonomous agent; usable only as an interactive/CLI helper.**
- **License/pricing**: Requires a paid Copilot subscription (Business/Enterprise for coding agent).
- **Caveats**: Best treated as a GitHub-ecosystem tool only; exclude from the ADO-centric agent pool except as an optional "GitHub mirror" convenience.

### Claude Code (Anthropic)
- **Description**: Terminal coding agent; also shipped as **Agent SDK** (Python/TypeScript) and a thin **Managed Agents** hosted option.
- **Headless/CLI**: `claude -p "<prompt>" --allowedTools "Read,Edit,Bash" --output-format json` — fully headless, scriptable, structured JSON output, exit codes for CI branching, `--bare` for fast deterministic CI runs (skips auto-loaded hooks/skills/MCP/CLAUDE.md for reproducibility). ([docs](https://code.claude.com/docs/en/headless))
- **SDK**: Python/TypeScript SDK wraps the same agent loop, with hooks, subagents, MCP, permission callbacks, session resume/fork — ideal as an orchestrator-embeddable library. ([Agent SDK overview](https://code.claude.com/docs/en/agent-sdk/overview))
- **Model neutrality**: Native path is Anthropic's API (and Bedrock/Vertex); **no official Azure OpenAI/local-Ollama backend** — it is Claude-model-only (can proxy via LiteLLM for routing/metering, but still Anthropic-compatible models).
- **Self-host**: CLI/SDK run entirely in your infra (your process invokes the binary) — but it always calls out to Anthropic's (or Bedrock/Vertex) hosted model; "Managed Agents" sandbox can be **self-hosted** per Anthropic docs (`self-hosted-sandboxes`).
- **CI fit**: Excellent — designed for exactly this (hooks, `--output-format stream-json`, non-interactive exit codes).
- **License/pricing**: API-metered (Anthropic API key) or Claude subscription (not permitted for 3rd-party agent redistribution per SDK ToS).
- **Azure DevOps fit**: Agnostic to source control — it's just a CLI/SDK acting on a local checkout, so it works fine invoked from Azure Pipelines; PR creation would need `az repos pr create` wrapping, or an MCP server for ADO (see Part E).

### OpenAI Codex CLI
- **Description**: Apache-2.0 licensed local coding agent CLI (`npm install -g @openai/codex` or native binary). ([repo](https://github.com/openai/codex))
- **Headless/CLI**: Supports `codex exec`/non-interactive automation flags (documented at developers.openai.com/codex); scriptable for CI.
- **Model neutrality**: Primarily targets OpenAI models (ChatGPT plan sign-in or API key); some community forks allow OpenAI-compatible endpoints (which **Azure OpenAI exposes** via an OpenAI-compatible API, so Azure OpenAI can often be wired in via base-URL override — verify current flag support per release).
- **Self-host**: CLI binary runs locally; depends on OpenAI/Azure OpenAI hosted inference.
- **License**: Apache-2.0 (open source), usage billed via ChatGPT plan or API key.
- **Azure DevOps fit**: Source-control agnostic CLI; fine for ADO repos invoked from Pipelines.

### Google Gemini CLI
- **Description**: Open-source terminal agent (`@google/gemini-cli`), **now scoped to enterprise Gemini Code Assist Standard/Enterprise** licenses; the free/individual use case has moved to a *different* product, **Antigravity CLI** (`agy`), and there's also a separate stateless **Gemini API CLI**. ([repo](https://github.com/google-gemini/gemini-cli))
- **Headless/CLI**: Yes, scriptable; but explicitly **not tuned for Gemini 4+ models** and the maintainers warn against using it for model benchmarking.
- **Model neutrality**: Gemini-only (Vertex AI / Gemini Code Assist); no Azure/local-model path documented.
- **Self-host**: No — requires Google Cloud Gemini Code Assist license or Vertex key.
- **Azure DevOps fit**: Neutral CLI, same caveat as others — works on any repo checkout, but needs a Google Cloud Gemini Code Assist/Vertex contract. Licensing complexity (enterprise-only, no free tier) makes it a weak fit for a self-hosted, model-neutral org.

### Aider
- **Description**: Mature open-source (Apache-2.0) terminal pair-programming agent focused on diff-based edits + auto-commits.
- **Headless/CLI**: Purpose-built for scripting: `aider --message "..." file.py --yes --auto-commits` runs one instruction and exits cleanly — ideal for CI loops (`for FILE in *.py; do aider --message ... ; done`). ([docs](https://aider.chat/docs/scripting.html))
- **Python API**: `from aider.coders import Coder; Coder.create(...); coder.run(...)` — embeddable in an orchestrator (unofficial/unsupported but functional).
- **Model neutrality**: **Best-in-class** — works with any LiteLLM-compatible backend: OpenAI, Anthropic, Azure OpenAI, Ollama, vLLM-served local models, Bedrock, etc. (Aider uses LiteLLM under the hood for provider abstraction.)
- **Self-host**: Fully — it's a plain Python CLI; pair with a local Ollama/vLLM endpoint for 100% on-prem operation.
- **License/pricing**: Free/open source; you pay only for whichever LLM endpoint you configure (including $0 for local models).
- **Azure DevOps fit**: Excellent — pure git-based CLI, works against any git remote (Azure Repos) and in any CI runner; combine with `az repos pr create` to open PRs from pipeline steps.
- **Caveats**: Weaker "agentic" planning/tool-use loop than Claude Code/Codex/OpenHands (no native browser/test-running loop by default, though `--auto-commits`/lint-fix exist); best for implementation/refactor stage, less ideal as a full autonomous SDLC agent.

### OpenHands (formerly Devin-like open source; now "OpenHands / Agent Canvas")
- **Description**: Self-hosted "developer control center" that can run the open-source OpenHands agent **or** drive Claude Code, Codex, Gemini CLI, Pi, OpenCode, or any ACP (Agent Client Protocol)–compatible agent, across local/Docker/VM/cloud backends. ([repo](https://github.com/OpenHands/OpenHands))
- **Headless/API**: REST API + SDK; built for automations triggered by webhooks/schedules — exactly the orchestrator-dispatch model the user wants.
- **Model neutrality**: "Bring your own model" — any LLM via LLM profile config, including local/Ollama/vLLM and Azure OpenAI-compatible endpoints.
- **Self-host**: First-class — designed to "self-host your way": laptop, Docker, VM, or internal server. Sandboxed execution uses Docker containers per task.
- **License/pricing**: Open source (MIT-family), optional paid OpenHands Cloud/Enterprise.
- **Azure DevOps fit**: Good — it's repo/git-agnostic; GitHub/Linear integrations exist natively, Azure DevOps would need a custom webhook/automation or MCP wiring, but the underlying agent and sandbox model is ADO-compatible.
- **Caveats**: Rapidly evolving product (recently rebranded "Agent Canvas"); verify current API stability before committing.

### Cline / Roo Code / Kilo Code
- **Cline**: VS Code/JetBrains extension **plus** a real headless CLI (`npm i -g cline`) "fully headless for CI/CD and scripting" and a Node.js **SDK** (`@cline/sdk`) for building custom multi-agent integrations. ([repo](https://github.com/cline/cline))
  - Model neutrality: supports many providers (Anthropic, OpenAI, Azure OpenAI, Bedrock, Ollama, OpenRouter, etc.) via its model-provider settings — self-hostable, local-model friendly.
  - License: open source core; CLI/SDK free, cloud features may be paid.
- **Roo Code**: VS Code fork of Cline with "Modes" (Code/Architect/Ask/Debug/Custom) — good conceptual match for stage-specific sub-agents, but **primarily an IDE extension**, not headless-first; CLI/automation story weaker than Cline. ([repo](https://github.com/RooCodeInc/Roo-Code))
- **Kilo Code**: Another Roo/Cline-family fork (listed as a supported client in the Azure DevOps MCP docs) — same category, IDE-centric with MCP support; headless story not verified here — flag for follow-up.

### Goose (Block, now under Agentic AI Foundation/Linux Foundation)
- **Description**: General-purpose open-source agent — desktop app, **CLI**, and **API**, built in Rust. ([repo](https://github.com/block/goose) — now hosted at `aaif-goose/goose`)
- **Model neutrality**: Explicitly supports 15+ providers — Anthropic, OpenAI, Google, **Ollama, Azure, Bedrock**, OpenRouter, plus subscription pass-through via ACP.
- **Self-host**: CLI/API run locally; "Custom Distributions" let you build your own goose distro with preconfigured providers/extensions/branding — attractive for an internal, branded factory agent.
- **MCP**: Connects to 70+ extensions via MCP — directly usable with the Azure DevOps MCP server.
- **License/pricing**: Apache-2.0, fully open source, free.
- **Azure DevOps fit**: Strong — neutral, scriptable, MCP-native, governance-friendly (Linux Foundation project), good licensing clarity.

### Cursor (CLI / Background/Cloud Agents)
- **Description**: `cursor.com/install` → `agent` CLI binary with interactive, **print/non-interactive mode** (`agent -p "..." --output-format text`), plan/ask/agent modes, and sandbox controls. ([docs](https://cursor.com/docs/cli/overview))
- **Headless/CI**: Yes — `-p` flag is purpose-built for scripts/CI; can hand off to Cursor's hosted **Cloud Agent** (`&` prefix) for long-running background tasks.
- **Model neutrality**: Multi-model (GPT, Claude, Gemini, etc. selectable via `--model`), but routed through Cursor's own backend — **no self-hosted/local-model/Azure-private-endpoint mode** for the core agent.
- **Self-host**: No — Cursor is SaaS; the CLI still talks to Cursor's cloud for orchestration/billing even when editing local files.
- **License/pricing**: Commercial subscription (Pro/Business), CLI usage metered.
- **Azure DevOps fit**: CLI itself is git-agnostic (fine against Azure Repos checkouts in Pipelines), but Cloud/Background Agents are GitHub/GitLab-centric for PR workflows; **not self-hosted**, a blocker for strict on-prem/intranet requirements.

### Amazon Q Developer CLI → now Kiro CLI
- **Status**: The open-source `aws/amazon-q-developer-cli` repo is **archived/unmaintained** ("no longer being actively maintained, critical security fixes only") — successor is **closed-source Kiro CLI**. ([repo](https://github.com/aws/amazon-q-developer-cli))
- **Implication**: Don't build on the open-source Q Developer CLI going forward; if AWS-ecosystem agent is wanted, evaluate Kiro CLI directly (closed-source, so self-host/model-neutrality must be separately verified — likely Bedrock-centric, weaker fit for an Azure-first shop).

### Continue.dev
- Open-source IDE extension + CLI with strong **model-provider neutrality** (explicit support for Azure OpenAI, Ollama, local models, and dozens of providers via a `config.yaml`), and a "Continuous" headless mode for CI usage. Not deep-dived live in this pass (not fetched) — flagged as **worth a dedicated follow-up fetch** of `github.com/continuedev/continue` before final tool selection, but it is well known in the ecosystem for exactly the self-hosted/model-neutral requirement here.

### Sourcegraph Amp
- **Description**: "Orb"-based agent dev environment (cloud sandboxes called "Orbs", portals for live preview, multiplayer, automations triggered by webhooks/CI-failure/schedule). ([site](https://ampcode.com/))
- **Headless/CI**: Automations react to GitHub/GitLab events and CI failures — but the product is **cloud-hosted** (Orbs run on Sourcegraph's infra), not something you self-host on-prem.
- **Model neutrality**: Lets you "bring your ChatGPT subscription," but the core product is Sourcegraph's hosted agent — **not self-hostable**; disqualifies it for the strict on-prem/intranet constraint.
- **Azure DevOps fit**: Native GitLab/GitHub integration shown; no Azure DevOps mention — weak fit.

### Qwen Code
- **Description**: Open-source CLI/agent framework from Alibaba's Qwen team — "Auto-Memory, Auto-Skills, SubAgents, Agent Teams, MCP"; explicitly **multi-protocol**: OpenAI, Anthropic, Gemini, and Qwen APIs, **plus any third-party provider or local model (Ollama/vLLM)**, switchable at runtime. ([repo](https://github.com/QwenLM/qwen-code))
- **Self-host**: Framework and models both open source — can run fully air-gapped with local Qwen weights via Ollama/vLLM.
- **License/pricing**: Open source; Qwen subscription optional, or BYO key/local model for $0 inference cost.
- **Azure DevOps fit**: Strong candidate for the **local-model / cost-sensitive** slot in the agent pool — genuinely model-neutral and self-hostable, unlike most competitors above.

### Devin (Cognition)
- **Description**: Fully autonomous cloud SWE agent with web UI, Slack/Teams integration, and a documented **REST API** plus a **CLI** (`cli.devin.ai`) that can `/handoff` long tasks to the cloud. ([docs](https://docs.devin.ai/))
- **Headless/API**: Yes — API-first by design, good for orchestrator dispatch.
- **Model neutrality**: No — Devin is a closed, vertically-integrated product; you cannot swap in Azure OpenAI/local models.
- **Self-host**: No — SaaS only, data leaves your intranet. **Disqualified** for the stated self-hosted/on-prem constraint.
- **License/pricing**: Commercial, session/ACU-based pricing.

### SWE-agent / mini-SWE-agent (Princeton/Stanford)
- **Description**: Research-grade open-source agent framework, config-driven via a single YAML, designed to plug in "your LLM of choice." **Actively being superseded by `mini-swe-agent`** (same performance, ~100 lines of Python, much simpler). ([repo](https://github.com/SWE-agent/SWE-agent))
- **Model neutrality**: Excellent — works with GPT-4o, Claude, open-weight models, anything reachable via an API-compatible endpoint (fits Azure OpenAI/local vLLM easily).
- **Self-host**: Fully — pure Python, run anywhere, Codespaces-friendly but not required.
- **License**: MIT/open — research tool, not a polished product; best suited to the **benchmark/evaluation harness** role, or as a lightweight custom-agent base to fork, rather than a turnkey production agent.
- **Azure DevOps fit**: Agnostic; would need custom glue for ADO PR creation.

### Plandex
- **Description**: Open-source terminal agent specializing in **large, multi-file, multi-step plans** — 2M-token effective context via tree-sitter project maps, cumulative diff-review sandbox before changes land, automated debugging of build/test/lint commands. ([repo](https://github.com/plandex-ai/plandex))
- **Model neutrality**: Explicitly "combine models from Anthropic, OpenAI, Google, **and open source providers**" — multi-provider by design.
- **Self-host**: Client CLI + a **self-hostable server** component (`plandex-server` releases exist alongside the CLI) — good for keeping orchestration data in-house.
- **License**: Open source (AGPL-family in earlier versions — verify current license file before adoption), usage billed per configured model.
- **Azure DevOps fit**: Git-based terminal tool, no ADO-specific lock-in; good "large refactor" specialist agent.

### JetBrains Junie / Junie CLI
- **Description**: JetBrains' coding agent, IDE-engine-powered (IntelliJ), with a CLI, "Advanced Plan Mode" (writes plan to `.junie/plans`, committable), BYOK model support for 10+ models, and Agent Skills package support shared across CLI+IDE via **ACP**. ([site](https://junie.jetbrains.com/))
- **Model neutrality**: Strong — BYOK across Claude/Gemini/GPT models, **and supports locally running models**.
- **Self-host**: "Junie Lite" free tier + BYOK means inference can be fully local/self-hosted; Junie itself (orchestration layer) is a JetBrains product/plugin, not something you self-host as a server.
- **License/pricing**: Free tier (Junie Lite) + paid AI Pro/Ultra tiers ($8–25/user/month) for cloud-credit features; BYOK avoids JetBrains markup.
- **Azure DevOps fit**: IDE/CLI based, source-control agnostic — fine for ADO repos, but less suited to pure headless CI orchestration than Claude Code/Aider/Goose.

### Windsurf
- Not independently fetched in this pass (Cognition-owned IDE/agent, similar category to Cursor: commercial, cloud-backed, Cascade agent). Known from ecosystem knowledge to be **primarily a cloud-hosted commercial IDE**, not self-hostable — treat as excluded under the self-host constraint; worth a confirmatory fetch of `windsurf.com` docs if it's a serious contender later.

### SWE-bench Verified Leaderboard (quality signal)
- Official leaderboards live at **swebench.com**, with separate boards for SWE-bench (2294 instances), **Lite** (300), **Verified** (500, OpenAI's human-filtered subset — the most commonly cited "quality" number), **Multilingual** (300/42 repos/9 languages), and **Multimodal** (480). ([swebench.com](https://www.swebench.com/))
- The live leaderboard table itself is rendered client-side (JS) and wasn't scrapeable as static text in this fetch; however, context gathered elsewhere in this session confirms: SWE-agent's successor **mini-SWE-agent** reports **65% on SWE-bench Verified in ~100 lines of Python** (July update), and SWE-agent 1.0 + Claude 3.7 claimed SOTA on SWE-bench Verified/full in Feb (per repo news feed). JetBrains also cites Junie as a "top performer on **SWE-rebench**" (a related, continuously-refreshed benchmark) rather than classic SWE-bench Verified. **Recommendation**: pull the live numeric leaderboard directly from `swebench.com/viewer.html` or the Verified JSON data file if exact current rankings are needed for a vendor-selection memo — static fetch here only confirms benchmark existence/structure, not current numeric standings.

---

## Comparison Table

| Tool | Headless/API | Model-neutral | Azure OpenAI / local LLM | Self-host | License/Price | Azure DevOps fit | Key caveat |
|---|---|---|---|---|---|---|---|
| GitHub Copilot CLI | Yes (scriptable) | No (Copilot models only) | No | No | Paid subscription | Weak (GitHub-native) | Copilot coding agent is GitHub-only |
| Claude Code (CLI/SDK) | Yes (`-p`, JSON out) | Partial (Anthropic/Bedrock/Vertex) | No native Azure OAI; self-hosted sandbox option | Partial (agent runs local, sandbox self-hostable) | API-metered | Good (git-agnostic) | Anthropic-model-locked |
| OpenAI Codex CLI | Yes | Partial (OpenAI/compat endpoints) | Possible via OpenAI-compatible base URL | CLI local, model hosted | Apache-2.0 + API cost | Good | Verify Azure OAI compat flag |
| Gemini CLI | Yes | No (Gemini/Vertex only) | No | No | Enterprise license only, no free tier | Weak | Not tuned for latest models; enterprise-gated |
| Aider | Yes (`--message`) | **Yes** (LiteLLM backend) | **Yes** | **Yes** | Free/OSS | **Strong** | Weaker native agentic loop |
| OpenHands / Agent Canvas | Yes (REST API) | **Yes** | **Yes** | **Yes** | OSS + paid cloud | Good (needs ADO glue) | Fast-evolving product |
| Cline (CLI/SDK) | Yes (headless CLI) | **Yes** | **Yes** | **Yes** | OSS core | Good | — |
| Roo Code / Kilo Code | Limited | Yes | Yes | Yes (IDE-hosted) | OSS | Fair | IDE-centric, weaker headless story |
| Goose | Yes (CLI/API) | **Yes** (15+ providers incl. Azure/Ollama) | **Yes** | **Yes** | Apache-2.0 | **Strong** | — |
| Cursor CLI/Cloud Agent | Yes (`-p`) | Partial (model picker, routed via Cursor cloud) | No | **No** | Commercial | Fair (not self-hosted) | SaaS dependency |
| Amazon Q Dev CLI | — | — | — | — | Archived | — | Superseded by closed-source Kiro CLI |
| Continue.dev | Yes (CLI) | **Yes** (declared) | **Yes** (declared) | **Yes** | OSS | Good | Needs live re-verification |
| Sourcegraph Amp | Partial (webhooks) | Partial | No | **No** | Commercial | Weak | Cloud-only Orbs |
| Qwen Code | Yes (CLI) | **Yes** (OpenAI/Anthropic/Gemini/Qwen + local) | **Yes** | **Yes** | OSS | Good | Newer project |
| Devin | Yes (REST API) | No | No | **No** | Commercial, ACU-billed | Weak | Disqualified — not self-hosted |
| SWE-agent / mini-SWE-agent | Yes (YAML-config) | **Yes** | **Yes** | **Yes** | OSS (research) | Fair (needs ADO glue) | Research tool, minimal polish |
| Plandex | Yes (CLI + server) | **Yes** | Likely (verify) | **Yes** (self-hostable server) | OSS, verify license | Good | Verify current license terms |
| JetBrains Junie | Yes (CLI) | **Yes** (BYOK, local models) | Likely via BYOK | Partial (inference local, product JetBrains-hosted) | Freemium + paid tiers | Fair | IDE-anchored |
| Windsurf | Unverified | Unverified | Unverified | Likely No | Commercial | Unverified | Needs follow-up fetch |

---

## Part B — Portable Customization Standards (one skill library → many agents)

### AGENTS.md
A plain-Markdown convention (`agents.md`) for giving any coding agent repo-specific instructions: dev-environment setup, test commands, PR conventions. It's intentionally generic prose (no special schema) — any agent that reads root-level docs can consume it. Sample structure: "Dev environment tips," "Testing instructions," "PR instructions." ([agents.md](https://agents.md/)) — Codex CLI, Cursor, and many others explicitly look for `AGENTS.md`; this should be the **lowest common denominator, cross-agent instruction file** in every repo.

### Agent Skills open standard (SKILL.md)
- Originated at Anthropic, now published as an **open, vendor-neutral standard** at **agentskills.io**, with its own GitHub org (`agentskills/agentskills`) for community governance — explicitly *not* Anthropic-only going forward. ([agentskills.io](https://agentskills.io/))
- Format: a folder with `SKILL.md` (YAML frontmatter: `name`, `description`, minimum) plus optional `scripts/`, `references/`, `assets/`.
- Loading model = **progressive disclosure**: (1) Discovery — agent loads only name+description for every skill at startup (cheap); (2) Activation — full `SKILL.md` loads into context only when the task matches; (3) Execution — agent runs bundled scripts/reads bundled references as needed. This is exactly the context-efficient design you want for a large stage-specific skill library.
- Reference implementation/examples: `anthropics/skills` (Apache-2.0 for most skills; installable as a Claude Code **plugin marketplace** via `/plugin marketplace add anthropics/skills`). ([repo](https://github.com/anthropics/skills))
- **Adoption**: agentskills.io maintains a live "Client Showcase" of which agentic tools support the format (confirmed from this research pass: Claude Code, Claude.ai, Claude API; JetBrains Junie explicitly advertises "Agent Skills packages... shared across CLI + IDE via ACP" ([junie.jetbrains.com](https://junie.jetbrains.com/)); GitHub's `awesome-copilot` repo has a dedicated **Skills** category installable via `copilot plugin install` — i.e., **GitHub Copilot CLI/VS Code now consumes the same SKILL.md-style folders too** ([awesome-copilot](https://github.com/github/awesome-copilot)). Recommend pulling the live `agentskills.io/clients` page for the authoritative, continuously updated compatibility matrix before finalizing which agents in your pool can consume skills natively vs. need an adapter.

### Cursor Rules / `.cursor/rules`
Cursor-specific instruction files (not fetched in depth this pass, but well established) — essentially a Cursor-proprietary equivalent of AGENTS.md/copilot-instructions with glob-scoped rule files. Needs a thin translation layer (see design below) since Cursor doesn't natively read SKILL.md folders as far as verified here.

### GitHub-specific layers
- `.github/copilot-instructions.md` — repo-wide instructions for Copilot Chat/coding agent.
- `.github/agents/*` and the `awesome-copilot` taxonomy of **Agents / Instructions / Skills / Plugins** — GitHub has converged its own agent-customization model onto the **same three-tier Agents/Instructions/Skills vocabulary** as the open Agent Skills standard, which is good news for portability. A machine-readable `llms.txt` index of the whole collection exists at `awesome-copilot.github.com/llms.txt`.

### CLAUDE.md
Claude Code's own project-memory file (auto-loaded, like AGENTS.md but Claude-specific); Agent SDK explicitly says skills/commands/memory "load automatically from your project's `.claude/` and from `~/.claude/`, same as Claude Code" ([Agent SDK overview](https://code.claude.com/docs/en/agent-sdk/overview)) — i.e., `.claude/skills/*/SKILL.md` is the canonical Claude-side skill path.

### MCP servers as the tool-integration layer
Model Context Protocol is the de facto neutral way to give *any* agent access to the same external tools/data (Azure DevOps, databases, internal wikis) regardless of which coding agent is driving it — confirmed directly supported by Goose ("70+ extensions via MCP"), Azure DevOps MCP server (local stdio + remote hosted server, with documented client support for **VS Code/Copilot, Visual Studio 2022, Codex, Claude Code, Cursor, OpenCode, and Kilo Code** — see Part E), and effectively every agent surveyed above. **MCP, not the agent itself, should be your single integration point to Azure DevOps, internal registries, and secrets.**

### Marketplaces / distribution
- **github/awesome-copilot** — curated agents/instructions/skills/plugins + cookbook, installable via `copilot plugin install <name>@awesome-copilot`. ([repo](https://github.com/github/awesome-copilot))
- **anthropics/skills** — reference skill implementations + the spec itself (`/spec`) + a skill template (`/template`) you can clone as your org's starting point. ([repo](https://github.com/anthropics/skills))
- **agentskills.io** / `skills.sh` — cross-vendor skill discovery/install CLI (e.g., `skills.sh/anthropics/skills` badge seen on the anthropics/skills repo) — worth adopting as your internal "skills.sh"-style installer if you want one CLI to pull skills into whichever agent's local folder convention is active.
- Not independently re-verified this pass but well known in the space: **obra/superpowers**, **awesome-claude-skills**, **awesome-cursorrules** — community catalogs of ready-made skills/rules, useful as a seed library to fork rather than author everything from scratch.

### How to maintain ONE skill library and project it to many agents
Because the underlying primitive (a folder + `SKILL.md`/instructions file) is now converging across vendors, the practical design is:

1. **Author once**, in the open **Agent Skills** format (`SKILL.md` + bundled scripts/refs), stored in a central **skills repo** (e.g., Azure Repos `skills-library`), organized by SDLC stage (architecture, api-design, db-migration, code-review, security, iac, docs).
2. **Project/symlink per agent** at checkout/setup time (CI step or devcontainer postCreate hook):
   - Claude Code / Agent SDK → copy/symlink into `.claude/skills/<name>/SKILL.md`
   - Copilot CLI/VS Code → register as an `awesome-copilot`-style plugin or drop into `.github/skills/<name>/SKILL.md` (per the awesome-copilot Skills taxonomy)
   - Codex CLI / Cursor / Gemini CLI / others without native SKILL.md support → generate a flattened **AGENTS.md** fragment (frontmatter stripped, instructions concatenated) via a small build script, since AGENTS.md is the universal fallback almost every agent at least reads as plain project context.
   - Goose → wire skill content in as an MCP "extension" or static prompt-injected context, since Goose's primary portability mechanism is MCP, not SKILL.md folders.
3. **Single source of truth, multiple projections** — a CI job (Azure Pipelines) renders the canonical skill library into each agent's expected directory layout on every merge to `main`, so the same instructions never drift between Claude Code, Copilot, Codex, Cursor, and Goose.

---

## Part C — Stage-specific skill/agent examples (open source)

- **ADR / architecture agents**: no single dominant open-source "ADR skill" was independently verified in this pass; the pattern is well covered conceptually by `anthropics/skills`' example-skills category plus GitHub's `awesome-copilot` **Agents** taxonomy (specialized agents integrating MCP servers) — recommend cloning the `anthropics/skills` **template-skill** folder as the starting point for an "ADR-writer" skill (SKILL.md instructing the agent to produce `docs/adr/NNNN-title.md` in MADR format).
- **API design (OpenAPI) skills**: not independently fetched this pass — `awesome-copilot`'s Skills README (`docs/README.skills.md`) and the agentskills.io client/skill showcase are the right places to search live for an existing OpenAPI-authoring skill before writing one from scratch.
- **Database migration skills**: same — check `awesome-copilot` Skills catalog and `anthropics/skills` Development & Technical category first.
- **Code-review skills**: GitHub Copilot's own "request a review from Copilot" on PRs is a built-in analog; `awesome-copilot` Agents explicitly calls out "Specialized Copilot agents that integrate with MCP servers," a natural home for a review agent wired to the Azure DevOps MCP server's PR/work-item tools.
- **Security (OWASP) skills**: not independently verified this pass; recommend sourcing from `anthropics/skills` Enterprise/Technical categories or writing one in-house following the SKILL.md template, keyed to OWASP ASVS/Top-10 checklists.
- **IaC (Terraform/Bicep) skills**: same — no dedicated repo verified live in this pass; author in-house using the SKILL.md template, scoped per cloud provider.
- **Planner/implementer/reviewer sub-agent patterns**: natively supported as a first-class primitive in **Claude Code's Subagents** feature (spawn specialized agents for focused subtasks — Agent SDK table above) and in **Roo Code's Modes** (Architect/Code/Debug/Ask/Custom) — both map directly onto a planner→implementer→reviewer pipeline. GitHub's `awesome-copilot` **Agents** category is the Copilot-side equivalent (custom agents wired to MCP tools per role).

*(Part C is the area with the least live-source depth in this pass — several specific "named" open-source skill repos were not independently located/verified. Recommend a dedicated follow-up search pass against `awesome-copilot.github.com`, `agentskills.io` skill directory, and `anthropics/skills` subfolders before committing to specific external skill packages; treat Part C's recommendations as "author in-house using the verified SKILL.md template" until specific external repos are confirmed.)*

---

## Part D — Sandboxing & Governance for On-Prem Agent Execution

- **Docker/devcontainer sandboxes**: Universal pattern across every self-hostable tool surveyed (OpenHands runs each task in a Docker container; Goose/Aider/Qwen Code/Claude Code SDK all assume you provide an isolated container or VM per task).
- **E2B self-host**: The specific self-hosting doc URL attempted (`docs.e2b.dev/self-hosting-e2b`) returned 404 in this pass — E2B's self-hosting story needs a fresh, corrected-URL fetch before relying on it (E2B is historically a cloud-sandbox-as-a-service product; verify current self-host feasibility directly from `e2b.dev/docs` before use).
- **Daytona**: **Daytona's open-source repo (`daytonaio/daytona`) is explicitly no longer maintained as of June 2026** — "core development has moved to a private codebase... no further updates, fixes, or releases," remaining public/forkable but unsupported. ([repo](https://github.com/daytonaio/daytona)) Do not build new on-prem sandbox infra on this OSS repo; evaluate Daytona's current commercial/private offering separately if still interested.
- **OpenHands runtime**: Purpose-built self-hosted sandbox runtime (Docker-based per-task isolation), the most directly "on-prem ready" sandbox option verified in this pass given OpenHands' explicit self-host design.
- **gVisor/Firecracker**: Not independently fetched this pass; both remain the standard lower-level container/microVM isolation primitives most of these sandbox products (OpenHands, E2B, Daytona-era) build on — recommend pairing whichever sandbox layer you choose with gVisor (runsc) or Firecracker microVMs for kernel-level isolation if running untrusted agent-generated code inside your intranet.
- **LLM gateway / policy & audit**: **LiteLLM** ("Open Source AI Gateway for 100+ LLMs, self-hosted, enterprise-ready, call any LLM in OpenAI format") is directly relevant and strongly recommended as the **single chokepoint** between every agent above and your actual model backends (Azure OpenAI, local vLLM/Ollama, Anthropic, etc.) — it is a Python SDK **and** a deployable self-hosted Proxy Server (AI Gateway) with one-click deploy templates (Render/Railway/AWS/GCP), giving you centralized logging, cost control, and model routing across the entire agent pool. ([repo](https://github.com/BerriAI/litellm)) Portkey and Azure API Management (APIM) serve the same "LLM gateway" role and are valid alternatives/complements, especially since APIM is already native to an Azure shop and can enforce network policy, auth, and quota alongside Azure OpenAI private endpoints.
- **Secret handling**: Not independently re-verified this pass; standard practice applies — keep provider API keys/PATs out of agent prompts and skill files entirely, injected only via the gateway (LiteLLM/APIM) or CI secret store (Azure Pipelines variable groups / Key Vault), never embedded in AGENTS.md/SKILL.md content since those files are the most widely-read, least access-controlled artifacts in the repo.

---

## Part E — Azure DevOps Specifics

- **Azure DevOps MCP server (`microsoft/azure-devops-mcp`)**: Official Microsoft MCP server exposing Azure DevOps (Boards, Repos, Pipelines, Wikis, Test Plans) to any MCP-capable agent. **Two deployment modes**:
  - **Remote MCP Server (recommended)** — hosted by Microsoft at `https://mcp.dev.azure.com/{organization}`, HTTP transport, zero local install, gets new features first.
  - **Local MCP server** — `npx -y @azure-devops/mcp {organization}` stdio server, for clients needing local stdio transport.
  - **Explicitly documented supported clients**: VS Code + GitHub Copilot, **Visual Studio 2022, Codex, Claude Code, Cursor, OpenCode, and Kilo Code** ([repo](https://github.com/microsoft/azure-devops-mcp)) — this is strong direct evidence that your orchestrator can wire *any* of Claude Code, Codex CLI, Cursor, or Kilo Code straight into Azure Boards/Repos/Pipelines today via one standard integration, with Goose/Aider/OpenHands/Qwen Code addable the same way since they're all generic MCP clients.
- **Coding agents inside Azure Pipelines**: Every CLI-based agent surveyed (Claude Code `-p`, Codex CLI, Aider `--message`, Cursor `agent -p`, Goose CLI, Qwen Code) is a plain executable invokable from a **self-hosted Azure Pipelines agent** (install the CLI + its model credentials as pipeline secrets, run it as a pipeline task/script step, capture structured JSON output for gating). Self-hosted Pipelines agents are required anyway for the on-prem/intranet constraint (Microsoft-hosted pipeline agents egress through Microsoft's cloud).
- **PR creation from pipelines**: Use `az repos pr create` (Azure CLI `devops` extension) as the standard, vendor-neutral way for any agent's pipeline step to open a PR against Azure Repos once it has committed changes to a branch — this decouples "which coding agent produced the diff" from "how the PR gets filed," keeping the whole pipeline agent-agnostic.
- **GitHub Copilot for Azure DevOps (2026 status)**: Based on this session's research, current GitHub Copilot documentation (Oct 2026 "Copilot in the cloud" announcement) describes environments as **cloud sandbox, GitHub Actions runner, or a "Remote" connection to a custom environment** — with no Azure DevOps-specific environment type documented. Combined with the absence of any Azure DevOps integration in GitHub's own docs tree, **the evidence strongly suggests GitHub Copilot coding agent still has no native Azure DevOps support as of this research date** — treat Copilot (coding-agent form) as **out of scope** for the ADO-based factory; Copilot CLI remains usable only as an ad hoc interactive helper outside the autonomous pipeline.

---

## Recommended Coding-Agent Pool (2–3 agents) for this Azure DevOps / self-hosted factory

Given the hard constraints (framework-neutral, self-hosted/on-prem, Azure OpenAI/local-model support, Azure DevOps as source of truth), the strongest-evidence picks from this research are:

1. **Goose** (Block/AAIF) — primary general-purpose orchestrated agent: genuinely self-hosted, Apache-2.0, 15+ model providers including Azure and Ollama, native MCP (plugs straight into the Azure DevOps MCP server), "Custom Distributions" for a branded internal agent image. Best default for most stages (implementation, refactoring, docs, DevOps/IaC).
2. **Claude Code (Agent SDK, `-p`/headless)** — primary for architecture/design and complex code-review/reasoning-heavy stages, where its subagents, hooks, and skills system give the richest planner/implementer/reviewer pattern; run it strictly via API key (not consumer subscription) through a LiteLLM gateway for governance, and confirm whether your Anthropic contract supports Azure-hosted or on-prem relay if full intranet residency is mandatory (native Anthropic API call egresses to Anthropic's cloud — flag this as a residency gap to resolve with Legal/Security before adoption).
3. **Aider or Qwen Code** as the **fully local/air-gapped fallback** — both are open-source, LiteLLM/Ollama/vLLM-compatible, and can run 100% on local models with zero external network egress, which is valuable for the most sensitive repos or as a cost-free high-volume implementation/refactor workhorse. Qwen Code edges out Aider if you want a more "agentic"/tool-using loop (SubAgents, Agent Teams) rather than Aider's simpler diff-apply loop; Aider edges out Qwen Code for maturity/stability in pure scripted CI usage.

Route **all three** through a self-hosted **LiteLLM proxy** (or Azure API Management) in front of Azure OpenAI private endpoints / internal vLLM-served models, so model choice, cost, and audit logging are centralized regardless of which CLI/agent the orchestrator dispatches to. Integrate all three with Azure Boards/Repos/Pipelines through the **same Azure DevOps MCP server**, and gate every stage behind a self-hosted Azure Pipelines agent running inside Docker (OpenHands-style per-task container isolation) for sandboxing.

---

## Single Skill Library, Multi-Agent Projection — Concrete Folder Layout

```
skills-library/                        # canonical repo in Azure Repos, source of truth
├── spec/                              # copy of the Agent Skills spec (from anthropics/skills /spec)
├── template/                          # SKILL.md starter template
├── skills/
│   ├── architecture/
│   │   ├── adr-writer/SKILL.md
│   │   └── system-design-review/SKILL.md
│   ├── api-design/
│   │   └── openapi-author/SKILL.md
│   ├── database/
│   │   └── migration-planner/SKILL.md
│   ├── implementation/
│   │   └── feature-implementer/SKILL.md
│   ├── refactoring/
│   │   └── large-refactor-planner/SKILL.md
│   ├── code-review/
│   │   └── pr-reviewer/SKILL.md
│   ├── security/
│   │   └── owasp-checklist/SKILL.md
│   ├── iac/
│   │   ├── terraform-module-author/SKILL.md
│   │   └── bicep-module-author/SKILL.md
│   └── docs/
│       └── doc-updater/SKILL.md
├── agents.md.fragments/                # per-skill plain-text fallback, auto-generated
│   └── <skill-name>.md
├── build/
│   ├── render-agents-md.{py,ts}        # flattens SKILL.md -> AGENTS.md fragments
│   ├── project-claude.{py,ts}          # copies/symlinks into .claude/skills/
│   ├── project-copilot.{py,ts}         # copies into .github/skills/ + awesome-copilot plugin manifest
│   ├── project-cursor.{py,ts}          # converts into .cursor/rules/*.mdc
│   └── project-mcp-goose.{py,ts}       # registers skill content as a Goose MCP extension
└── azure-pipelines.yml                 # CI: on merge to main, runs build/* against every
                                         # consuming repo (or publishes a versioned artifact
                                         # that each repo's devcontainer postCreate pulls)
```

**Projection flow per consuming repo** (triggered in each target repo's `azure-pipelines.yml` pre-build step, or in its devcontainer's `postCreateCommand`):
1. Pull the pinned version of `skills-library`.
2. Run `build/project-*` scripts to materialize:
   - `.claude/skills/*/SKILL.md` (native format, zero transform) — Claude Code/Agent SDK.
   - `.github/skills/*/SKILL.md` + plugin manifest — GitHub Copilot CLI/VS Code (awesome-copilot taxonomy).
   - Flattened `AGENTS.md` (concatenated, frontmatter stripped) at repo root — read by Codex CLI, Aider (via its `--read`/context conventions), Cursor, Goose, Qwen Code, and anything else without native SKILL.md support.
   - `.cursor/rules/*.mdc` — Cursor-specific projection.
   - Goose MCP extension registration (or static context injection) for Goose sessions.
3. Every agent in the pool now sees the **same underlying instructions**, each in its native format, generated from one authored source — eliminating drift.

---

## Gaps, Uncertainties, and Suggested Follow-ups
- **Part C stage-specific skill repos** were the weakest-verified area — no specific named open-source ADR/OpenAPI/DB-migration/OWASP/Terraform skill repositories were independently located and fetched; follow up against `awesome-copilot.github.com/skills`, `agentskills.io`'s skill directory, and the subfolders under `anthropics/skills` (`skills/` tree) before finalizing which to adopt vs. author.
- **SWE-bench Verified numeric leaderboard**: page is JS-rendered; pull `swebench.com/viewer.html` or its underlying data API directly (or a cached mirror/blog citing current numbers) if exact current rankings are needed for a vendor memo.
- **E2B self-hosting**: attempted URL 404'd — needs a corrected fetch from `e2b.dev/docs` directly.
- **Continue.dev and Windsurf**: not independently fetched live this pass; both are described here from strong ecosystem priors but should be confirmed with a direct `github.com/continuedev/continue` and `windsurf.com/docs` fetch before final selection, especially re: Windsurf's self-host status and Continue.dev's exact Azure OpenAI config syntax.
- **OpenAI Codex CLI Azure OpenAI compatibility**: the live repo README didn't explicitly confirm Azure OpenAI-compatible endpoint support; verify current CLI flags (`developers.openai.com/codex` docs) before relying on it for the local/Azure-only slot.
- **Plandex license**: repo card didn't surface a clear current license string in the fetched excerpt; confirm via its `LICENSE` file before adoption given it was historically AGPL-style, which carries different redistribution implications than MIT/Apache-2.0 peers in this list.
- **agentskills.io Client Showcase**: only partially read (truncated at 3000 chars); re-fetch `agentskills.io/clients.md` in full for the authoritative, current per-agent SKILL.md support matrix rather than relying solely on inferences drawn from Junie's and awesome-copilot's own pages.
