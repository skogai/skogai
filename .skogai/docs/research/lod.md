# LOD Ladder for AI-Facing Repo Context: What Exists and What to Combine (Nov 2026)

There is no de facto standard "level of detail" ladder for AI-facing repo information. What does exist is a set of widely adopted pieces that each cover one or two rungs, plus one standardized *mechanism*, progressive disclosure from Agent Skills, that you can reuse for the whole ladder. The best approach is to compose your L0–L4 from established parts (mani + AGENTS.md + C4/arc42-style docs + ctags/repomix --compress/Serena + raw source) and write a short "descent rules" convention yourself. That convention is the only part nobody has standardized.

## TL;DR
- **No standard LOD ladder exists.** Every mature artifact covers only part of one: AGENTS.md is the L1 file (now under the Linux Foundation's Agentic AI Foundation, whose 9 Dec 2025 press release says it "has already been adopted by more than 60,000 open source projects and agent frameworks"), Agent Skills (metadata → body → referenced files) is the de facto *loading pattern*, and C4 (Context → Container → Component → Code) is the closest established *zoom vocabulary*. Nobody ties levels to question types. That is your gap to fill.
- **Recommended stack:** L0 `mani.yaml` + generated index line → L1 AGENTS.md (short, hand-written) → L2 `docs/architecture/` in C4/arc42 terms (+ ADRs) → L3 deterministic outline (universal-ctags or `repomix --compress`, Aider-style ranked map, or Serena `get_symbols_overview` live) → L4 source. Keep generated wikis (deepwiki-open, CodeWiki) optional and off the critical path.
- **Evidence says keep upper levels thin and hand-written, and keep routing shallow.** ETH Zurich (Gloaguen et al., arXiv 2602.11988) found LLM-generated context files reduced resolution "by 0.5% and 2% on average" with "a cost increase of 20% and 23%", while developer-written files gave "an increase of 4% on average". He et al., "Is Progressive Disclosure All You Need for Long-Context Agents?" (arXiv 2607.17598, Jul 2026) found one routing level helps and "a second, deeper routing level never helps and sometimes breaks accuracy outright, so one level is enough." Your 4-hop router chain (AGENTS → SKOGAI → ROUTES → knowledge/index) is likely deeper than optimal.

## Key Findings

**1. The "levels" the ecosystem actually agrees on**

| Source | Levels it defines | Tied to files or questions? | How the agent descends |
|---|---|---|---|
| Agent Skills spec (SKILL.md) | Per the agentskills.io spec: L1 metadata (name+description, "~100 tokens", "loaded at startup for all skills") → L2 body ("< 5000 tokens recommended", on trigger) → L3 referenced files (on demand) | Files; the description is the trigger | Description matches the task → read body → body points to references |
| C4 model | Context → Container → Component → Code | Diagrams/docs by audience | Human convention; agents only via third-party skills |
| Codified Context (arXiv 2602.20478) | Hot memory (always-loaded constitution) → specialist agents → cold memory (34 on-demand spec docs) | Files + trigger tables by file area | Trigger table routes by which files are touched |
| Hierarchical summarization research (CodeWiki, RepoSummary, Agent4cs) | Component (function/class) → module (directory) → repository | Generated files | Top-down at query time |
| llms.txt | Index (curated links) → llms-full.txt (everything concatenated) | Files | Agent reads index, fetches link or full dump |
| Aider repo map | One level: ranked symbol map sized to a token budget (default 1k) | Generated per turn | PageRank personalized to the files in chat |

**Takeaway:** the field agrees on the *pattern* (small always-loaded summary → on-demand detail), not on named rungs or on rules like "for question type X, stop at level Y."

**2. Comparison by tool (maturity flagged)**

*Agent context file conventions (L1)*
- **AGENTS.md**: https://agents.md — Released by OpenAI Aug 2025, donated to the Linux Foundation's Agentic AI Foundation Dec 2025 alongside MCP and goose. The Linux Foundation reports adoption by "more than 60,000 open source projects and agent frameworks", and it is read by Codex, Cursor, Copilot, Gemini CLI, Jules, VS Code, Aider, Zed and others. The spec repo has ~25.7k stars (MIT). Plain Markdown, no schema. Nested AGENTS.md files per subdirectory are allowed (the nearest one wins), which is the only built-in notion of depth. **MATURE — the L1 standard.** Caveat: an arXiv study notes the community "has not converged on what an AGENTS.md file is" (inline instructions vs pointer vs hybrid). Your router style is the "pointer" variant.
- **CLAUDE.md / .cursor rules**: vendor-specific equivalents. Symlink or point them at AGENTS.md. **Mature but proprietary.**
- **llms.txt**: https://llmstxt.org — Jeremy Howard's proposal from Sep 2024, revised as v2 in Aug 2026. An H1 + blockquote summary + H2 link sections, with an "Optional" section for skippable content. `llms-full.txt` is *not* in the spec; it is a Mintlify-popularized convention. Built for doc *websites*, not repos. Adoption estimates range from ~10% of domains to 36k files in a 3M-site crawl, and Google says it is not required for Search. **Moderate adoption, meant for websites.** Useful idea for L0: a "summary + curated links + Optional" format.
- **Agent Skills**: https://agentskills.io — Open standard since Dec 2025, maintained by Anthropic. The spec repo has ~27.5k stars (Apache-2.0) and 40+ platforms support it. Name ≤64 chars, description ≤1,024 chars, SKILL.md <500 lines, references one level deep. **MATURE — the best-specified progressive-disclosure mechanism.** It is about *procedures*, but its three-tier loading maps directly onto repo knowledge.

*Repo packing / outline tools (L3, partly L4)*
- **repomix**: https://github.com/yamadashy/repomix — ~28–29k stars, MIT, TypeScript, very active, ~255k npm downloads/month. `--compress` uses Tree-sitter to keep signatures and drop bodies; the repomix README claims it "reduces token usage by ~70% while preserving semantic meaning" (a vendor figure; the feature was labelled experimental at launch). Also has a token-count tree, include/ignore globs, an MCP server and a GitHub Action. **MATURE.** Best existing "L3 skeleton" generator for when you need a file artifact. Recommended two-stage use: compressed global map first, then full content for the relevant files.
- **gitingest**: ~15k stars. Full dump plus tree; `github.com` → `gitingest.com` URL trick; CLI via pip. **Mature but flat (L4 only)**, with server-side processing in hosted mode.
- **code2prompt**: ~7.4k stars, MIT, Rust. Handlebars templates and token budgets. **Mature, flat.**
- **files-to-prompt** (simonw): ~2.7k stars, Apache-2.0, the most Unix-composable option. **Stable but low activity, flat.**
- **Aider repo map**: https://aider.chat/docs/repomap.html — Tree-sitter extracts definitions and references, then NetworkX personalized PageRank ranks symbols and binary-searches to fit `--map-tokens` (default 1,024; expands up to ×8 when no files are in chat). Aider has ~50k stars (Apache-2.0), but the map is locked inside Aider; standalone ports exist, e.g. `repo-mapper` in Rust. **The concept is mature; standalone use is experimental.** Its key idea is a *question-dependent* L3: personalization toward the files in play.
- **universal-ctags**: ~7.3k stars, GPL-2.0, active. The oldest Unix-native symbol index; `ctags -R --output-format=json` gives a deterministic L3 that fits git hooks. **Very mature** but has no ranking.

*Semantic code intelligence (live L3 queries)*
- **Serena**: https://github.com/oraios/serena — ~31.5k stars, MCP server backed by LSP, 40+ languages. Its tools form an L3 ladder: `list_dir` → `get_symbols_overview` (top-level symbols, `depth` 0/1) → `find_symbol` (with `include_body` false/true) → `find_referencing_symbols`. Its docs/skills explicitly tell agents to "start with get_symbols_overview" and avoid reading full files. **MATURE, actively released (v2.x).** License: secondary sources say MIT, but the repo README indicates GPL-3.0-or-later for the app and MIT for the SolidLSP part. Check before vendoring.
- **Sourcegraph MCP + SCIP**: SCIP is an open index format for precise cross-repo go-to-definition and references. The MCP server is **Enterprise-only**, so it is overkill for a personal org. Mention it only as the "precise cross-repo" ceiling.
- **Code knowledge graph MCPs** (CodeGraphContext, codebase-memory-mcp): Tree-sitter (+ optional SCIP) feeding a graph DB, served over MCP. **Experimental.**

*Generated code wikis (L2 candidates)*
- **deepwiki-open**: https://github.com/AsyncFuncAI/deepwiki-open — ~18.5k stars, MIT. A self-hosted DeepWiki clone with RAG and diagrams that needs embeddings + LLM providers (Ollama supported). Some forks add an MCP server with `get_wiki_summary` → `get_wiki_page` → `search_code`, which is itself a 3-level ladder. **Popular, but a heavy web app.**
- **OpenDeepWiki** (AIDotNet): ~3.4k stars, MIT, .NET. Incremental updates + MCP. **Moderate.**
- **CodeWiki** (FSoft-AI4Code, ACL 2026 Findings): ~1.8k stars, MIT. `codewiki generate` writes `docs/overview.md` + per-module .md + `module_tree.json`. That is a file-based L2 in-repo, which makes it the most git-friendly of the wiki generators. It beats DeepWiki by a modest 4.73% on its own benchmark, which uses LLM judges. **Research-grade, active.**
- **RepoAgent** (OpenBMB): ~0.9k stars, Apache-2.0, function/file-level docs incrementally updated on commit. **Stale (last release ~Dec 2024).**

*Documentation frameworks (L2 vocabulary)*
- **C4**: the only established framework with explicit zoom levels. Agent tooling exists as community skills (wshobson/agents `c4-code` → `c4-component` → `c4-container` → `c4-context`, built bottom-up). softaworks' skill writes `docs/architecture/c4-context.md`, `c4-containers.md` and `c4-components-{feature}.md`, and notes that "Context + Container diagrams are sufficient for most teams." An LLM multi-agent C4 paper exists (arXiv 2510.22787). **The framework is mature; AI mappings are community-grade.**
- **arc42**: 12 sections; section 5, "Building Block View", is itself hierarchical (level 1, 2, ...). Community AI toolkits exist (MSiccDev/arc42-toolkit). **Mature framework, experimental AI use.**
- **Diátaxis**: classifies by *reader intent* (tutorial/how-to/reference/explanation), not depth. That makes it orthogonal and useful for the *question-type* axis your ladder lacks. **Mature; AI mapping is blog-level only.**
- **ADRs**: they answer "why", which none of the zoom levels do. Keep them as a side branch off L2.

*Multi-repo / org level (L0)*
- **mani**: https://github.com/alajmo/mani — ~800 stars, MIT, Go single binary. `mani.yaml` holds project name, URL, **description** and tags, with `mani sync` and filtered tasks. **Stable, small project, low-but-alive activity.** Alternatives: myrepos (oldest), gita, gitopolis, Google `repo`. mani's per-project `desc` + tags is effectively a ready-made L0 schema.
- **Backstage `catalog-info.yaml`**: the de facto enterprise catalog descriptor (Component/System/API, owner, lifecycle, relations). It now has an MCP Actions backend (`catalog:query-catalog-entities` etc.) and community catalog→MCP graph servers. **Mature but heavyweight.** The *file format* is worth borrowing for L0 metadata even without running Backstage.

**3. What the research says about how deep to go**
- **ETH Zurich, "Evaluating AGENTS.md" (Gloaguen, Mündler, Müller, Raychev, Vechev; arXiv 2602.11988):** LLM-generated context files reduced resolution "by 0.5% and 2% on average on SWE-BENCH LITE and AGENTBENCH" with "a cost increase of 20% and 23%"; developer-written files gave "an increase of 4% on average" but still added 3.34 steps and raised cost by "at most 19%". Codebase overviews and directory listings didn't speed up navigation. **Implication:** L1 should hold non-inferable facts only. Don't auto-generate L1, and don't restate structure the agent can discover via L3 tools.
- **He, Zhao, Wang, Chen, "Is Progressive Disclosure All You Need for Long-Context Agents?" (arXiv 2607.17598):** On small corpora, strong agents just grep, so disclosure adds nothing. Once the corpus outgrows native navigation, one-level disclosure "becomes decisive." "A second, deeper routing level never helps and sometimes breaks accuracy outright, so one level is enough." **Implication:** across an *org* (many repos), L0 routing pays off. *Within* a repo, prefer one routing file that links straight to targets over multi-hop routers.
- **Codified Context (Vasilopoulos, arXiv 2602.20478):** In a 108,000-line C# system measured "across 283 development sessions", a ~660-line hot constitution + "19 specialized domain-expert agents" + "a cold-memory knowledge base of 34 on-demand specification documents" (~16k lines) + a trigger table worked. This is the closest published analogue to your router + knowledge/index design.

## Recommendations: a ladder built from existing parts

| Level | Content | Tool / convention | Generated or written | Descend when... |
|---|---|---|---|---|
| **L0 org index line** | One line per repo: name, purpose, tags, link to cached AGENTS.md | `mani.yaml` (`desc`, `tags`) in `skogai/monorepo`; render to `index.md` in llms.txt style (H1, blockquote, H2 sections, "Optional") | Written descriptions; render generated | The question names a capability/domain, not a repo |
| **L1 repo contract** | Build/test commands, boundaries, non-obvious conventions, pointers to L2 | AGENTS.md (≤ ~100–150 lines), cached into L0 via a `mani run` task | **Hand-written** | The question is about *how/why* something is structured, or the task spans >1 module |
| **L2 module / architecture** | C4 Context+Container (+ Component per module), ADRs for "why" | `docs/architecture/c4-*.md` + `docs/adr/`; optionally seed from CodeWiki then hand-edit | Written (generation as draft only) | You need to know *which symbols* to touch |
| **L3 symbol outline** | Signatures, classes, ranked map | Static: `ctags -R` or `repomix --compress` in a pre-commit/CI hook; Live: Serena `get_symbols_overview` → `find_symbol` | **Generated, deterministic** | You must read or change implementation |
| **L4 source** | Raw code, history | `git`, `rg`, Serena `find_symbol include_body=true` | — | — |

Concrete actions:
1. **Write a ~20-line `LOD.md` convention** (your one home-made piece), linked from every AGENTS.md. It should name L0–L4 and give *question-type → starting level* rules, e.g. "which repo does X" → L0; "how do I build/test" → L1; "where does X happen / how do parts interact" → L2; "what calls Y / what's the signature" → L3; "fix bug / change behavior" → L3→L4. Borrow Diátaxis intent categories for the question taxonomy. Nothing in the ecosystem covers this, and it is the piece that makes levels useful to agents.
2. **Flatten the in-repo router chain to one hop.** Make AGENTS.md link directly to `knowledge/index.md` sections and `docs/architecture/`, per arXiv 2607.17598. Keep SKOGAI.md/ROUTES.md only if they hold non-inferable content.
3. **Put each L2 doc's routing metadata in YAML front-matter** (`name`, `description`, `level: L2`), copying the Agent Skills shape. Agents already know how to treat descriptions as triggers, and a `rg '^description:'` gives you a free L2 index.
4. **Generate L3, never hand-write it.** Prefer universal-ctags (Unix-native, deterministic, diffable) or `repomix --compress` for file artifacts, and Serena for live queries. Gitignore the outputs or build them in CI, so they never go stale in git history.
5. **Use mani for L0**, with a task that copies each repo's AGENTS.md into `monorepo/cache/<repo>/AGENTS.md`. Borrow Backstage's `owner/lifecycle/system` keys only if you need relations.
6. **Treat generated wikis as optional.** deepwiki-open/OpenDeepWiki add servers, embeddings and staleness. CodeWiki is the only one that writes plain Markdown into the repo, so use it once as an L2 *draft*.

## Caveats
- Star counts are from late-2026 GitHub/aggregator snapshots and vary by a few percent between sources. Aider's ~50k stars are for the whole tool, not the repo map. Aider's development has visibly slowed (last push May 2026 per one tracker).
- Adoption figures for AGENTS.md ("more than 60,000 open source projects and agent frameworks", per the Linux Foundation's 9 Dec 2025 press release) come from OpenAI/the Linux Foundation and count repos containing the file, not usage quality. The "35–55% fewer bugs" figures circulating on blogs are unsourced; ignore them.
- ETH Zurich results come from SWE-bench-style Python tasks. The progressive-disclosure study used book QA, not code. Both point the same way, but neither tests a multi-repo org ladder directly.
- The Serena license is reported inconsistently (MIT vs GPL-3.0 for the core). Verify in the repo.
- The C4/arc42/Diátaxis "for AI" mappings found are community skills and blog posts, not maintained standards.
