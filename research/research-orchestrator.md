# AI Software Factory — Orchestrator & Multi-Agent Framework Research Report

---

## Summary

I verified live GitHub metadata (stars, license, `pushed_at` activity) via the GitHub API and shields.io badges as of **2026‑11‑07**, plus official docs/websites for products without public source. Key high-level findings: **Microsoft Agent Framework (MAF)** has fully superseded AutoGen (now archived); **LangGraph** and **CrewAI** remain the most mature framework-neutral orchestration libraries; **MCP** and **A2A** are the dominant interoperability protocols and both self-hostable; **n8n/Dify/Flowise/Mastra** are all "fair-code"/source-available (not pure OSS) with self-host editions; **GitHub Copilot coding agent** and **Cognition Devin** are cloud-only (data leaves intranet) which conflicts with the on-prem constraint; **GPT-Pilot had a serious supply-chain malware incident** (Aug 2025–Jun 2026) and is now unmaintained — disqualifying; **Azure DevOps MCP server** (`microsoft/azure-devops-mcp`) is the key integration glue for the Azure DevOps constraint.

---

# A) Open-Source Multi-Agent Orchestration Frameworks

### Microsoft Agent Framework (MAF) — successor to AutoGen + Semantic Kernel
- **One-line**: Microsoft's unified, production-grade multi-language (Python/.NET, samples in Go) framework for building and orchestrating AI agents and multi-agent *workflows*, merging AutoGen's multi-agent patterns with Semantic Kernel's enterprise plumbing.
- **Maturity**: 14,568★, MIT-ish (repo shows standard OSS license), very active — `pushed_at: 2026‑11‑06` (daily commits). Created April 2025, still pre-1.0-to-1.0 transition messaging but marketed as "enterprise-grade... stable APIs... long-term support."
- **Self-host**: Yes — runs locally/on-prem; ships Python/.NET packages (PyPI `agent-framework`, NuGet `Microsoft.Agents.AI`).
- **LLM neutrality**: Explicitly supports Microsoft Foundry, Azure OpenAI, OpenAI, and "GitHub Copilot SDK," with cross-runtime interop "via A2A and MCP" — i.e., not locked to Azure; Ollama/local-model support is available through OpenAI-compatible connectors.
- **Fit for orchestrator**: Strong — native MCP + A2A support makes it framework-neutral by design (can delegate to Claude Code/Codex/Cursor as external MCP/A2A agents), enterprise workflow engine with durable/stateful execution, human-in-the-loop checkpoints, Azure-friendly without being Azure-exclusive.
- **Caveats**: Young project (relaunched April 2025); some enterprise features (observability, governance) are deepest when paired with Azure AI Foundry; .NET/Python only (no first-class Go/Java SDK yet, unlike ADK).
- URL: https://github.com/microsoft/agent-framework , docs: https://learn.microsoft.com/en-us/agent-framework/

### AutoGen (Microsoft) — ⚠️ ARCHIVED
- **One-line**: Original multi-agent conversation framework (AgentChat + AutoGen Studio); historically influential.
- **Maturity**: Repository banner explicitly states **"Project archived and no longer maintained"** — Microsoft is not providing bug/security fixes. Users directed to migrate to MAF via an official migration guide.
- **Self-host/neutrality**: Was self-hostable and provider-agnostic, now moot given archived status.
- **Fit**: **Not recommended** for new builds — include only as historical/compatibility note; any existing AutoGen agents should migrate to MAF.
- URL: https://github.com/microsoft/autogen

### LangGraph (LangChain)
- **One-line**: Low-level, graph-based orchestration framework for building durable, stateful, long-running agents/workflows.
- **Maturity**: 44,588★, 7,682 forks, **MIT license**, very active (`pushed_at: 2026‑11‑06`). Used in production by Klarna, Replit, Elastic per README. First released 2023.
- **Self-host**: Fully self-hostable (pure Python/JS library); LangGraph Platform (hosted deployment option) is separate/optional — the core library needs no cloud dependency.
- **LLM neutrality**: Fully provider-agnostic via LangChain's model abstraction — works with Azure OpenAI, OpenAI, Anthropic, and local models through Ollama/vLLM-compatible connectors (LangChain has native Ollama + OpenAI-compatible-endpoint integrations).
- **Fit for orchestrator**: Excellent as the **workflow/state-machine engine** — durable execution, interrupts/human-in-the-loop, persistent memory, LangSmith tracing (optional, can be self-hosted or disabled). It is a low-level *library*, not a packaged SDLC product, so an orchestrator built on it requires custom role/agent design (PM, Design, Coding, Test, Release nodes as graph nodes).
- **Caveats**: Low-level — you build the SDLC orchestration logic yourself (no built-in "coding agent" or ADO connector); best combined with MCP tool servers (incl. azure-devops-mcp) for tool access.
- URL: https://github.com/langchain-ai/langgraph

### CrewAI
- **One-line**: High-level "crew of role-playing agents" framework for multi-agent task orchestration (Crews = autonomous collaboration, Flows = deterministic control).
- **Maturity**: 60,591★, 9,081 forks, **MIT license**, extremely active (`pushed_at: 2026‑11‑07`, same day). Standalone (no LangChain dependency), Python.
- **Self-host**: Fully self-hostable; optional "Cloud Trial"/enterprise offering (crewai.com) is separate from the OSS core.
- **LLM neutrality**: Provider-agnostic — supports OpenAI, Azure OpenAI, Anthropic, and local models via LiteLLM/Ollama integration out of the box.
- **Fit for orchestrator**: Good fit for a **role-based department model** (PM agent, Architect agent, Coder agent, Tester agent, Release agent as "Crew" roles) — maps naturally onto the department/SDLC concept in the prompt. Simpler mental model than LangGraph but less fine-grained state control.
- **Caveats**: Less durable-execution/workflow rigor than LangGraph or Temporal; best for logical task delegation rather than long-running, crash-safe pipelines.
- URL: https://github.com/crewAIInc/crewAI

