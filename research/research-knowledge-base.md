# AI Software Factory — Knowledge Base Layer Research Report

> **最終決策備註（2026-11-07）**：本報告為原始調研紀錄，撰寫時以「Azure OpenAI Private Endpoint 或本地模型」與「Azure DevOps Server 地端」為假設前提。後續規劃已定案為：**完全地端 Kubernetes**、**唯一允許的雲端服務為 Azure DevOps Services**、**不使用任何雲端 LLM**（含 Azure OpenAI），模型一律為叢集內 vLLM 或公司內部 endpoint，經 LiteLLM Proxy 路由。閱讀時請以 `AI-Software-Factory-Plan.md` 為準，本文中的 Azure OpenAI / ADO Server 相關內容僅供參考。


**Scope**: Self-hosted/on-prem platforms for PM-authored URDs/specs that must be queryable by downstream AI agents (RAG/MCP), framework-neutral LLM usage (Azure OpenAI / Ollama / vLLM), integrated with Azure DevOps (Repos/Boards/Wiki). All star/activity figures below were gathered live by research subagents (GitHub API / mirrors, Nov 7, 2026); treat them as point-in-time snapshots.

---

## A. Self-Hosted Wiki / Docs Platforms

### 1. Wiki.js — [js.wiki](https://js.wiki) · [github.com/Requarks/wiki](https://github.com/Requarks/wiki)
- **License**: AGPL-3.0. **Maturity**: ⭐29,200, daily commits, only 8 open issues (well-triaged).
- **Self-host**: Docker/Compose/K8s Helm; Postgres/MySQL/SQLite/MSSQL.
- **API**: GraphQL only. **MCP**: community-only, small/experimental (`osianet/wiki-js-mcp`, `grtxx/mcp-wikijs`).
- **Git-backed**: **Yes** — unique optional Git storage/sync mode (pages as Markdown files, bidirectional push/pull) — strong RAG-ingestion story.
- **Diagramming**: Mermaid native; draw.io via embed only.
- **Access control**: Fine-grained page/path rules, AD/LDAP/OAuth/SAML.
- **Fit**: Good classic wiki; git-sync mode is its killer feature for this use case. **Caveat**: single-maintainer bus-factor risk, no first-party AI integration.

### 2. Outline — [getoutline.com](https://www.getoutline.com) · [github.com/outline/outline](https://github.com/outline/outline)
- **License**: Source-available "Outline" license (NOT pure OSS — commercial hosting-as-a-service restricted; verify legal terms for enterprise self-host). **Maturity**: ⭐41,194, very active, well-funded.
- **Self-host**: Docker (Postgres+Redis+S3) — heavier ops than Wiki.js/BookStack.
- **API**: Rich REST JSON API. **MCP**: best community server `mmmeff/outline-mcp-server` (121⭐, updated Oct 2026) + ~9 smaller variants.
- **Git-backed**: No (Postgres DB; clean MD import/export).
- **Diagramming**: Native Mermaid; no native draw.io.
- **Fit**: **Best writing UX** of the OSS set (real-time co-editing, templates, nested docs) — ideal for PM authoring experience. **Caveat**: license needs legal review for enterprise.

### 3. BookStack — [bookstackapp.com](https://www.bookstackapp.com) · [github.com/bookstackapp/bookstack](https://github.com/bookstackapp/bookstack) (mirror; **primary dev moved to Codeberg**)
- **License**: MIT. **Maturity**: ⭐19,131 (GitHub mirror — check Codeberg for live activity).
- **Self-host**: Docker/LAMP, lightweight.
- **API**: REST, in-app Swagger docs. **MCP**: strongest MCP density of any pure wiki — `pnocera/bookstack-mcp-server` (101⭐), `ttpears/bookstack-mcp` (51⭐), plus a Rust pgvector-powered variant.
- **Diagramming**: **Native draw.io/diagrams.net** signature feature + Mermaid.
- **Fit**: Good Books→Chapters→Pages hierarchy for a spec library, simpler/lighter than Outline/Confluence. **Caveat**: Codeberg migration is a governance flag to verify.

