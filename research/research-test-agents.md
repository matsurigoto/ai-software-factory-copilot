# AI Software Factory — Test & Review Agent Pool Research Report

> **最終決策備註（2026-11-07）**：本報告為原始調研紀錄，撰寫時以「Azure OpenAI Private Endpoint 或本地模型」與「Azure DevOps Server 地端」為假設前提。後續規劃已定案為：**完全地端 Kubernetes**、**唯一允許的雲端服務為 Azure DevOps Services**、**不使用任何雲端 LLM**（含 Azure OpenAI），模型一律為叢集內 vLLM 或公司內部 endpoint，經 LiteLLM Proxy 路由。閱讀時請以 `AI-Software-Factory-Plan.md` 為準，本文中的 Azure OpenAI / ADO Server 相關內容僅供參考。

**Self-hosted, framework-neutral, Azure DevOps–based environment | Azure OpenAI private endpoint or local Ollama/vLLM**
*Research compiled 2026-11-07. All claims cite primary sources (GitHub repos, official docs, vendor pages). Where a claim could not be verified, it is explicitly flagged as a gap.*

---

## A. Unit / Integration Test Generation Agents

| Tool | License | Self-host | Local/Azure OpenAI LLM | Azure DevOps/CI fit | Maturity | Best for |
|---|---|---|---|---|---|---|
| **Qodo Cover** (cover-agent) | AGPL-3.0 | Yes (CLI/binary) | **Yes** — LiteLLM: OpenAI-compatible `--api-base`, Azure (`azure/<deployment>` + `AZURE_API_*`), Ollama, HuggingFace | CI-agnostic CLI; no native ADO plugin | ~5.7k★, **archived/unmaintained since 2025-06-15** ⚠️ | Multi-language LLM test-gen if you accept maintenance risk or fork |
| **Meta TestGen-LLM** | Closed, paper only (arXiv:2402.09171) | N/A | N/A | N/A | Paper only (FSE'24), no public code | Design pattern: generate→build→pass→coverage-gate filter, as implemented by Qodo Cover |
| **Diffblue Cover** | Proprietary (free IDE Community tier; paid CLI/Enterprise) | **Yes**, incl. offline/air-gapped Enterprise | **N/A** — proprietary RL model, no external LLM swap (no data ever leaves to an LLM API) | **Explicit Azure Pipelines quick-start guide** + GitHub/GitLab/Jenkins/AWS | Established commercial product, actively sold | Java/Kotlin shops wanting zero-LLM-dependency, fully local generation |
| **Keploy** | Apache-2.0 | Yes, fully | Core engine is non-LLM (record/replay); "AI coverage-expansion" feature ties to Keploy Cloud (`app.keploy.io`) — unverified local-LLM path | Generic CI-agnostic CLI/Docker; no dedicated ADO task found | ~19.5–20k★, very active, CNCF-listed | API/integration regression tests + mocks from real traffic, not unit tests |
| **EvoSuite** (Java) | LGPL-3.0 | Yes (jar/Docker) | N/A (genetic-algorithm search, no LLM) | CLI/Docker, CI-agnostic | ~920★, **sporadic** (year+ gaps between commits) | Non-LLM Java coverage supplement; low confidence in longevity |
| **Pynguin** (Python) | MIT | Yes (pip) | N/A (search-based) | CLI, CI-agnostic | ~1.4k★, **very active** (near-daily commits) | Best-maintained classical Python generator — **must sandbox**, README warns it can wipe the host disk |
| **TestPilot** (MSR) | N/A — no public release | No | N/A | N/A | Paper only (arXiv:2302.06527) | Methodology reference: signature+impl+docs prompt, failure-driven repair loop |
| **Aider** (general coding agent as test writer) | Apache-2.0 | **Yes**, fully (pure Python CLI) | **Yes, first-class** — dedicated docs pages for Ollama, Azure OpenAI (`AZURE_API_*` env vars), and generic OpenAI-compatible/local endpoints | CLI, non-interactive `--yes` scripting in any pipeline | ~50k★, very active | **Standout pick** for framework-neutral, self-hosted test-writing against a private Azure/local LLM |
| **GitHub Copilot / Claude Code as test writers** | N/A — platform-governed | Governed by platform (Copilot Business/GHES; Claude Code via Bedrock/Vertex) | Varies by enterprise deployment | Pattern, not product | N/A | Prompt/skill pattern layered onto whichever coding agent the factory already standardizes on |
| **Stryker** (JS/.NET/Scala mutation) | Apache-2.0 | Yes (`npx`/`dotnet tool`) | N/A (mutation tester) | Stryker.NET's own CI runs partly on **Azure DevOps** (`dev.azure.com/stryker-mutator`) | ~3.3k★ (StrykerJS), releases through Oct 2026 | Validating AI-generated JS/TS/.NET test quality |
| **PIT (PITest)** Java mutation | Likely Apache-2.0 (unverified — LICENSE fetch 404'd) | Yes (Maven/Gradle/jar) | N/A | Own CI partly on Azure DevOps; Maven/Gradle goal, CI-agnostic | ~1.9k★, active (v1.30.2) | Validating AI-generated Java test quality |
| **mutmut** (Python mutation) | BSD-3-Clause | Yes (needs Linux/WSL — `fork()` dependency) | N/A | CLI, CI-agnostic | ~1.5k★, active | Validating AI-generated Python test quality |

**Key takeaway:** Pure LLM-driven unit-test generators are either archived (Qodo Cover) or unreleased research (TestGen-LLM, TestPilot). The most durable self-hosted path is a **general coding agent (Aider is the standout) wrapped in a custom "generate → run → coverage-gate → repair" loop**, validated by a **mutation-testing gate** (Stryker/PIT/mutmut) per language, with Diffblue Cover as a non-LLM Java-only alternative and Pynguin/EvoSuite as sandboxed classical supplements.

---

## B. E2E / UI Test Agents

| Tool | License | Self-host | Local/Azure LLM support | Mobile/Web | Maturity | Azure DevOps CI fit | Best for |
|---|---|---|---|---|---|---|---|
| **Playwright MCP** (microsoft/playwright-mcp) | Apache-2.0 | ✅ Full (local Node process) | N/A — vendor-neutral; LLM lives in whatever MCP client drives it | Web | 39.3k★, very active (Microsoft) | Excellent — headless Node CLI, no SaaS | Agentic/self-healing browser automation driven by your own Azure/local-LLM agent |
| **Stagehand** (Browserbase) | MIT | ⚠️ Partial — local browser OK; "Model Gateway" convenience needs Browserbase cloud | ✅ 5 built-in direct providers + **BYO-LLM callback** for Azure OpenAI/Bedrock/self-hosted (custom glue required) | Web | 26k★, very active | Good (TS/Python/Go SDK) | Teams willing to write a small LLM-callback adapter for full on-prem operation |
| **browser-use** | MIT | ✅ Full (pure Python+Playwright; cloud optional) | ✅ **Excellent** — native `ChatAzureOpenAI`, `ChatOpenAI` with custom base URL (Ollama/vLLM), Bedrock, Vertex, 15+ providers | Web | 120k★, very active (largest community surveyed) | Very good (pip package, headless) | **Primary recommendation** for general-purpose on-prem AI browser agent |
| **Skyvern** | **AGPL-3.0** ⚠️ copyleft | ✅ Full (local server+UI; cloud optional) | Unverified in this pass (docs 404'd) | Web | 23.4k★, active | Moderate (service+UI) | Vision-heavy no-code workflows **if** AGPL is acceptable |
| **Midscene.js** (ByteDance) | MIT | ✅ Full (npm) | ✅ **Excellent** — generic `MIDSCENE_MODEL_BASE_URL`/`API_KEY`/`MODEL_NAME`/`MODEL_FAMILY` = any OpenAI-compatible endpoint (Azure, Ollama, vLLM) | **Web + Android** | 15.5k★, very active | Strong (CLI+YAML, Playwright/Puppeteer) | Natural-language YAML E2E across web+Android with full LLM flexibility |
| **TestZeus Hercules** | **AGPL-3.0** ⚠️ | ✅ Full (pip/Docker) | ✅ Good — generic `LLM_MODEL_NAME`/`LLM_MODEL_API_KEY`, multi-agent JSON config, `CDP_ENDPOINT_URL` for on-prem browser attach | Web | 1.2k★, smaller community | Good (own CI badge, Docker image) | Gherkin-driven E2E in CI/CD **if** AGPL acceptable |
| **Shortest** (antiwork) | MIT | ✅ Execution only | ❌ **Locked to Anthropic API** — no Azure/local path found | Web | 5.7k★, **stale since Aug 2026** | Fair | Only if standardizing directly on Anthropic Claude |
| **"Magnitude"** | Apache-2.0 (legacy pkg) | ⚠️ original design was self-hosted | N/A | Web (legacy) | 7.7k★ but **project has pivoted to an unrelated local-inference-engine product** | N/A | **Avoid / high risk** — treat as discontinued for E2E testing pending confirmation |
| **Maestro** (mobile.dev) | Apache-2.0 | ✅ Full (single binary, no drivers; cloud optional) | N/A — AI layer is Maestro MCP, inherits whatever LLM your MCP client uses; core flows are deterministic YAML | **Mobile (Android+iOS sim) + Web** | 16.3k★, very active, production use at Meta/Expo | Excellent (YAML flows in-repo, CI-proven) | **Best mobile/web UI pick** — deterministic CI replay + optional agentic authoring |
| **Appium + LLM wrapper** | Apache-2.0 | ✅ Full, industry-standard | ✅ Full flexibility — you choose the LLM layer, no lock-in | Mobile + Desktop + Web (widest coverage) | 22.2k★, very active, industry standard | Excellent, proven | Teams needing maximum platform coverage, willing to build the NL wrapper |
| **Testsigma / Mabl / testRigor / Functionize / Checksum.ai / Octomind** | Proprietary SaaS | ❌ Primarily SaaS; only Testsigma possibly offers enterprise self-host (unconfirmed) | ❌ Cloud-vendor LLM only | Mixed | Established vendors | Cloud APIs/plugins only | **Not a fit** for strict on-prem/no-SaaS requirement |

**Key takeaway:** **browser-use** (web) + **Maestro** (mobile/web) + **Playwright MCP / Midscene.js** (agentic/self-healing layer) form a fully self-hostable, Azure-OpenAI/Ollama-compatible E2E stack with no copyleft or vendor lock-in. Avoid Skyvern/Hercules unless AGPL is cleared legally, avoid Shortest (Anthropic lock-in) and "Magnitude" (apparent abandonment for testing use case).

---

## C. API Testing Agents

| Tool | License | Self-host | AI/LLM-driven? | OpenAPI-spec driven? | Azure DevOps/CI fit | Maturity | Best for |
|---|---|---|---|---|---|---|---|
| **Keploy** | Apache-2.0 | Yes (eBPF-based; some protocol parsers enterprise-only) | Partial — AI coverage-expansion tied to cloud console (`app.keploy.io`) | Uses OpenAPI/Swagger schema for coverage expansion | Generic CLI/Docker step | Very high, 19.5k★ | Zero-code regression test + mock generation from real traffic |
| **Schemathesis** | MIT | Yes, pure Python CLI | Opt-in — supports LLM-generated fuzz dictionaries dropped in as plain files | **Yes — core OpenAPI/GraphQL/gRPC property-based testing** | Excellent — pip CLI, JUnit/HTML/Allure output | High, 3.75k★, very active | Spec-driven fuzzing/contract testing, fully on-prem |
| **RESTler** (Microsoft) | MIT | Yes (Win/Linux/Docker) | No — heuristic/dynamic-analysis | Yes — core stateful-sequence inference from OpenAPI | **Confirmed** — its own CI runs on Azure DevOps | Moderate, 2.97k★, research cadence | Stateful security/reliability fuzzing of REST APIs |
| **Postman AI (Agent Mode, fka Postbot)** | Proprietary (Newman CLI MIT) | No for AI features (cloud-only); Newman CLI self-hostable | Yes, but cloud-credit billed; no confirmed BYO-model | Can generate/update OpenAPI specs | Newman/Postman CLI as generic script step; no confirmed dedicated ADO extension | Market-leading | Teams accepting cloud AI; **not** for air-gapped factories |
| **Hoppscotch** | MIT (CE); open-core EE | Yes — CE and EE both explicitly self-hosted/on-prem | **No AI/LLM feature found** (confirmed gap) | No dedicated spec-driven test generation found | CLI (`hopp`) as generic script step | Very high, 80.8k★, very active | Self-hosted, git-friendly Postman replacement (manual testing) |
| **Step CI** | MPL-2.0 | Yes (npm/Homebrew CLI) | Unverified — `generate` command referenced but nature unconfirmed | YAML-driven workflows | Generic script step; designed for CI/CD | **Caution: ~2+ years without commits** | Lightweight multi-protocol testing if maintenance re-verified |
| **Bruno** | MIT | Yes — offline-first, no cloud sync ever | No AI/LLM feature; `bru import` converts OpenAPI/WSDL → collection (not LLM) | Partial (spec import only) | Documented CI/Docker guides, JUnit output | Very high, 48.3k★, very active | Offline/air-gapped, git-native API client + CI runner |
| **EvoMaster** | LGPL-3.0 | Yes (pip/Docker) | "AI-driven" via evolutionary search, not LLM | Yes — compiles OpenAPI/GraphQL/RPC schemas into fuzz targets | pip/Docker CLI + own GitHub Action | High, 800★, active, academically backed | Whitebox/security-fault-class test generation (BOLA, SQLi, etc.) on-prem |
| **RESTGPT / APITestGenie** (academic) | MIT / GPL-3.0 | Yes (Docker), but call OpenAI cloud API per run | Yes — LLM-driven spec-constraint extraction / Jest test generation | Yes | Research scripts, not CI-packaged | Low, research prototypes (14★/5★) | Illustrative of the "spec + LLM → test" direction, not production-ready |

**Key takeaway:** **Schemathesis** is the strongest fully self-hosted, OpenAPI-spec-driven, CI-native pick; **Keploy** adds zero-code regression/mock generation from real traffic; **RESTler**/**EvoMaster** add stateful/security fuzzing. True LLM-generative API testing from specs remains academic-stage (RESTGPT, APITestGenie) — treat as a pattern to prototype internally, not adopt directly. Bruno/Hoppscotch cover manual/git-native API client needs but have no AI layer.

---

## D. AI Code Review & Quality Gates — Azure DevOps Verified

| Tool | License | Self-host | Local/Azure OpenAI LLM | **Azure DevOps PR Support** | Maturity | Best for |
|---|---|---|---|---|---|---|
| **Qodo Merge / PR-Agent** | MIT (OSS) | Yes (CLI/Docker/webhook) | **Yes** — LiteLLM → Azure OpenAI, Bedrock, Vertex, Ollama, etc. | **✅ YES, explicit & documented** — full `azure-pipelines.yml` sample, PAT or `DefaultAzureCredential`. Caveat: Azure Repos doesn't honor YAML `pr:` triggers (use Build Validation policies); no comment-triggered re-run | Active OSS (community-maintained, distinct from Qodo's commercial product) | **Best overall fit** — free, self-hosted, genuinely local-LLM-capable, first-class ADO docs |
| **CodeRabbit** | Commercial SaaS | Yes, **Enterprise-only (500+ seats)** | Yes — "Bring your own model" documented reason to self-host | **✅ YES, explicit first-class** — Entra service-principal or PAT auth, `@coderabbitai` interactive comments | Mature, well-funded vendor | Polished commercial reviewer, if 500+-seat Enterprise self-host is viable |
| **Sourcery** | Commercial SaaS | No (SaaS only, even against self-hosted GitLab) | Not offered | **❌ NO** — GitHub/GitLab only, confirmed from docs tagline | Established | Not viable here |
| **Danger / Danger-JS** | MIT | Yes (CLI) | N/A — DIY (write LLM calls inside a Dangerfile) | **⚠️ PARTIAL** — Azure Pipelines is a supported *CI trigger*, but Azure Repos is **not** a supported PR data source (GitHub/Bitbucket only); custom plugin needed | Mature, low-velocity | Rule-based linting on GitHub/Bitbucket; not a real ADO solution out of the box |
| **SonarQube** (Community/Developer+) | Community: OSS (self-host); Developer+/AI features: commercial | Yes (Community Build on-prem) | AI CodeFix/Vortex/Gitar tier-gating on self-hosted Server unclear (marketing references "SonarQube Cloud Enterprise") | **✅ YES, but tiered** — official Marketplace extension; **Quality Gate works on Community, but PR decoration requires Developer/Enterprise edition** | Extremely mature, dominant | Teams already needing SAST/quality-gate, willing to pay for Developer+ for PR decoration |
| **Semgrep** (OSS core + AppSec Platform) | OSS core + commercial SaaS platform | CLI: yes, fully; AppSec Platform (PR comments/dashboard): SaaS only | N/A (pattern-based, not LLM) | **✅ YES (CI + SCM connector confirmed)**, but PR-comment feature matrix documents only GitHub/GitLab explicitly — Azure Repos support appears secondary/less polished | Mature, active; ships an MCP server | Supplementary SAST CI gate; verify PR-comment behavior on Azure Repos empirically |
| **CodeQL** | **Not OSS for general private use** — restricted to open-source/academic unless GHAS-licensed | CLI runs anywhere technically, but legally gated | N/A (static analysis) | **⚠️ PARTIAL, via GHAzDO** (separate paid Microsoft product) — on-prem **Azure DevOps Server** availability unverified (docs lean toward Azure DevOps *Services*) | Extremely mature engine, weak path outside GitHub | Only if already licensing GHAS/GHAzDO; otherwise avoid for private ADO repos |
| **Snyk** | Commercial SaaS (Broker component OSS) | No true self-hosted scanning engine; Broker only proxies network reachability | Not offered | **✅ YES** — official "Snyk Security Scan" Azure Pipelines task | Mature | CI dependency/container scanning; weak fit for strict data-residency/air-gap |
| **Trivy** | **Apache-2.0 (OSS)** | **Yes, fully** | N/A (vuln-DB scanner) | Likely via generic CLI/container step; dedicated marketplace extension unverified | Very active, Aqua-backed | Strongest pure-OSS self-host fit for vuln/IaC/secret/container scanning |
| **GitLab Duo Code Review** | Commercial add-on | Yes (GitLab Self-Managed) | Unverified | **❌ NO** — GitLab MRs only | Mature within GitLab | Reference only, not applicable |
| **Bito** | Commercial (self-host Docker/K8s) | **Yes** | **Yes** — Governor is literally an LLM gateway; BYOK documented | **❌ NO** — GitHub/GitLab/Bitbucket only | Product pivoted away from standalone PR-review branding | Interesting for LLM-gateway needs, not ADO review |
| **Ellipsis** | Commercial SaaS | **No** | No (fixed harness models) | **❌ NO** — GitHub App only | Newer cloud platform | Not viable (no self-host, no ADO) |
| **Greptile** | Commercial (Enterprise self-host) | **Yes** (Docker Compose/K8s/air-gapped) | Likely yes (custom LLMs mentioned, not deeply verified) | **❌ NO** — GitHub/GitLab/Bitbucket/Gitea/Perforce only | Growing fast | Strong self-host story but hard-blocked by missing ADO support |
| **Graphite Diamond** | Commercial SaaS | No | Not offered | **❌ NO** — GitHub-only | Established in GitHub niche | Not applicable |

**Critical finding: of 14 tools evaluated, only 3 have confirmed, documented Azure DevOps Repos PR integration — Qodo Merge/PR-Agent, CodeRabbit, and SonarQube (tiered).** Semgrep has partial/CI-only support; Danger works generically via CLI but not as a PR data source. The rest are GitHub/GitLab/Bitbucket-exclusive.

---

## E. Performance / Security Test Agents

| Tool | Category | License | Self-host | AI-driven? | Azure DevOps/CI fit | Maturity | Best for |
|---|---|---|---|---|---|---|---|
| **k6** (Grafana) | Performance | AGPL-3.0 | Yes, fully (binary/Docker); Grafana Cloud k6 is optional add-on | No native AI test-gen found; integrates into CI generically | CLI/Docker step; no confirmed dedicated Azure DevOps Marketplace extension found in this pass | ~32k★, very active (commits "yesterday") | Core load/stress/soak testing in Azure Pipelines via script step |
| **Locust** | Performance | MIT | Yes, fully (Python) | No AI component | Plain Python CLI, trivially scriptable in Azure Pipelines | ~28k★, very active | Python-native distributed load testing |
| **OWASP ZAP** | Security (DAST) | Apache-2.0 | Yes, fully — "world's most widely used web app scanner," fully OSS | Not LLM-driven; the **Automation Framework** add-on provides declarative `-autorun`/YAML job plans with programmatic exit codes (0/1/2) for CI gating — an MCP server for ZAP was not confirmed to exist at the docs path checked (gap) | Runs as a CLI/Docker step (`zap.sh -cmd -autorun config.yaml`) in any pipeline incl. Azure Pipelines; CodeQL/SonarCloud badges show it's CI-proven itself | ~16k★, very active (commit "yesterday") | **Strongest pure-OSS DAST gate** — deterministic, exit-code-driven, zero license cost |
| **Nuclei** (ProjectDiscovery) | Security (template-based vuln scanning) | MIT | Yes, fully (Go binary) | No (template/YAML signature-based, not generative) | CLI, trivially scriptable in any CI | ~33k★, very active | Fast, broad CVE/misconfig scanning as a CI gate |
| **PentestGPT** | Security (LLM-driven pentest agent) | MIT | Yes — CLI tool, runs locally | **Yes** — v1.0 "Agentic Upgrade" drives Claude Code/Codex autonomously through recon→exploit→report stages; a separate **legacy interactive mode (`pentestgpt-legacy`)** explicitly supports OpenAI, Anthropic, Gemini, DeepSeek, xAI, Qwen, Moonshot, **and local Ollama** | CLI; no CI-specific packaging found — designed as an interactive/semi-autonomous assistant, not a CI gate | ~16k★, USENIX Security 2024 paper, last commit "July" (a few months stale) | Research-grade autonomous/assisted pentesting; **not CI-gate-ready**, best for periodic manual-assisted security assessments with a local LLM |
| **HexStrike AI** (0x4m4/hexstrike-ai) | Security (LLM-driven MCP pentest framework) | MIT | Yes — "lets local AI agents autonomously run 150+ cybersecurity tools" via an MCP server, implying local/any-LLM-client operation (Claude, GPT, Copilot, etc.) | **Yes**, core premise | No CI packaging found; MCP-server architecture, so it's invoked by an interactive/agentic client, not a deterministic pipeline step | ~13k★, MIT, last commit "August" (a few months stale) | Experimental agentic pentest orchestration layered on existing OSS security tools — treat as early-stage |

**Honest maturity caveat (explicit, as requested):** The "AI/LLM-driven pentest agent" space (PentestGPT, HexStrike AI, and the broader ecosystem of forks/wrappers around HexStrike found during search) is **experimental and research-grade, not production CI-gate-ready**. Both flagship tools show several months of commit inactivity at research time, neither ships a deterministic CI/exit-code integration comparable to ZAP's Automation Framework, and running an autonomous agent against production-adjacent infrastructure carries real operational risk (unbounded tool execution, prompt-injection surface from scanned content, no built-in blast-radius controls). **Recommendation: use ZAP + Nuclei (deterministic, mature, OSS) as the actual CI security gate; treat PentestGPT/HexStrike as an assisted, human-supervised exploratory-security tool run periodically in a sandboxed environment, not as an automated pipeline stage.**

*Gap:* could not confirm a dedicated Azure DevOps Marketplace extension for k6 or Nuclei in this pass (both run fine as generic CLI/Docker pipeline steps regardless).

---

## F. Test Management & Requirements-to-Test Traceability

| Tool | License | Self-host | Azure DevOps integration | AI features | Maturity | Best for |
|---|---|---|---|---|---|---|
| **Kiwi TCMS** | GPL-2.0 | Yes (Docker) | None native; generic JSON-RPC/XML-RPC/OpenAPI + **MCP server** exposed; custom sync needed | MCP server for LLM tooling (not auto-analysis) | ~1.3k★, active (weekly commits) | OSS test-case/plan management with a flexible API/plugin layer |
| **TestLink** | GPL | Yes (LAMP/Docker) | None | None | ~1.6k★ but **confirmed stale** — maintainers state 1.9.x is final, only AI-assisted PHP-8 compatibility patches in 2025 | **Not recommended** — legacy reference only |
| **Azure Test Plans** (Azure DevOps native) | Commercial, per-user | **Yes** — Azure DevOps Server is explicitly on-prem | **Native** — same work-item graph as Boards/Repos/Pipelines | None documented as "AI" | Microsoft first-party, continuously shipped | **Best native fit if already on Azure DevOps Server on-prem** — zero integration effort, WIQL-queryable traceability |
| **Allure Report (OSS)** | Apache-2.0 | Yes | **Native** — official `PublishAllureReport@2` Azure Pipelines extension | None (reporting only) | ~5.6k★, very active (daily commits) | Rich test-result reporting bolted onto existing Azure Pipelines runs |
| **Allure TestOps** | Commercial (Qameta) | Yes — "hosted by us or deployed in your infrastructure" | Documented Xray integration; **no confirmed native Azure Boards sync** (gap) | Not verified | Commercial | Full case-management layer on top of Allure reporting |
| **ReportPortal** | Apache-2.0 | **Yes** (Docker Compose, primary deployment model) | None native (Jira/Rally/SauceLabs plugins only) | **Yes — verified**: dedicated `service-auto-analyzer` microservice does ES/OpenSearch log-similarity failure clustering + continuous retraining, fully self-hostable, no cloud AI dependency | ~2k★, active | **Best on-prem AI failure-triage/flaky-test clustering dashboard** |
| **Xray** (Jira) / **Zephyr** (SmartBear) | Commercial | Jira-bound (DC "on-prem" still requires Jira) | **None found** for Azure DevOps | Yes (marketed AI test generation) | Mature | Jira-centric orgs only — not applicable here |
| **Doorstop** | LGPLv3 | Yes (Git + local CLI, trivially on-prem) | None native; tag ADO work-item IDs as custom YAML fields + custom CI reconciliation via REST API | None | ~693★, active | Lightweight, diffable, git-native requirements store for safety/compliance-style SDLCs |
| **StrictDoc** | Apache-2.0 | Yes (Git + local CLI/web server) | Same custom-tagging pattern as Doorstop | None | ~448★, active, younger/smaller community | More polished UI/traceability-matrix view, custom `.sdoc` DSL |
| **sphinx-needs + sphinx-test-reports** | MIT | Yes (pure local toolchain) | Indirect — ingests JUnit XML (native to Azure Pipelines) to auto-build traceability matrices | None | ~321★, active, niche | Building a custom OSS CI-generated traceability matrix without a full TCMS |
| **pytest-azurepipelines** | OSS | N/A (CI plugin) | **Native** — publishes pytest results directly into Azure Pipelines UI; `record_pipelines_property()` tags a test with arbitrary key/value (e.g. work-item ID) | None | Established PyPI plugin | Simplest way to tag pytest results with requirement/work-item IDs |

**Key takeaway:** No OSS tool natively syncs with Azure Boards/Test Plans out of the box except **Azure Test Plans itself** and **Allure Report's official Azure Pipelines extension**. The practical in-ecosystem pattern is: tag tests with Azure Boards work-item IDs via `pytest-azurepipelines`/sphinx-needs, reconcile via the Azure DevOps REST API/WIQL in CI, and use **ReportPortal** for AI-driven failure-pattern analysis layered on top.

---

## G. Agent Evaluation / QA-of-the-Agents & Observability

| Tool | Category | License | Self-host | Local/Azure OpenAI support | Maturity | Best for |
|---|---|---|---|---|---|---|
| **SWE-bench** | Eval harness/methodology | MIT | Yes — pure local/Docker harness, no SaaS required | Model-agnostic — you supply patches from any model | Mature since 2023, ICLR'24 oral, active through 2026 | Blueprint for an in-house "issue→patch→hidden-tests→pass/fail" harness scoring the factory's own coding/testing agents |
| **Promptfoo** | Eval (prompt/model) + red-team | MIT core; some enterprise add-ons | Yes — "100% locally," no backend needed | Native `azure` provider (API key/SP/CLI auth) and `ollama` provider; generic HTTP for vLLM-compatible endpoints | Mature, very active (now OpenAI-affiliated) | CI-gated prompt/response quality + red-teaming |
| **DeepEval** | Eval (unit-test style, agent trajectories) | Apache-2.0 (core) | Yes — fully local; optional paid SaaS (Confident AI) for prod trace monitoring, not required | `AzureOpenAIModel` (key/AD-token/SP) and `OllamaModel` built in | Very active, frequent releases | pytest-style testing of agent steps/trajectories — directly relevant to coding/testing agent QA |
| **Ragas** | Eval (RAG-focused) | Apache-2.0 | Yes — pip library, no backend | Via LiteLLM/Instructor: documented Azure OpenAI example; Ollama/vLLM inferred via LiteLLM (not directly documented) | Active; canonical org possibly moved (`explodinggradients`→`vibrantlabsai`, unresolved) | Only relevant if factory agents do RAG-style retrieval; agent-trajectory templates still "coming soon" |
| **Langfuse** | Observability/tracing + evals | **Open-core**: MIT core + paid Enterprise features (RBAC, audit logs, data retention, SCIM, masking) | Yes — Docker Compose/K8s/Helm/Azure; needs Postgres+Redis+ClickHouse+S3 | BYO LLM API, same-VPC/intranet reachable (Azure OpenAI/Ollama/vLLM feasible) | Mature, YC-backed, active | Full-stack prod observability/prompt-management, if EE features (RBAC/audit) aren't hard requirements |
| **Arize Phoenix** | Observability/tracing + evals | **Fully OSS** — "100% Free... no feature gates" when self-hosted | Yes — CLI/Docker/K8s/Helm/Azure, documented air-gap support | OpenInference auto-instrumentation + OpenAI/Anthropic/Bedrock/LiteLLM (→Azure/Ollama) | Very active, GitHub-trending | **Cleanest no-catch OSS tracing+eval platform** — best default for air-gapped observability |
| **OTel GenAI Semantic Conventions** | Spec / integration layer | Apache-2.0 | N/A (spec); OTel SDKs run entirely locally | Vendor-neutral by design — any OTLP backend (Jaeger, Tempo, Phoenix, Langfuse) | **Pre-stable**, actively moving repos ("Schema URL: TODO") | Standardizing instrumentation so the observability backend stays swappable |

**Key takeaway:** **Promptfoo + DeepEval + Phoenix are fully OSS/self-hostable with no cloud dependency and native Azure OpenAI/Ollama support** — the cleanest combination for evaluating and observing the factory's own agents. SWE-bench's architecture (not its dataset) is the right template for an in-house "coding-agent-produced-patch passes hidden tests" gate. Standardize instrumentation on OTel GenAI conventions / OpenInference now, even while the spec is pre-stable, to keep the observability backend replaceable.

---

## Final Recommended On-Prem Stack

### 1. Test agent pool (per SDLC stage)

| Stage | Primary pick | Why | Fallback/complement |
|---|---|---|---|
| Unit/integration test generation | **Aider** (self-hosted, Azure OpenAI/Ollama-native) wrapped in a custom generate→run→coverage-gate→repair loop | Only framework-neutral, actively maintained, first-class Azure/local-LLM tool in this category | Diffblue Cover for Java-only, zero-LLM-dependency teams; Pynguin (sandboxed) as classical Python supplement |
| Unit test **quality gate** | **Stryker** (JS/TS/.NET) + **PIT** (Java) + **mutmut** (Python, Linux/WSL) | Mutation testing catches assertion-free/no-op AI-generated tests that merely execute code | — |
| API test generation | **Schemathesis** (OpenAPI property-based) + **Keploy** (record/replay regression + mocks) | Fully self-hosted, spec-driven, CI-native; Keploy adds zero-code traffic-based regression | RESTler/EvoMaster for stateful/security-class fuzzing |
| E2E/UI test (web) | **browser-use** (general agent) + **Playwright MCP** or **Midscene.js** (agentic/self-healing layer) | Best-documented Azure OpenAI/Ollama support, largest OSS community, MIT-licensed | Stagehand if a BYO-LLM callback adapter is built |
| E2E/UI test (mobile) | **Maestro** | Apache-2.0, deterministic YAML flows, CI-proven at Meta/Expo scale, optional Maestro MCP for agentic authoring | Appium + custom LLM wrapper for maximum platform coverage |
| Exploratory QA | **browser-use** / **Playwright MCP** driven interactively by the factory's coding-agent LLM | Reuses the same self-hosted LLM infrastructure, no new vendor | — |
| Performance testing | **k6** + **Locust** | Both fully OSS, mature, CI-native as generic pipeline steps | — |
| Security testing (CI gate) | **OWASP ZAP** (Automation Framework, deterministic exit codes) + **Nuclei** (template scanning) + **Trivy** (container/IaC/vuln) | Fully OSS, deterministic, production-grade, zero license cost | — |
| Security testing (periodic/assisted) | **PentestGPT** (legacy mode with local Ollama) run human-supervised, out-of-band | Agentic pentest tooling is not yet CI-gate-ready; use as an assisted exploratory tool, not an automated pipeline stage | HexStrike AI if MCP-orchestrated multi-tool pentesting is wanted (equally experimental) |

### 2. AI code review / quality gate

- **Primary: Qodo Merge / PR-Agent**, self-hosted via Docker in Azure Pipelines, configured against the factory's Azure OpenAI private endpoint via LiteLLM. It is the only tool surveyed that is simultaneously **free, OSS, genuinely local-LLM-capable, and has first-class documented Azure DevOps Repos support** (with the caveat: configure Build Validation branch policies, not YAML `pr:` triggers, since Azure Repos doesn't support PR-triggered YAML pipelines; no comment-triggered re-runs).
- **Static-analysis/SAST layer: SonarQube Community/Developer+** (Developer+ needed for actual PR decoration in Azure Repos) + **Trivy** (container/IaC/vuln, pure OSS) + **Semgrep CLI** (pattern-based SAST as a generic CI step; don't rely on its PR-comment feature for Azure Repos without empirical verification).
- **If budget allows Enterprise-tier (500+ seats): CodeRabbit self-hosted**, which has explicit BYO-LLM and first-class Azure DevOps support, as a second opinion alongside PR-Agent.
- **Avoid for this environment:** Sourcery, Bito, Ellipsis, Greptile, Graphite Diamond, GitLab Duo — all lack Azure DevOps Repos PR support; CodeQL's real PR-annotation path (GHAzDO) has an unresolved on-prem/Azure DevOps **Server** gap that needs direct Microsoft licensing confirmation before being relied upon.

### 3. Test management & traceability

- If the org is already committed to Azure DevOps Server on-prem: **Azure Test Plans** gives native, zero-integration-effort requirement↔test-case↔work-item traceability via the existing work-item graph (queryable with WIQL).
- Layer **Allure Report** (OSS, official Azure Pipelines extension) on top for rich per-run test reporting, and **ReportPortal** (fully self-hosted, Apache-2.0) for its verified ML-based failure-clustering/auto-analysis engine — the only tool in this category with a genuine, self-hostable "AI failure analysis" capability.
- For a lighter-weight, fully git-native alternative to Azure Test Plans licensing costs: **Doorstop or StrictDoc** for requirements-as-code, combined with **pytest-azurepipelines** (`record_pipelines_property`) or **sphinx-needs + sphinx-test-reports** to tag tests with Azure Boards work-item IDs and auto-generate a traceability matrix as a build artifact.

### 4. Agent evaluation & observability

- **Instrumentation layer:** standardize all agents (coding, review, and every test agent above) on **OpenTelemetry GenAI semantic conventions / OpenInference**, exporting OTLP to a self-hosted collector. This keeps the backend swappable even though the spec is still pre-stable.
- **Observability backend: Arize Phoenix** as the default — it is the only tool surveyed with an explicit "100% free, no feature gates, fully air-gappable" self-hosted story, and it natively consumes OTel/OpenInference traces from Azure OpenAI, Ollama, vLLM, or LiteLLM-proxied calls. Add **Langfuse** only if its Enterprise-gated features (RBAC, audit logs, data retention policies) are organizationally required — know that those specific features need a paid license even when self-hosted.
- **Eval/regression gate: Promptfoo** (prompt/response quality + red-teaming, fully local) + **DeepEval** (pytest-style agent-trajectory metrics — Task Completion, Tool/Argument Correctness — directly applicable to scoring the coding/testing agents' decisions).
- **End-to-end agent QA: build an internal SWE-bench-style harness** — pin real internal tickets with gold fixes and hidden regression tests, have the coding/testing agents produce patches, score automatically via Docker-isolated test runs. This is the most rigorous, architecturally transplantable methodology found for validating the factory's own agents against ground truth, independent of any specific LLM vendor.

### Overall architecture reasoning
This stack was chosen to satisfy three hard constraints simultaneously: **(1) data residency** — every primary recommendation either runs fully offline/on-prem or explicitly supports a BYO-LLM/Azure OpenAI private-endpoint configuration with no mandatory SaaS call; **(2) Azure DevOps compatibility** — verified rather than assumed, which eliminated otherwise-popular tools (Greptile, Graphite Diamond, Ellipsis, Bito, Sourcery) that are GitHub/GitLab-exclusive; **(3) license hygiene** — AGPL-3.0 tools (Qodo Cover, Skyvern, TestZeus Hercules, k6) are flagged for legal review before any closed-source internal distribution, with permissively-licensed alternatives (Aider, browser-use, Locust) preferred as primaries where functionally equivalent. Where no production-ready OSS tool exists (LLM-native unit-test generation, LLM-native API-spec test generation, agentic pentesting), the report recommends adopting the published **pattern** (filter-gated generate/validate loops, SWE-bench-style harnesses) over a specific immature tool, and treats experimental agentic-security tooling (PentestGPT, HexStrike AI) as human-supervised assistants rather than automated pipeline gates.