### Agno (formerly Phidata)
- **One-line**: "Self-managing agent platform" — AgentOS runtime providing API/MCP server, auth, RBAC, human approvals, scheduling, durable execution for agents/teams/workflows.
- **Maturity**: 43,086★, Apache‑2.0, very active (`pushed_at: 2026‑11‑06`). Pivoted recently toward a platform ("Company OS") rather than pure library.
- **Self-host**: Yes — explicit templates for Docker, Railway, AWS; "Run your platform in your own cloud and keep your data in your control."
- **LLM neutrality**: Multi-model by design; built-in MCP server and REST API, works with "Claude and Codex engineers" alongside its own agent SDK — i.e. framework-neutral at the platform layer.
- **Fit for orchestrator**: Interesting as a **runtime/ops layer** (AgentOS) rather than orchestration logic — could host the ORCHESTRATOR's API/MCP surface and durable execution, but its SDLC-specific tooling is thin; marketing has shifted toward generic "company operations" agents, not SDLC-specific.
- **Caveats**: Rapid pivoting/rebranding (Phidata→Agno→AgentOS/Company OS) suggests API instability; smaller enterprise track record than LangGraph/CrewAI.
- URL: https://github.com/agno-agi/agno

### OpenAI Agents SDK
- **One-line**: Lightweight multi-agent workflow framework (agents, handoffs, tools, guardrails, tracing) built by OpenAI but explicitly "provider-agnostic."
- **Maturity**: 30,535★, very active (`pushed_at: 2026‑11‑07`). Created March 2025.
- **Self-host**: Yes, pure Python library — no required cloud dependency beyond whichever model endpoint you configure.
- **LLM neutrality**: README states explicit support for "the OpenAI Responses and Chat Completions APIs, as well as **100+ other LLMs**" (via LiteLLM-style model adapters) — can point at Azure OpenAI or local OpenAI-compatible servers (vLLM, Ollama) using the Chat Completions-compatible interface.
- **Fit for orchestrator**: Good lightweight option; "Agents as tools"/"Handoffs" model maps to department hand-offs (PM→Design→Coding→Test→Release). Includes sandboxed/long-horizon "sandbox agents" concept relevant to coding tasks.
- **Caveats**: Younger, OpenAI-centric community/ecosystem despite neutrality claims; built-in tracing UI defaults to OpenAI's platform (can be disabled/self-hosted alternatives exist via OpenTelemetry export).
- URL: https://github.com/openai/openai-agents-python

### Google Agent Development Kit (ADK)
- **One-line**: Code-first Python (+ Java/Kotlin/Go variants) framework for building, evaluating and deploying agents, model-agnostic despite Google origin.
- **Maturity**: 22,127★, **Apache‑2.0**, very active (`pushed_at: 2026‑11‑07`, essentially live). "ADK 2.0" branding indicates a major version milestone.
- **Self-host**: Fully self-hostable library; deployable to Vertex AI Agent Engine optionally, but not required.
- **LLM neutrality**: Explicitly model-agnostic (works with Gemini, but also LiteLLM wrapping for Azure OpenAI, Anthropic, Ollama/vLLM local models per ADK docs).
- **Fit for orchestrator**: Strong multi-language story (Java/Kotlin/Go SDKs exist too) is attractive for mixed enterprise stacks; good agent evaluation tooling.
- **Caveats**: Google-centric samples/tutorials bias toward Gemini/Vertex; smaller mind-share for pure-Azure/on-prem shops than MAF/LangGraph.
- URL: https://github.com/google/adk-python , docs: https://adk.dev/

### Temporal (durable workflow engine)
- **One-line**: General-purpose durable-execution platform (not AI-specific) increasingly used as the reliability backbone under agent orchestration (retries, long-running state, exactly-once activity execution).
- **Maturity**: Mature, originated from Uber's Cadence; backed by Temporal Technologies (commercial company + OSS core). MIT-style OSS server (verify per-component; Temporal Cloud is the commercial SaaS).
- **Self-host**: Yes — `temporal server start-dev` / self-managed clusters are first-class; this is one of the most mature fully self-hostable durable execution systems available.
- **LLM neutrality**: Fully neutral — it's infrastructure, not an agent framework; any agent framework (LangGraph, custom Python/Go workers) can run its steps as Temporal Activities/Workflows for crash-safe, auditable SDLC pipelines (e.g., a multi-day "feature build" workflow surviving restarts).
- **Fit for orchestrator**: Excellent as the **durable backbone/state-machine layer under the ORCHESTRATOR**, not as the agent-reasoning layer itself — pairs well with MAF/LangGraph/CrewAI for long SDLC pipelines that must survive crashes/days-long human-approval waits. Several blog posts/OSS examples show "Temporal + LangGraph/AutoGen agent" patterns.
- **Caveats**: Steeper ops overhead (need to run/maintain a Temporal cluster); not AI-specific, so agent reasoning logic still needs one of the frameworks above.
- URL: https://github.com/temporalio/temporal , https://temporal.io/

### n8n
- **One-line**: Low-code visual workflow automation platform with native AI agent/tool nodes, MCP client/server support.
- **Maturity**: 209,719★, 61,387 forks, **extremely active** (`pushed_at: 2026‑11‑07`). License: **"Sustainable Use License"** (source-available, NOT OSI-approved) — free for internal business use/self-hosting, but restricts reselling n8n itself as a competing service; some enterprise ("`.ee.`") files require a commercial license.
- **Self-host**: Yes, explicitly "self-host or cloud" — one of the most common fully on-prem no/low-code automation tools; Docker-first.
- **LLM neutrality**: Native nodes for OpenAI, Anthropic, Azure OpenAI, and local models (Ollama) plus generic HTTP/MCP nodes — fully provider-agnostic.
- **Fit for orchestrator**: Good as a **glue/automation layer** triggering and sequencing calls to external coding agents (via MCP/HTTP), webhook-driven ADO pipeline triggers, human-approval forms, Slack/Teams notifications — less suited as the "brain" doing agentic reasoning/coding itself.
- **Caveats**: Source-available license, not permissive OSS — fine for internal use but legal review recommended if the factory itself might be redistributed/resold.
- URL: https://github.com/n8n-io/n8n