### 4. Docmost — [docmost.com](https://docmost.com) · [github.com/docmost/docmost](https://github.com/docmost/docmost)
- **License**: AGPL-3.0 core + proprietary Enterprise Edition (open-core). **Maturity**: ⭐22,249, youngest (2023) but fastest-growing, named on-prem adopters (Bechtle, govt agencies, Red Cross).
- **Self-host**: Docker (Postgres+Redis), explicitly marketed "On-Premises Wiki for Enterprise Teams."
- **API**: REST. **MCP**: **Marketing claims native/official self-hosted AI+MCP support** ("Ask questions grounded in your documentation") + large community ecosystem (`MrMartiniMo/docmost-mcp` 34⭐, RBAC-aware and default-deny-writes variants).
- **Diagramming**: **Draw.io + Excalidraw + Mermaid simultaneously** — broadest native support reviewed.
- **Access control**: Spaces+Groups+RBAC plus a built-in **Page Verification workflow** (review/approve/sign-off) explicitly for ISO 9001/27001/SOC 2 — mirrors URD sign-off processes.
- **Fit**: **Likely strongest overall candidate** — on-prem by design, native AI/MCP, approval workflow matches URD review cycles. **Caveat**: younger/less battle-tested; verify which AI features are AGPL-core vs EE-paid.

### 5. AFFiNE — [affine.pro](https://affine.pro) · [github.com/toeverything/AFFiNE](https://github.com/toeverything/AFFiNE)
- **License**: Mixed MIT + non-OSS AFFiNE license (NOASSERTION — needs legal review). **Maturity**: ⭐74,282 but 802 open issues (high churn).
- **Self-host**: Complex stack (server+web+Redis+Postgres+object storage).
- **MCP**: `DAWNCR0W/affine-mcp-server` (323⭐ — most-starred MCP server found in this entire research).
- **Git-backed**: No (CRDT block model, not Markdown-native).
- **Fit**: Weak-moderate for formal spec writing; better for brainstorming/whiteboarding. Diagram format is proprietary (edgeless canvas), not diagram-as-code.

### 6. AppFlowy — [appflowy.io](https://appflowy.io) · [github.com/AppFlowy-IO/AppFlowy](https://github.com/AppFlowy-IO/AppFlowy)
- **License**: AGPL-3.0. **Maturity**: ⭐78,340 (highest in set) but 1,024 open issues.
- **Self-host**: Heavy stack (GoTrue+MinIO+Redis+Postgres).
- **MCP**: Modest community activity (≤11⭐ each).
- **Fit**: Moderate — blends docs+databases+Kanban, not purpose-built for long-form specs; weak native diagramming.

### 7. XWiki — [xwiki.org](https://xwiki.org) · [github.com/xwiki/xwiki-platform](https://github.com/xwiki/xwiki-platform)
- **License**: LGPL-2.1. **Maturity**: ⭐1,333 (oldest/most enterprise-established codebase, 2011; low stars typical for enterprise Java wikis — continuous commercial-backed dev is itself a maturity signal).
- **Self-host**: Docker/WAR on Tomcat/Jetty, LDAP/AD, clustering.
- **API**: **Full REST + Velocity/Groovy scripting API** — can define structured "XObjects" (typed URD fields: status/owner/priority), more RAG-friendly for structured data than freeform MD.
- **MCP**: Modest community servers (≤5⭐).
- **Access control**: Most sophisticated ACL model reviewed (global/wiki/space/page/document, allow/deny, groups).
- **Fit**: Strong for structured, semi-formal requirement templates via XClass; steep learning curve.

### 8. DokuWiki — [dokuwiki.org](https://www.dokuwiki.org) · [github.com/splitbrain/dokuwiki](https://github.com/splitbrain/dokuwiki)
- **License**: GPL-2.0. **Maturity**: ⭐4,738, slower cadence (last push ~6 weeks prior), but still active.
- **Self-host**: **No database** — plain `.txt` files on disk, trivially git-trackable (`git init` the data folder directly).
- **API**: XML-RPC only (weak); bypassed by reading flat files directly.
- **MCP**: Several small/experimental community servers.
- **Fit**: Best "put it in git directly" story of any option; dated UX, weak native API, no real-time collaboration.