### Dify
- **One-line**: Visual LLM app/agent/RAG-workflow builder with a Studio UI, positioned as "from prototype to production."
- **Maturity**: 160,581★, 25,408 forks, very active (`pushed_at: 2026‑11‑07`). License: **modified Apache 2.0** ("Open Source License" with added restrictions) — verified via raw LICENSE file: free to self-host and use commercially, **except** operating a **multi-tenant SaaS** version of Dify itself requires a commercial license from the vendor; also restricts removing Dify's frontend branding.
- **Self-host**: Yes, explicit "Deploy on cloud, VPC, or self-hosted."
- **LLM neutrality**: Supports many model providers incl. Azure OpenAI and self-hosted/local models (Ollama, Xinference, LocalAI) via its model-provider plugin system.
- **Fit for orchestrator**: Reasonable for a **non-coder-friendly agent/RAG workflow layer** (e.g., PM/requirements chat agent, design-doc RAG agent) but not purpose-built for SDLC/CI orchestration or ADO integration; would need custom plugins for Azure Repos/Boards.
- **Caveats**: License restriction only matters if you'd resell the platform as multi-tenant SaaS; for a single internal factory, restriction is largely moot. Heavier full-stack deployment (Postgres/Redis/Weaviate etc.) than a pure library.
- URL: https://github.com/langgenius/dify