### 9. Confluence Data Center (commercial reference)
- **License**: Commercial (Atlassian). Data Center = the on-prem/clustered edition (Server retired Feb 2024).
- **API**: Extensive REST v1/v2. **MCP**: **No official Atlassian MCP for Data Center** — Atlassian's official "Atlassian Remote MCP Server" targets **Cloud only**; community servers exist for DC/Server (`anuragagrawal0430/atlassian-confluence-mcp-server`, `sooperset/mcp-atlassian`) but unofficial.
- **Storage**: XHTML "Storage Format" — not Markdown, not git-backed — weakest diff/RAG-ingestion story of any tool reviewed.
- **Fit**: Best-in-class authoring UX/ACL/diagramming, but Jira-centric (less native value in an Azure DevOps shop) and has a real AI/MCP gap for the self-hosted edition.

### 10. Azure DevOps Wiki (built-in)
- **License**: Bundled with Azure DevOps (Services or self-hosted **Server** edition — satisfies no-egress natively, zero new infra).
- **Storage**: **Natively IS a Git repo** of Markdown files — the single strongest git-backed-Markdown story of any option; native Mermaid rendering; `#123` auto-links to Boards work items (real traceability win).
- **API**: Full Azure DevOps REST API (same auth as Repos/Boards).
- **MCP**: **Official Microsoft `microsoft/azure-devops-mcp`** (2,083⭐, actively maintained, covers Boards/Repos/Wiki/Pipelines) — the most mature/official MCP integration found in this entire research effort. On-prem/Server(TFS) variant also exists (`burcusipahioglu/azure-devops-mcp-onprem`, 16⭐).
- **Known limits**: Page hierarchy gets unwieldy at scale (hundreds of URDs), search weaker than dedicated wiki tools, coarser (repo/branch-level) ACLs than Wiki.js/XWiki/BookStack.
- **Fit**: **Pragmatic top pick given existing ADO investment** — zero new infrastructure, official first-party MCP, native work-item traceability. The scaling/search/ACL gaps are real but solvable.

### 11. Docs-as-code: MkDocs(+Material) / Docusaurus / Backstage TechDocs
- **MkDocs** (⭐22,539, BSD-2) + **mkdocs-material** (⭐27,603, MIT, very active, 1 open issue): static-site build from pure Markdown in Azure Repos; native Mermaid; no built-in ACL (needs reverse-proxy SSO).
- **Docusaurus** (facebook, ⭐66,665, MIT, very active): same model with MDX; heavier Node/React toolchain.
- **Backstage TechDocs** (⭐34,793, Apache-2.0, CNCF, very active): wraps MkDocs, tied to a full developer-portal/service-catalog platform — only worth the overhead if Backstage is already (or will be) deployed for other reasons; MCP ecosystem essentially nonexistent (one abandoned hobby repo).
- **Common theme**: Markdown **is** the source of truth, no DB/export layer — the practical RAG/MCP integration point is to point a git/filesystem/Azure DevOps MCP connector directly at the Azure Repos source tree, not at the static-site tool itself. Best for technically-comfortable PMs doing PR-reviewed spec changes; poor fit for non-technical PMs wanting WYSIWYG.

### Comparison Table A

| Tool | License | Stars | Self-host | API | MCP available? | Git-backed MD? | Diagramming | Best-fit note |
|---|---|---|---|---|---|---|---|---|
| Wiki.js | AGPL-3.0 | 29,200 | Docker/K8s | GraphQL | Community only | **Yes** (git-sync mode) | Mermaid native | Solid; git-sync is a real RAG asset |
| Outline | Source-available | 41,194 | Docker (heavy) | REST | Community (121⭐ best) | No | Mermaid native | Best writing UX; license review needed |
| BookStack | MIT | 19,131 | Docker/LAMP | REST+Swagger | Strong community (101⭐) | No | **Native draw.io**+Mermaid | Simple library, great diagrams |
| Docmost | AGPL+EE | 22,249 | Docker | REST | **Claimed native** + community | No | Draw.io+Excalidraw+Mermaid | **Likely best overall fit** |
| AFFiNE | Mixed/unclear | 74,282 | Docker (complex) | GraphQL (thin) | Community (323⭐ highest) | No (CRDT) | Proprietary canvas | High buzz, weak spec fit |
| AppFlowy | AGPL-3.0 | 78,340 | Docker (heavy) | REST (thin) | Community (≤11⭐) | No | Weak | Docs+DB+Kanban hybrid |
| XWiki | LGPL-2.1 | 1,333 | Docker/WAR | **Full REST+scripting** | Community (≤5⭐) | No (own syntax) | Draw.io+Mermaid ext. | Best structured-data templates |
| DokuWiki | GPL-2.0 | 4,738 | **No DB** | XML-RPC (weak) | Community, low-star | **Yes** (flat files) | Plugins | Best "git it directly" |
| Confluence DC | Commercial | N/A | Yes (DC) | Full REST | **No official for DC** | No (XHTML) | Excellent | Gold UX/ACL, AI gap |
| **Azure DevOps Wiki** | Bundled | N/A | **Native** | Full REST | **Yes — official MS server (2,083⭐)** | **Yes (is a Git repo)** | Native Mermaid | **Pragmatic top pick** |
| MkDocs+Material | BSD/MIT | 22,539/27,603 | Static, trivial | N/A | None needed | **Yes, pure source** | Native Mermaid | Best for PR-reviewed specs |
| Docusaurus | MIT | 66,665 | Static | N/A | None needed | Yes | Mermaid plugin | Similar, heavier toolchain |
| Backstage TechDocs | Apache-2.0 | 34,793 | Full platform | Catalog API | Minimal (0⭐) | Yes | Mermaid (MkDocs) | Only if Backstage already adopted |

---

## B. Requirements Management & Spec-Driven Development

### B1. Requirements-Management Tools