### Flowise
- **One-line**: Low-code visual agent/LLM-flow builder ("Build AI Agents, Visually"), similar niche to Dify/n8n.
- **Maturity**: 55,561★, 25,091 forks. **⚠️ Repository shows `"archived": true`** in the GitHub API as of Nov 2026, and last push was **2026‑08‑13** (stale ~3 months vs. peers' same-day activity) — this is a meaningful maturity red flag that should be independently reconfirmed before adoption (possible pause, reorg, or renamed successor repo).
- **License**: "Other"/custom (not pure MIT/Apache) — verify terms before commercial self-hosted use.
- **Self-host**: Historically yes (Docker/npm), but given the archived flag, treat as **uncertain/at risk**.
- **Fit for orchestrator**: **Not recommended** right now given the archived status and stale activity until clarified.
- URL: https://github.com/FlowiseAI/Flowise

### Mastra
- **One-line**: TypeScript-first framework for AI agents/workflows (positioned as "the Vercel/Next.js of agents" for JS/TS shops).
- **Maturity**: 29,384★, very active (`pushed_at: 2026‑11‑07`). License: core is **Apache License 2.0** (verified via raw LICENSE.md); only code under `ee/` directories is separately licensed (enterprise features) — core framework is genuinely open.
- **Self-host**: Yes, fully self-hostable Node/TS library; no server dependency required for core use.
- **LLM neutrality**: Explicit multi-provider model routing (OpenAI, Anthropic, Azure OpenAI, and local/self-hosted models via the Vercel AI SDK-style provider abstraction it's built on).
- **Fit for orchestrator**: Best option if the org's dominant stack is **TypeScript/Node** rather than Python/.NET — otherwise a secondary choice vs. LangGraph/MAF for a Python/.NET-centric enterprise.
- **Caveats**: Younger ecosystem, smaller enterprise track record than LangGraph/CrewAI; workflow/durable-execution features newer/less battle-tested.
- URL: https://github.com/mastra-ai/mastra

### PydanticAI
- **One-line**: Python agent framework from the Pydantic team emphasizing type-safety, "every model, every interface, typed end to end."
- **Maturity**: 21,064★, **MIT license**, very active (`pushed_at: 2026‑11‑07`).
- **Self-host**: Yes, pure Python library.
- **LLM neutrality**: Explicitly "every model" — native support for OpenAI, Azure OpenAI, Anthropic, Google, Groq, Mistral, and OpenAI-compatible local endpoints (Ollama/vLLM) via its model interface.
- **Fit for orchestrator**: Good for teams that want strong typed I/O contracts between agent stages (e.g., strictly-typed PM requirement → Design spec → Code diff handoffs), which aids reliability in a multi-department pipeline. Less turnkey for multi-agent "crew" patterns than CrewAI; more of a single/few-agent, tool-calling-focused library requiring custom orchestration for 5-stage SDLC flow.
- URL: https://github.com/pydantic/pydantic-ai

### smolagents (Hugging Face)
- **One-line**: "Barebones library for agents that think in code" — minimalist code-executing agent framework.
- **Maturity**: 30,241★, very active (`pushed_at: 2026‑11‑03`). Created Dec 2024.
- **Self-host**: Yes, trivial pure-Python library, tiny footprint (size 7,429 vs. 300k+ for CrewAI/Agno — genuinely "smol").
- **LLM neutrality**: Explicit multi-model support incl. local models via Hugging Face Inference/Transformers and Ollama-compatible endpoints.
- **Fit for orchestrator**: Too minimalist to be the ORCHESTRATOR itself for a 5-stage SDLC factory — better suited as the implementation of an individual specialized agent (e.g., a lightweight "code agent" worker) than the top-level coordinator.
- URL: https://github.com/huggingface/smolagents

### Model Context Protocol (MCP)
- **One-line**: Open protocol (originated at Anthropic, now an independent spec org) standardizing how LLM apps/agents connect to tools, data sources, and context — the dominant "USB-C for AI agents" standard in 2025–2026.
- **Maturity**: MIT-licensed spec repo; extremely widely adopted — GitHub Copilot, Claude Code, Cursor, VS Code, n8n, MAF, ADK, OpenAI Agents SDK all support MCP natively or via connectors (confirmed from n8n docs above and framework READMEs).
- **Self-host**: Protocol itself is free/self-hostable; the **official Azure DevOps MCP server** (below) is the key enabler for this scenario.
- **Fit**: **Essential interoperability glue** — any framework-neutral orchestrator should standardize tool access (Azure Repos, Boards, Pipelines, file systems, internal wikis) behind MCP servers so that whichever coding agent (Copilot/Claude Code/Codex/Cursor) is plugged in can use the same tool surface without bespoke integration per agent.
- URL: https://github.com/modelcontextprotocol/modelcontextprotocol , https://modelcontextprotocol.io

### Agent2Agent (A2A) Protocol
- **One-line**: Open protocol (originated at Google, now `a2aproject` org with IBM/partners) for opaque agent-to-agent communication/discovery/task collaboration across frameworks and vendors.
- **Maturity**: Apache‑2.0 licensed; has a DeepLearning.AI course co-taught with Google Cloud/IBM Research — signals genuine cross-vendor backing, not just a Google side-project. MAF README explicitly cites A2A + MCP as its interoperability story.
- **Self-host**: Yes — it's a protocol/SDK (`pip install a2a-sdk`), self-hostable agent servers.
- **Fit**: Complements MCP — MCP standardizes **agent↔tool** access, A2A standardizes **agent↔agent** handoffs/negotiation. For a multi-department factory where PM/Design/Coding/Test/Release agents may run on *different* frameworks (one team's coding agent is Claude Code, another's is Codex), A2A lets the ORCHESTRATOR treat each department agent as an opaque, discoverable peer regardless of its internal framework — directly matching the "framework-neutral" requirement.
- URL: https://github.com/a2aproject/A2A

### Agent Skills (agentskills.io)
- **One-line**: Open, lightweight format (`SKILL.md` + bundled scripts/assets) for portable, progressively-disclosed agent capabilities/procedural knowledge — originated at Anthropic, released as an open standard, adopted by "a growing number of agent products" (per official site) including GitHub Copilot.
- **Maturity**: Spec hosted at https://github.com/agentskills/agentskills ; adoption tracked via a public "Client Showcase."
- **Self-host**: N/A (it's a file-format standard, not a service) — skills are just version-controlled folders, trivially kept on-prem/in Azure Repos.
- **Fit**: Directly useful for the factory: standardize department-specific procedural knowledge (e.g., "how our org writes ADO work items," "our coding standards," "our release checklist") as portable Skills folders that **any** compliant coding agent (Copilot, Claude Code, others as adoption grows) can load — reduces per-agent prompt engineering duplication across the framework-neutral agent roster.
- **Caveat**: Standard is new and adoption is still growing beyond Anthropic-family tools; verify each target coding agent's actual SKILL.md support before relying on it as the sole knowledge-portability mechanism.
- URL: https://agentskills.io/ , spec repo: https://github.com/agentskills/agentskills

---

# B) "AI Software Factory" Products / Reference Projects

### MetaGPT (now FoundationAgents/MetaGPT)
- **One-line**: Multi-agent framework that assigns GPT-style agents distinct SOP roles (PM, Architect, Engineer, QA) to simulate a software company from a one-line requirement — the closest direct precedent to the user's "department" concept.
- **Maturity**: ~71k★ (shields.io), **MIT license**, repo renamed/moved to `FoundationAgents/MetaGPT` (redirect from `geekan/MetaGPT`) — still active with recent ICLR 2025 paper and a commercial spinoff product "MGX" (mgx.dev).
- **Self-host**: Yes, open-source Python, runs locally/on-prem against any OpenAI-compatible endpoint.
- **LLM neutrality**: Config-driven model selection; supports OpenAI-compatible endpoints broadly (works with Azure OpenAI and local model gateways in practice, per community docs).
- **Fit**: Excellent **conceptual reference architecture** for "PM→Architect→Engineer→QA" role SOPs and multi-agent "software company" simulation — but it is a research-grade project/example, not an Azure DevOps-integrated product; would need significant engineering to bolt onto Azure Repos/Boards/Pipelines.
- **Caveats**: Commercial energy has shifted toward the closed MGX SaaS product; core OSS repo's day-to-day SDLC-production hardening is less proven than purpose-built coding-agent products.
- URL: https://github.com/FoundationAgents/MetaGPT

### ChatDev
- **One-line**: Originally a "virtual software company" multi-agent SDLC simulator (CEO/CTO/Programmer roles); has pivoted in Jan 2026 to **ChatDev 2.0 "DevAll"**, a general-purpose zero-code multi-agent orchestration platform (classic SDLC-focused version now legacy on a `chatdev1.0` branch).
- **Maturity**: ~35k★, active, backed by OpenBMB/Tsinghua-affiliated research group; NeurIPS 2025 paper on "evolving orchestration" with a learnable central orchestrator.
- **Self-host**: Yes, OSS.
- **Fit**: Legacy v1.0 branch is directly on-topic (full SDLC agent company) but is now explicitly "legacy/maintenance only" — the actively developed v2.0 has broadened scope away from pure SDLC. Use mainly as a **research reference** for multi-agent SDLC "seminar"-style collaboration patterns, not as a production orchestrator.
- URL: https://github.com/OpenBMB/ChatDev

### OpenHands (formerly OpenDevin, now rebranded "Agent Canvas" under org `OpenHands/OpenHands`)
- **One-line**: Self-hosted "developer control center for coding agents and automations" — can run OpenHands' own agent, **Claude Code, Codex, Gemini, OpenCode, or any ACP-compatible agent** across local/remote/cloud backends.
- **Maturity**: ~93k★, **MIT license**, extremely active ("last commit: today"). Rebrand from "OpenDevin"→"OpenHands" is well established; now further evolving toward a multi-agent-backend "Agent Canvas" product.
- **Self-host**: Explicitly "self-hosted," core design goal.
- **LLM neutrality**: Very strong — explicitly backend-agnostic across multiple vendor coding agents (not just LLM-neutral but *coding-agent*-neutral), which directly matches the "Copilot is just one of several coding agents" requirement.
- **Fit for orchestrator**: **Strong candidate**, arguably the best public match to "framework-neutral coding-agent control center" in the whole list — it doesn't replace PM/Design/Test/Release orchestration, but as the **coding-agent execution/sandboxing layer** underneath a higher-level ORCHESTRATOR, it's uniquely positioned (self-hosted + multi-agent-backend + ACP protocol support).
- **Caveats**: Scope is "run/manage coding agents," not full-SDLC (PM/requirements, design, test, release) orchestration — would be one component (the "Coding" department's execution backend) rather than the top-level factory orchestrator. Needs an Azure DevOps/Repos connector evaluation (GitHub-centric historically; verify ADO repo support before committing).
- URL: https://github.com/OpenHands/OpenHands (redirects from `All-Hands-AI/OpenHands`)

### GPT-Pilot — ⚠️ SECURITY INCIDENT, DO NOT USE AS-IS
- **One-line**: Early "AI developer" tool (Pythagora) that scaffolds and builds apps turn-by-turn with a human in the loop.
- **Maturity**: ~34k★, but the repo now carries a prominent **security notice**: a malicious commit (`065ee8eb`, disguised as a revert) inserted a credential-stealing supply-chain worm (`core/telemetry/_hooks.py`, Shai-Hulud-class) that was live from **August 2025 until 11 June 2026** before being publicly reported and removed. The repo is explicitly flagged **"no longer actively maintained."**
- **Fit**: **Disqualified** for any serious evaluation — both because it's unmaintained and because of the credential-theft history; even cleaned, trust should be considered damaged until a maintained fork/successor proves otherwise.
- URL: https://github.com/Pythagora-io/gpt-pilot

### Devika
- **One-line**: Open-source "agentic AI software engineer," modeled explicitly after Cognition's Devin, with Claude 3/GPT-4/Gemini/Mistral/**local LLM via Ollama** support.
- **Maturity**: ~20k★, repo banner says **"This project is currently in a very early development/experimental stage... a lot of unimplemented/broken features"** and points users to a newer rewrite called **Opcode** ("the second iteration of Devika, new version out soon").
- **Self-host/neutrality**: Yes, and explicit native Ollama support is notable for the on-prem/local-model requirement.
- **Fit**: Interesting for the **local-model story** but explicitly experimental/early-stage per its own README, and actively being superseded by "Opcode" — track Opcode instead if pursuing this lineage; not production-ready today.
- URL: https://github.com/stitionai/devika

### Factory.ai (Droids)
- **One-line**: Commercial "agent-native software development platform" — Droids (CLI, desktop, web, mobile, Slack/Teams/Jira/Linear/CI integrations) for planning, building, reviewing, testing, shipping.
- **Maturity**: Commercial SaaS/enterprise product (not OSS); has a documented public REST API and OpenAPI spec.
- **Self-host**: Not open-source; "enterprise deployment controls" are mentioned but this is a hosted/managed product — verify directly with Factory.ai whether a genuinely on-prem/air-gapped deployment option exists (not evidenced in the fetched docs) before assuming it satisfies the self-hosted constraint. **No Azure DevOps-specific integration mentioned** in the fetched overview (Jira/Linear/CI yes; ADO not listed).
- **Fit**: Likely **not suitable** for a strict on-prem/no-data-leaves-intranet requirement unless their enterprise tier offers private/VPC deployment — needs vendor confirmation.
- URL: https://www.factory.ai/ (redirects to factory.com)

### Cognition Devin
- **One-line**: Commercial autonomous software engineer agent ("runs in the cloud or on your machine, tests in its own browser").
- **Maturity**: Well-funded commercial product with major enterprise customers listed (Citi, Nvidia, Cloudflare, etc.).
- **Self-host**: Marketing explicitly says "runs in the cloud **or on your machine**" — suggests a local/agent-runner option exists, but core product is cloud-hosted with cloud-based orchestration; not open-source, not framework-neutral (it's Cognition's own proprietary agent).
- **Fit**: Not a fit as the **orchestrator** (it's a single vendor's proprietary coding agent, the opposite of framework-neutral) — could at most be **one more coding-agent spoke** plugged into a neutral orchestrator via A2A/MCP if Cognition exposes such interfaces (not confirmed from the fetched page).
- URL: https://devin.ai/

### GitHub Copilot coding agent (and "Copilot in the cloud")
- **One-line**: GitHub's autonomous coding agent that plans/implements/reviews changes, now unified into a continuous "Copilot in the cloud" session spanning GitHub.com, Mobile, IDEs, CLI.
- **Maturity**: Actively developed, rolling out Oct 28, 2026 onward per fetched docs.
- **Self-host**: **Important caveat** — Copilot coding agent's cloud sessions run in **GitHub-hosted "cloud sandboxes"** (isolated Linux environments GitHub creates and manages); the docs explicitly state "Copilot clones each [repo] into the sandbox" and "the session doesn't run on your machine." This is **cloud-hosted by GitHub**, not self-hosted/on-prem by default, which conflicts with the "data must not leave the intranet" constraint unless the org is comfortable with GitHub-hosted (not Azure DevOps-hosted) execution for this specific agent. A separate "local sandboxing" mode exists (session stays on the user's machine with OS-level restrictions) which is more aligned with on-prem — but that's a different, more manual mode than the full cloud agent experience.
- **Fit**: Use as **one spoke** (a coding agent a developer explicitly invokes), but **not** as the factory's data-handling backbone if intranet-only data residency is a hard requirement; GitHub Copilot's native ADO integration is also more limited than its GitHub-native integration (verify current state — Copilot's PR/Issue automation is built around GitHub Issues/PRs, not Azure Boards/Repos, so Azure DevOps shops typically rely on the Azure DevOps MCP server + a generic coding agent rather than GitHub Copilot's native repo automation).
- URL: https://docs.github.com/en/copilot/concepts/security-governance-and-network-settings/about-cloud-sandboxes

### Amazon Kiro
- **One-line**: Agentic IDE/CLI product ("Kiro Crew" — persistent agents, spec-driven "Workflows" that split tasks into verified checked steps) from AWS.
- **Maturity**: Active commercial product with enterprise testimonials (SmugMug, Iress, Rackspace).
- **LLM neutrality**: Notably **model-neutral by design** — explicitly supports "Claude, GPT, and open-weight models like DeepSeek and Qwen, or Auto-routing."
- **Self-host**: Not evidenced as self-hostable/on-prem from the fetched page (appears to be a cloud-backed product, "local machine, or in the cloud" execution but the control plane/billing is AWS-hosted); verify data-residency terms directly with AWS before use in a strict intranet-only scenario.
- **Fit**: Interesting for its **spec-driven "Workflows"** concept (a checked-step SDLC pattern worth emulating architecturally) but not itself the on-prem orchestrator; AWS-centric, not Azure DevOps-native.
- URL: https://kiro.dev/

### Google Jules / Google Antigravity
- **Jules**: Async, multi-agent coding assistant product, tiered plans — cloud SaaS, no self-host evidenced. URL: https://jules.google/
- **Antigravity**: Google's newer "agentic development platform" — Antigravity App/CLI/IDE/**SDK** (the SDK explicitly lets you "prototype custom agents leveraging Antigravity's harness... Simple Python scripts," suggesting some local/custom extensibility), multi-local-agent management ("command center to manage multiple local agents in parallel"). Still appears to be a Google-hosted product family rather than an on-prem-deployable orchestrator.
- **Fit**: Both are **not self-hostable orchestrators** for this scenario — interesting as competitive/UX references (Antigravity's "multiple local agents in parallel" command-center concept is architecturally relevant) but not adoptable given the on-prem/Azure DevOps constraints.
- URL: https://antigravity.google/

### Microsoft Agent 365 / Azure AI Foundry (Microsoft Foundry) Agent Service
- **One-line**: Microsoft's **managed cloud platform** (now rebranded "Microsoft Foundry Agent Service") for building/deploying/scaling agents, "build with any framework, any supported model," content filters, observability, tool calling.
- **Maturity**: Actively maintained first-party Microsoft Learn doc, updated Oct 2026; "Microsoft Agent 365" branded product page was not found at the expected URL (404) — it may have been consolidated into/renamed to "Microsoft Foundry," or is marketed under a different path; **treat "Agent 365" as not independently confirmed** and verify current Microsoft naming directly before citing it to stakeholders.
- **Self-host**: This is a **managed Azure cloud service**, not self-hosted/on-prem by nature — conflicts with "data must not leave the intranet" unless deployed via Azure-in-your-tenant with private networking (Azure private endpoints), which is an Azure-hybrid option worth exploring with Microsoft directly but is not "self-hosted" in the open-source sense.
- **Fit**: Could be relevant as **one Azure-hosted agent runtime option within a hybrid deployment** (private-endpoint-secured Azure tenant counts as "intranet-adjacent" for some compliance regimes) but is not framework-neutral in spirit (it's Microsoft's managed runtime) — best viewed as a possible execution backend for Azure-OpenAI-model-based agents, not the cross-vendor orchestrator itself.
- URL: https://learn.microsoft.com/en-us/azure/ai-foundry/agents/overview (canonical redirect: `.../azure/foundry/agents/overview`)

### Atlassian Rovo
- **One-line**: Atlassian's AI layer across Jira/Confluence/Loom — "Unlock human+AI teamwork."
- **Maturity**: Commercial, bundled into Atlassian's cloud suite.
- **Fit**: **Not relevant** to this scenario — Rovo is Jira/Confluence-centric; the user's stack is Azure DevOps (Boards/Repos/Pipelines), not Jira. Only relevant if the org also runs Jira in parallel; otherwise skip.
- URL: https://www.atlassian.com/software/rovo

### Spec-driven orchestration approaches
- **BMAD-METHOD**: "Agile AI-Driven Development" methodology/toolkit — ~55k★ (shields), **MIT license**, actively developed (v6.12 stable, v7 alpha in progress, ETA mid-Nov 2026). Framework-neutral by design ("carry its briefs, specifications, and architecture into your existing delivery workflow") — a **methodology + tool** for turning ideas into specs/architecture/build tasks that **any** coding agent can then execute. Strong conceptual fit for the PM/Design phases of the factory. URL: https://github.com/bmad-code-org/BMAD-METHOD
- **GitHub Spec Kit**: GitHub's own open-source "spec-driven development" toolkit — structured processes/templates for turning a spec through planning→implementation→convergence, agent-agnostic (works with "your coding agent," not GitHub Copilot specifically). Extremely high star count per shields badge (verify directly on github.com given API rate-limiting prevented a second cross-check) reflecting its viral adoption as the reference spec-driven-dev tool in 2025–2026. URL: https://github.com/github/spec-kit
- **OpenSpec** (Fission-AI): Competing spec-framework tool, MIT license, "fluid not rigid... built for brownfield not just greenfield," npm-distributed CLI. URL: https://github.com/Fission-AI/OpenSpec
- **Kiro specs**: Amazon Kiro's "Workflows" feature (see above) is a proprietary analog of the same spec-driven, checked-step idea, not independently extractable/self-hostable outside Kiro.
- **Fit**: These spec-driven tools (BMAD, Spec Kit, OpenSpec) are **excellent complements, not replacements**, for the orchestrator — they standardize the **PM/Design artifact format** (specs, plans, architecture docs) that the ORCHESTRATOR hands down to whichever coding agent executes the "Coding" stage. Because they're agent-agnostic markdown/template conventions, they fit the framework-neutral requirement perfectly and can live as files inside Azure Repos.

---

# C) Azure DevOps Integration Options

### Official Azure DevOps MCP Server (`microsoft/azure-devops-mcp`)
- **One-line**: Microsoft's official MCP server exposing Azure DevOps (Repos, Boards, Pipelines, Wikis, Test Plans) to any MCP-compatible AI agent via a thin layer over the ADO REST APIs.
- **Maturity**: ~2.1k★ (shields), actively maintained by Microsoft. Two deployment modes documented: a **Remote MCP Server** (hosted, "no installation," gets new features first — this is a Microsoft-hosted option, verify its data-flow/region settings for intranet-only requirements) and a **Local MCP Server** (self-hosted, `stdio` connection — fully on-prem, "remains supported" though "new development will focus" on the remote variant).
- **Self-host**: **Yes, via the Local MCP Server** — this is the correct choice for the "data must not leave the intranet" constraint; the Remote MCP Server should likely be avoided or carefully vetted since it's Microsoft-hosted infrastructure.
- **Fit**: **Critical integration component** — this is the natural "Azure DevOps tool adapter" for *any* MCP-capable orchestrator/agent in this architecture (MAF, LangGraph+MCP client, OpenHands, CrewAI with MCP tool wrapper, Claude Code, Cursor, etc. can all consume the same ADO tool surface).
- URL: https://github.com/microsoft/azure-devops-mcp

### Azure DevOps REST API / Python SDK
- **One-line**: `azure-devops` Python package and the underlying REST APIs for direct programmatic access to Repos, Boards (work items), Pipelines, Test Plans — the fallback/complement to the MCP server for custom orchestration code (e.g., Temporal Activities, LangGraph tool nodes) that needs typed, low-level access beyond what the MCP tool surface exposes.
- **Fit**: Use for **custom glue code** (e.g., a release-department agent that needs fine-grained Pipeline run status, or a PM agent creating structured Epic/Feature/PBI hierarchies in Boards) where the MCP tool abstraction is too coarse.

### GitHub Copilot + Azure DevOps
- **Finding**: GitHub Copilot's native coding-agent automation (PR creation, review requests, issue-driven kickoff) is built around **GitHub Issues/PRs**, not Azure Boards/Repos. There is no first-party "GitHub Copilot for Azure DevOps" product evidenced in the fetched docs — Azure DevOps shops wanting Copilot-style agent behavior against ADO repos would route through: (a) the Azure DevOps MCP server feeding a framework-neutral agent (MAF/LangGraph/OpenHands) that happens to use Copilot's underlying model via Azure OpenAI/Copilot SDK, or (b) GitHub Copilot used only against a GitHub mirror, which reintroduces the cross-repo-sync problem the user is presumably trying to avoid by standardizing on Azure Repos.
- **Recommendation**: Treat GitHub Copilot as "one of several coding agents" invoked **through** the MCP/A2A-based orchestrator against Azure Repos via the Local Azure DevOps MCP Server, rather than relying on any native GitHub↔ADO product bridge.

---

# Comparison Table

| Item | Stars | License | Last Activity | Self-Host | LLM-Neutral | SDLC/ADO Fit |
|---|---|---|---|---|---|---|
| Microsoft Agent Framework | 14.6k | OSS (MIT-style) | 2026-11-06 | Yes | Yes (MCP/A2A native) | High |
| AutoGen | — | OSS | **Archived** | Yes | Yes | None (migrate to MAF) |
| LangGraph | 44.6k | MIT | 2026-11-06 | Yes | Yes | High (low-level) |
| CrewAI | 60.6k | MIT | 2026-11-07 | Yes | Yes | High |
| Agno | 43.1k | Apache-2.0 | 2026-11-06 | Yes | Yes | Medium (platform pivot) |
| OpenAI Agents SDK | 30.5k | OSS | 2026-11-07 | Yes | Yes (100+ LLMs) | Medium |
| Google ADK | 22.1k | Apache-2.0 | 2026-11-07 | Yes | Yes | Medium |
| Temporal | mature | OSS core | active | Yes | N/A (infra) | High (as backbone) |
| n8n | 209.7k | Sustainable Use (source-available) | 2026-11-07 | Yes | Yes | Medium (glue layer) |
| Dify | 160.6k | Modified Apache-2.0 | 2026-11-07 | Yes | Yes | Low-Medium |
| Flowise | 55.6k | Custom/"Other" | **⚠️ 2026-08-13, archived flag** | Uncertain | Yes | Low (verify status) |
| Mastra | 29.4k | Apache-2.0 (core) | 2026-11-07 | Yes | Yes | Medium (TS shops) |
| PydanticAI | 21.1k | MIT | 2026-11-07 | Yes | Yes | Medium |
| smolagents | 30.2k | OSS | 2026-11-03 | Yes | Yes | Low (too minimal for top-level) |
| MCP | — | MIT | active | Yes | N/A (protocol) | Essential glue |
| A2A | — | Apache-2.0 | active | Yes | N/A (protocol) | Essential glue |
| Agent Skills | — | Open standard | active | N/A | N/A | Useful (knowledge portability) |
| MetaGPT | ~71k | MIT | active | Yes | Yes | High concept, low ADO integration |
| ChatDev | ~35k | OSS | active (v2 pivot) | Yes | Yes | Medium (v1 legacy) |
| OpenHands | ~93k | MIT | today | Yes | Yes (multi-agent-backend) | High (coding-execution layer) |
| GPT-Pilot | ~34k | OSS | **⚠️ unmaintained, malware history** | Yes | Yes | Disqualified |
| Devika | ~20k | OSS | **early/experimental** | Yes | Yes (incl. Ollama) | Low (use "Opcode" successor) |
| BMAD-METHOD | ~55k | MIT | active | N/A (methodology) | Yes | High (spec/PM layer) |
| GitHub Spec Kit | very high | OSS | active | N/A | Yes | High (spec/PM layer) |
| OpenSpec | ~74k | MIT | active | N/A | Yes | High (spec/PM layer) |
| Factory.ai | — | Commercial | active | Unclear | Yes | Low (data residency unclear) |
| Cognition Devin | — | Commercial | active | Partial | No (proprietary) | Low (not neutral) |
| GitHub Copilot coding agent | — | Commercial | active | **No (GitHub cloud sandbox)** | No | Low (data residency conflict) |
| Amazon Kiro | — | Commercial | active | Unclear | Yes | Medium (spec concept only) |
| Google Jules/Antigravity | — | Commercial | active | No | Partial | Low |
| Azure AI Foundry Agent Service | — | Commercial (Azure) | active | No (managed) | Yes (any model) | Medium (Azure-hybrid only) |
| Atlassian Rovo | — | Commercial | active | No | N/A | Not relevant (Jira-centric) |
| Azure DevOps MCP Server | 2.1k | OSS | active | **Yes (local mode)** | N/A (tool adapter) | **Critical** |

---

# Top 3 Recommended Orchestrator Options

**1. Microsoft Agent Framework (MAF) as the ORCHESTRATOR core, with Temporal as the durable-execution backbone and the local Azure DevOps MCP Server + MCP/A2A as the tool/agent-interop layer.**
Reasoning: MAF is purpose-built as the enterprise successor to both AutoGen and Semantic Kernel, is fully self-hostable in Python/.NET, explicitly supports Azure OpenAI *and* non-Microsoft models/agents via native MCP and A2A, and is the only option in this list designed from day one for "multi-agent workflows" at enterprise scale with Microsoft's own long-term support commitment. Pairing it with Temporal for crash-safe, long-running SDLC pipelines (a feature build can span days awaiting human approval) hardens the one area MAF alone doesn't fully own (industrial-grade durable execution). The local Azure DevOps MCP server gives every department agent — regardless of whether it's Copilot, Claude Code, Codex, or a custom LangGraph node — uniform, on-prem access to Repos/Boards/Pipelines.

**2. LangGraph + CrewAI (hybrid) as the ORCHESTRATOR, using LangGraph for the durable state-machine/pipeline (PM→Design→Coding→Test→Release) and CrewAI-style role agents for department-level task delegation, glued together with MCP (incl. Azure DevOps MCP) and A2A for external coding-agent handoff.**
Reasoning: Both are mature (44.6k/60.6k★), MIT-licensed, extremely active, fully self-hostable, and genuinely LLM/vendor-neutral with no corporate platform lock-in — arguably the most "neutral" combination on the list, avoiding both Microsoft's and OpenAI's gravitational pull while still happily consuming Azure OpenAI or local Ollama/vLLM models. The low-level control of LangGraph plus CrewAI's higher-level role abstraction lets you faithfully model "PM/Design/Coding/Test/Release" as explicit graph stages/crew roles. Risk: more assembly required than MAF (no out-of-the-box enterprise governance layer), so plan for added engineering investment in observability, RBAC, and durable execution (bring in Temporal here too if long-running robustness is required).

**3. OpenHands as the Coding-department execution layer underneath a thin custom orchestrator (built on MAF or LangGraph), standardizing PM/Design artifacts via GitHub Spec Kit or BMAD-METHOD, with Azure DevOps MCP as the universal tool adapter.**
Reasoning: OpenHands is uniquely proven (93k★, MIT, daily commits) as a **self-hosted, multi-coding-agent-backend control center** — it already solves "GitHub Copilot is just one of several coding agents" at the execution layer by natively running OpenHands' own agent, Claude Code, Codex, Gemini, and any ACP-compatible agent side-by-side, on-prem. Combine it with a spec-driven methodology (BMAD-METHOD or GitHub Spec Kit — both agent-agnostic, markdown-based, and naturally stored in Azure Repos) to standardize how the PM/Design departments hand off work to whichever coding agent OpenHands dispatches, and wrap the whole thing in a thin MAF or LangGraph coordinator for the PM→Test→Release stages OpenHands doesn't itself own. This option minimizes custom coding-agent integration work (OpenHands has already solved "many backends, one control plane") at the cost of needing to build/verify its Azure DevOps (vs. GitHub) connector depth yourself.

**Why not the others as the primary pick**: Google ADK and OpenAI Agents SDK are solid but carry stronger single-vendor gravitational pull than MAF/LangGraph despite neutrality claims; n8n/Dify/Flowise are good automation glue but not SDLC-reasoning engines and carry non-permissive licenses (n8n, Dify) or an unresolved archived-status flag (Flowise); MetaGPT/ChatDev are valuable conceptual references but research-grade, not production/ADO-ready; GPT-Pilot is disqualified (security incident, unmaintained); and all the fully commercial products (Factory.ai, Devin, Copilot cloud agent, Kiro, Jules/Antigravity, Azure AI Foundry Agent Service, Rovo) either aren't self-hostable, aren't framework-neutral, or explicitly route data through vendor-hosted cloud sandboxes — each directly conflicting with the stated on-prem/intranet-only constraint.

### Gaps / Follow-ups
- Could not independently re-verify `github/spec-kit`'s and `Fission-AI/OpenSpec`'s exact star counts via the GitHub REST API (rate-limited to 403 after initial calls); shields.io badge values (148k and 74k respectively) were used as the best available live source — recommend a quick manual spot-check on github.com before quoting these numbers externally, as they are high even for viral 2025–2026 tools.
- "Microsoft Agent 365" could not be located at its expected Microsoft.com URL (404) — likely renamed/folded into "Microsoft Foundry Agent Service" branding; recommend confirming current official naming directly with a Microsoft rep or the Azure doc site search before citing in a formal deck.
- Factory.ai's and Amazon Kiro's actual on-prem/air-gapped deployment options were not confirmed from public docs — both should be confirmed directly with the vendors (via sales/enterprise contacts) before ruling in/out based on this public-page research alone.
- Did not independently verify whether GitHub Copilot's "local sandboxing" mode (as opposed to the cloud-sandbox mode) could satisfy the intranet-only constraint if Copilot is used strictly in that mode — worth a deeper dive if the org still wants Copilot as one of the pluggable coding agents.