| Tool | License | Maturity | Fit for URD workflow | Caveat |
|---|---|---|---|---|
| [OpenProject](https://github.com/opf/openproject) | GPL-3.0 | ⭐17,061, daily commits | Configurable work-package types for requirements, built-in wiki, Gantt/roadmap | No native traceability matrix; not spec-authoring; general PM tool |
| [Taiga](https://github.com/taigaio/taiga-back) | MPL-2.0 | 858⭐ (backend repo), active | Good backlog/story mgmt with acceptance criteria | Multi-repo architecture; no requirements/traceability concept |
| [Doorstop](https://github.com/doorstop-dev/doorstop) | LGPLv3 | 693⭐, active (niche safety-critical audience) | **Requirements-as-code**: each requirement = a YAML file, parent/child traceability validated by CLI, multi-format publishing | CLI-only, no web UI/review workflow, niche adoption |
| [StrictDoc](https://github.com/strictdoc-project/strictdoc) | Apache-2.0 | 448⭐, very active | Structured `.sdoc` format (`[REQUIREMENT]` UID/TITLE/STATEMENT + relations), local web server, Git-native/diffable | Smaller community, learning curve, no multi-stakeholder review UI |
| Jama Connect / Polarion (Siemens) | Commercial | Mature enterprise products | **Live, auto-updated traceability matrices**, formal review/approval/e-signature (FDA/DO-178C/ISO 26262), test/defect linkage | Licensing cost, vendor lock-in; no native agent-consumable spec export |
| **Azure DevOps Boards + Wiki** | Bundled | Mature | See critical gap analysis below | — |
| AFFiNE / Docmost / Anytype (Notion-likes) | Mixed | See Section A | Good drafting surfaces, no requirements/traceability schema | Role = authoring surface, not source of truth |

**Critical gap — Azure DevOps Boards + Wiki**: There is **no native "Requirement" work item type** with formal baselining/approval/e-signature, and no built-in traceability matrix view. Wiki pages are **not linked to work items as structured data** — a PM's URD prose has no native decomposition into individually-addressable, machine-parseable requirement IDs. **This is the single most important architectural gap the orchestrator must bridge**: build a transformation/extraction layer (agent or script) that reads Wiki Markdown via the ADO REST API, decomposes it into a structured schema (Doorstop/StrictDoc-like, or a Spec Kit/OpenSpec `specs/` folder), and pushes derived Work Items + links back into Boards for tracking. The ADO REST API fully supports both Work Items and Wiki read/write, making this bridge feasible.

### B2. Spec-Driven Development Toolkits (the critical layer for agent-consumable specs)

| Tool | Output format | License | Maturity | Agent-consumable? | Integration notes |
|---|---|---|---|---|---|
| **GitHub Spec Kit** | Markdown under `.specify/` (constitution→specs→plans→tasks) | MIT | ⭐148,214, created Aug 2025, daily commits | Yes, purpose-built; supports air-gapped/self-hosted catalogs | Plain-file MD → trivial RAG ingestion; GitHub first-party |
| **Amazon Kiro Specs** | 3 files: `requirements.md`, `design.md`, `tasks.md`; dependency-graph "wave" execution | Commercial/closed | Mature product, not OSS | Yes, within Kiro's engine | Design-pattern reference only — not self-hostable |
| **OpenSpec** (Fission-AI) | Plain MD: `proposal.md`, `specs/` (SHALL + Given/When/Then scenarios), `design.md`, `tasks.md` | MIT | ⭐74,176, created Aug 2025, daily commits | Yes, explicitly ("built with OpenSpec") | **"Stores" feature (beta)**: dedicated Git repo holding only specs, decoupled from code — enables **cross-repo canonical requirements** readable by every coding agent in a multi-repo ADO org |
| **BMAD-Method** | Likely MD PRDs/briefs/architecture via role-based agent personas (PM/architect/UX/dev/QA) | MIT | ⭐55,085, created Apr 2025, daily commits | Yes by design (schema mid-migration v6→v7, verify before committing) | Full "AiDD" methodology, heavier-weight than Spec Kit/OpenSpec |
| **Tessl** | Unclear for PRD/URD; current emphasis is "skills" (agent context packages), not classic requirements | Commercial/closed SaaS | Enterprise-adopted (Cisco cited) | Yes for skills; weak fit for this specific use case | Weakest direct fit of the five; possible complement for skill-governance/observability |
| **awesome-copilot PRD agents/skills** | MD PRDs with requirement IDs (e.g. `GH-001`), fixed outline, optional auto-creation of GitHub Issues | MIT (part of `github/awesome-copilot`, ⭐40,882) | Actively maintained | Yes — designed for Copilot agents, direct precedent for an "ADO Boards equivalent" bridge | `prd.agent.md`, `skills/prd/SKILL.md`, `plugins/project-planning/` (`breakdown-epic-pm`, `breakdown-feature-prd`, `create-github-issues-feature-from-implementation-plan`) |

**Synthesis**: OpenSpec and GitHub Spec Kit are the two most mature, agent-consumable, actively developed open frameworks (both MIT, huge, growing, pure-Markdown, Git-native). **OpenSpec's "Stores" concept is particularly relevant** for a multi-repo ADO org wanting one canonical, cross-repo requirements source. The `awesome-copilot` PRD skill set is the most directly reusable *prompt-level* asset for the PM-facing authoring experience, and its issue-creation command is a template for the needed ADO-Boards-sync bridge.

---

## C. Self-Hosted RAG / Enterprise Knowledge-Search Platforms

### Comparison Table C1 — Full RAG Platforms/Apps

| Tool | License | Stars | Self-host | MCP? | Confluence/ADO-Wiki connector? | Local LLM | Fit for agent-queryable KB |
|---|---|---|---|---|---|---|---|
| **[Onyx](https://github.com/onyx-dot-app/onyx)** (fka Danswer) | MIT core / EE source-available | 33,256 | Docker/K8s/Terraform | **Yes — official server (OAuth, Streamable-HTTP) + consumes MCP plugins** | **Confluence: yes (Cloud + Data Center)**; ADO Wiki: no native (use generic Web connector) | Ollama/LM Studio/vLLM via LiteLLM | **Best fit** — purpose-built enterprise search, native MCP both directions |
| **[RAGFlow](https://github.com/infiniflow/ragflow)** | Apache 2.0 | 93,707 | Docker Compose only (**no official Helm/K8s docs**; multi-service: API/Admin/Ingestor/Syncer + MySQL + ES/Infinity + MinIO + Kvrocks + NATS + ClickHouse) | **Official MCP server** (dedicated port 9382); consuming external MCP tools unverified | **Broad native connectors** (Confluence, SharePoint, OneDrive, S3, GitHub/GitLab/**Azure DevOps**, Jira, sitemap, DBs) with incremental sync — ADO *Wiki* coverage unverified | vLLM/Ollama official guides; OpenAI-compatible custom provider; TEI local embeddings | Strongest document understanding (DeepDoc layout/OCR/TSR, CPU-only in 1.0; Knowledge Compilation replaces GraphRAG/RAPTOR since v0.27); built-in agent workflow canvas; Teams in OSS but no SSO/OIDC/fine RBAC found; **v1.0.0 Go rewrite released 2026-11-06** — expect churn. *(Updated 2026-11-07 after dedicated re-verification.)* |
| **[AnythingLLM](https://github.com/Mintplex-Labs/anything-llm)** | MIT | 68,015 | Docker/Desktop | Client only (consumes MCP) | **Confluence: yes**; ADO Wiki: no | Extensive (llama.cpp, Ollama, LM Studio, LocalAI) | Good for small-team chat-with-wiki; weaker governance/scale |
| **[Open WebUI](https://github.com/open-webui/open-webui)** | Custom (BSD-3+branding clause) | 157,930 | Docker/K8s/clouds | Indirect (`mcpo` OpenAPI bridge) | None native (upload-centric) | Ollama-native, any OpenAI-compatible | Best as chat front-end, not connector layer |
| **[LightRAG](https://github.com/HKUDS/LightRAG)** | MIT | 40,584 | Docker (setup wizard) | Community only | None (library; custom ingestion) | Role-based local LLM config | Strong lightweight graph-RAG engine, needs custom glue |
| **[Dify](https://github.com/langgenius/dify)** | Modified Apache 2.0 | 160,493 | Docker Compose | **Yes — official MCP Tool plugin (consumer)** | Datasource plugin marketplace; no confirmed first-party connector | Via Model plugins | Strong agent/workflow platform; pair with external MCP servers |
| **[Kotaemon](https://github.com/Cinnamon/kotaemon)** | Apache 2.0 | 25,867 | Docker | None | None | Ollama, llama-cpp-python | Weak — stale (~4 months), no REST API |
| **[Haystack](https://github.com/deepset-ai/haystack)** (framework) | Apache 2.0 | 26,935 | DIY library | Ecosystem tool-calling | No dedicated connector; has Azure AI Search doc-store | Ollama/llama.cpp/OpenAI-compatible | Flexible, high build effort |
| **[LlamaIndex](https://github.com/run-llama/llama_index)** (framework) | MIT | 52,826 | DIY library | Official MCP tool-spec (consumer) | **Official Confluence Reader (CQL/space/label)**; no ADO Wiki reader | Ollama + OpenAI-compatible | Best framework option — ready Confluence connector |
| **[Microsoft GraphRAG](https://github.com/microsoft/graphrag)** | MIT | 36,582 | CLI/library only | None | None | Manual config | **Not recommended — explicit maintenance-mode warning in README** |
| **Azure AI Search** | Proprietary (Azure PaaS) | n/a | Managed (stays in Azure tenant, not on-prem) | None native | None native | n/a (pairs w/ Azure OpenAI) | Hybrid-search backend only, not a connector platform |

### Key MCP-server building blocks discovered
- **[`sooperset/mcp-atlassian`](https://github.com/sooperset/mcp-atlassian)** (MIT) — most mature community Atlassian MCP server; **Confluence Cloud + Server/Data Center**, Jira too.
- **[`microsoft/azure-devops-mcp`](https://github.com/microsoft/azure-devops-mcp)** (2,081⭐) — official, covers ADO Wikis/Boards/Repos directly.
- **Practical implication**: for an org using **Confluence + Azure DevOps Wiki**, the fastest path to "agent-queryable via MCP" today is **Onyx** (native Confluence connector + native MCP server) combined with **official `microsoft/azure-devops-mcp`** — rather than waiting for a single RAG platform to natively connect to both.

### Comparison Table C2 — Vector DBs

| DB | License | Stars | Self-host | API | Note |
|---|---|---|---|---|---|
| [Qdrant](https://github.com/qdrant/qdrant) | Apache 2.0 | 35,437 | Docker | REST+gRPC | Rust performance leader, rich filtering/quantization |
| [Milvus](https://github.com/milvus-io/milvus) | Apache 2.0 | 46,608 | Docker/K8s-native | gRPC/REST | Highest stars here; billion-scale, real-time streaming |
| [pgvector](https://github.com/pgvector/pgvector) | PostgreSQL License | 23,602 | Postgres extension | SQL | Lowest-ops if Postgres already in stack |
| [Weaviate](https://github.com/weaviate/weaviate) | BSD-3-Clause | 16,940 | Docker/K8s | REST+gRPC | All-in-one hybrid+rerank+multi-tenancy |
| [Chroma](https://github.com/chroma-core/chroma) | Apache 2.0 | 29,647 | Embedded/server | Minimal API | Easiest to prototype |

---

## D. Document-to-Structured-Knowledge Pipelines

### D1. Self-hosted transcription (meeting notes → requirements input)

| Tool | License | Stars | Self-host GPU/CPU | Output | Integration note |
|---|---|---|---|---|---|
| [openai/whisper](https://github.com/openai/whisper) | MIT | ~112k | CPU (slow)/GPU recommended | Raw transcript | Reference engine, no notes/summarization |
| [SYSTRAN/faster-whisper](https://github.com/SYSTRAN/faster-whisper) | MIT | ~27k | CPU-friendly (int8) | Raw transcript w/ timestamps | Backend for most downstream tools |
| [ggerganov/whisper.cpp](https://github.com/ggerganov/whisper.cpp) | MIT | ~55k | Best for CPU-only/edge | Raw transcript | Lightest footprint, zero deps |
| [m-bain/whisperX](https://github.com/m-bain/whisperX) | BSD-2-Clause | ~25k | GPU recommended | **Speaker-diarized, time-aligned transcript** | Richest input for a downstream LLM-notes step |
| [Zackriya-Solutions/meetily](https://github.com/Zackriya-Solutions/meetily) | MIT | ~32k | CPU/GPU, on-device | **Structured notes + action items (local LLM)** | Most mature genuine "self-hosted Otter.ai alternative" |
| [chidiwilliams/buzz](https://github.com/chidiwilliams/buzz) | MIT | ~22k | CPU/GPU | Raw transcript/translation | GUI front-end only |
| [collabora/WhisperLive](https://github.com/collabora/WhisperLive) | MIT | ~4.3k | GPU for real-time | Streaming transcript | Live captioning, not batch notes |
| [transcribee/transcribee](https://github.com/transcribee/transcribee) | AGPL-3.0 | 522 | CPU/GPU | Collaborative editable transcript | Copyleft caveat |
| RenzoBeux/murmur, helmcode-whisper | MIT | 3–5 | Local Whisper+pyannote+Ollama | Structured notes (local LLM) | Small/early but architecturally closest full-local pipeline reference |

**Finding**: No project does transcript→LLM→**requirements/PRD** output specifically out of the box. Closest are general "meeting-minutes" generators (Meetily, Murmur). Recommended build: whisper.cpp/faster-whisper (transcription) → local LLM (Ollama/vLLM) with a custom PRD/URD prompt template → rendered Markdown.

### D2. Open-source PRD/URD generators
The genuine open-source landscape is **thin** — nothing exceeds ~70 stars, several unlicensed or stale:
- `fivetaku/show-me-the-prd` (MIT, 66⭐, active) — CLI interview-driven generator, closest conceptual match.
- `dredozubov/prd-generator` (MIT, 57⭐) — Claude Code plugin, tightly coupled to that runtime.
- `AungMyoKyaw/prd-creator` (**no license declared** — caveat), `Vann4799/ai-prd-generator` (MIT, 24⭐), `nanagajui/agentic_prd` (MIT, 11⭐, stale 2025).
- **Recommendation**: don't adopt any as infrastructure — treat as prompt references and build a thin PRD-generation layer (prompt template + Markdown renderer) on a self-hosted LLM runtime. This is exactly what the `github/awesome-copilot` PRD skills (Section B2) already provide at a higher quality bar.
- Commercial-only (NOT self-hostable, reference only): plug.new, Notion AI, ChatPRD, Craft.PM-style tools.

---

## Recommended On-Prem Stack

**Wiki/authoring layer**: **Azure DevOps Wiki** as the system-of-record for narrative URD content (zero new infra, native Git-backed Markdown, official Microsoft MCP server, native work-item traceability via `#123` links), *optionally* paired with **Docmost** deployed alongside for PMs who need a richer Notion/Confluence-like authoring UX with built-in page-verification/sign-off workflow — Docmost content can be periodically exported/synced to Markdown in Azure Repos to keep a single git-backed source of truth.

**Spec format layer**: **OpenSpec** (or GitHub Spec Kit) as the canonical machine-readable spec format, using OpenSpec's **"Stores"** pattern — a dedicated Git repo (hosted in Azure Repos) holding only `openspec/specs/` and `openspec/changes/`, decoupled from code repos, so every downstream coding agent across the org reads one canonical, cross-repo source. Build a transformation agent (using the `awesome-copilot` `breakdown-feature-prd` pattern as a reference) that: (a) reads PM-authored URDs from Azure DevOps Wiki via its REST API, (b) decomposes them into OpenSpec-formatted requirement/scenario blocks, and (c) pushes linked Work Items back into Boards — closing the ADO-native traceability gap identified in Section B1.

**RAG/agent-query layer**: **Onyx** (formerly Danswer) as the primary enterprise-search/RAG platform — MIT core, native Confluence connector (if Confluence is also in play), official self-hosted MCP server in both directions (exposes itself as MCP server, consumes MCP plugins), Ollama/vLLM/Azure OpenAI support. Pair it with the **official `microsoft/azure-devops-mcp`** server for direct ADO Wiki/Boards/Repos access, since Onyx has no native ADO connector. For deeper custom pipelines, **LlamaIndex** is the best framework fallback thanks to its ready-made Confluence reader.

**Vector store**: **Qdrant** (Apache 2.0, simple Docker self-host, strong filtering) or **pgvector** if Postgres is already standard in the org's infra — both integrate natively with Onyx/LlamaIndex/Haystack.

**Transcription pipeline** (optional, for interview-to-requirements): **faster-whisper** or **whisper.cpp** for transcription (CPU-friendly, fully offline) → local LLM via **Ollama/vLLM** with a custom URD-extraction prompt (adapted from the `awesome-copilot` `skills/prd/SKILL.md` schema) → Markdown output committed into the OpenSpec Store.

**Reasoning**: This stack keeps all data on-prem/in-tenant, reuses existing Azure DevOps investment rather than introducing a parallel wiki/ACL system, uses the only two genuinely mature official MCP servers found in this entire research pass (Microsoft's ADO MCP and Onyx's native MCP), and adopts OpenSpec/Spec Kit — the two most agent-consumable, actively-maintained, pure-Markdown spec formats — as the bridge between PM prose and coding-agent-executable work, directly addressing the single biggest gap uncovered: **Azure DevOps Boards + Wiki have no native structured, agent-consumable requirements format**, which the OpenSpec Store + transformation-agent pattern is specifically designed to solve.
