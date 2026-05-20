# Research Methodology

Shared methodology for all agents with research capability. Used by: research-agent, security-reviewer, performance-reviewer, architecture-reviewer, user-experience-reviewer.

---

## Parallel Multi-Source Strategy

For any non-trivial query, use MULTIPLE tools in parallel (same turn) and synthesize the combined results. Do not stop after one tool returns — more sources = higher confidence. Exception: Website deep-dive (crawl) queries use a phased workflow instead — see that query type below.

### Tool Categories

| Tool | Best for | Freshness | Notes |
|------|----------|-----------|-------|
| Context7 MCP | Known libraries/frameworks | Good (tracks releases) | Resolve library ID first, then query |
| WebSearch | Broad queries, community discussion | Variable (DuckDuckGo/Bing index lag) | Good for "what are people saying" |
| Brave Search | Broad queries, alternative results | Variable (independent index, ~93% own crawler) | Different results from WebSearch — covers blind spots |
| mcp__fetcher__fetch_url | Official websites, JS-rendered pages | Best (live fetch) | Browser-based, most reliable |
| WebFetch | Simple/static pages, docs, APIs | Good (live fetch) | Lighter than fetcher, fails on JS-heavy sites |
| GitHub MCP | Repos, issues, PRs, releases, code search | Best (live API) | Conditional: only when query involves GitHub |
| Reddit MCP | Subreddit posts, comments, search | Best (live API) | Conditional: only when query involves Reddit |
| Crawl4AI (Bash) | Deep exploration of a specific website (many pages) | Best (live crawl) | Explicit user request only. Outputs markdown to disk. See agent definition for workflow. |

### How to Combine Tools Per Query Type

**Known library (React, Next.js, Supabase, etc.):**
- Fire in parallel: Context7 + WebSearch + Brave Search + fetcher (official site/blog)
- Synthesize: Context7 for API reference, search engines for community/recent, fetcher for latest blog/changelog
- If results reference a GitHub repo: fire GitHub MCP in follow-up turn

**Unknown/new tool (user provides URL or name):**
- Fire in parallel: mcp__fetcher__fetch_url (official site) + WebSearch + Brave Search
- Two independent search engines maximize chance of finding info on new tools
- If WebSearch AND Brave both return nothing, the fetcher result alone is sufficient
- Never conclude "doesn't exist" based on search engines alone

**General technical question (no specific library):**
- Fire in parallel: WebSearch + Brave Search
- If results seem dated (>3 months for fast-moving topics), supplement with direct fetches of official sources found in search results

**GitHub-specific query (repo, release, issue, code):**
- Fire GitHub MCP directly (structured API, most reliable for GitHub content)
- Supplement with WebSearch if broader context is needed

**Reddit-specific query (community discussion, sentiment, recommendations):**
- Fire Reddit MCP directly (search posts, read comments)
- Supplement with WebSearch if broader context is needed

**Website deep-dive (user explicitly requests crawling):**

Requires crawl4ai (Bash tool). Agents without Bash should use the "Unknown/new tool" query type instead, fetching the target site with mcp__fetcher__fetch_url.

When crawling is part of the research, switch from the default parallel strategy to a **phased approach**:

- **Phase 1 — Crawl**: Run crawl4ai on the target site. Read the `_INDEX.md` to see what was captured. If coverage gaps remain, run one follow-up crawl from a different starting URL — maximum 2 crawl passes per research task.
- **Phase 2 — Read**: Read the relevant crawled markdown files. Extract information that answers the research questions.
- **Phase 3 — Gap-fill**: Use FetchUrl, WebSearch, Brave ONLY for:
  - Pages the crawl missed (check the `_INDEX.md` — do NOT re-fetch URLs already crawled; normalize for trailing slashes, www prefixes, and HTTP/HTTPS when comparing)
  - Cross-domain sources (different websites for verification or additional perspectives)
  - Information the crawled site simply doesn't cover
- **Phase 4 — Synthesize**: Combine crawl data + gap-fill data into final output.

This phased workflow replaces the default parallel strategy ONLY when crawling is active. If crawling is not requested, use the normal parallel multi-source strategy above.

### Conditional Tool Use (Follow-Up Turns)

You operate in an agentic loop — you are NOT limited to one turn of tool calls. After observing results from parallel tools, you can fire additional tools:
- Results mention a GitHub repo → fire GitHub MCP to get structured data
- Results mention Reddit threads → fire Reddit MCP for post/comment details
- Results mention an official URL → fire fetcher to get latest content
- Results are ambiguous → fire a more targeted search query

---

## Freshness Rules

- Always check dates on search results. For AI/LLM/agent topics, results >3 months old are suspect.
- When search engines and a direct fetch disagree, prefer the direct fetch (it's live, not cached).
- When Context7 and a direct fetch disagree, prefer the direct fetch (Context7 index may lag).
- Report the date of each source in your output so the caller can judge freshness.

---

## Source Prioritization

When results conflict, trust in this order:
1. Live-fetched or live-crawled official documentation (mcp__fetcher__fetch_url / WebFetch / Crawl4AI of official site)
2. Context7 MCP documentation (indexed but may lag by days/weeks)
3. Official docs found via WebSearch (check publication date)
4. Primary sources (GitHub repos, release notes, RFCs — check date)
5. Reputable technical publications (check date for fast-moving topics)
6. Community resources (Stack Overflow, blogs) — use cautiously, always verify date

---

## Verification Process

- Cross-reference critical information across multiple sources
- Check publication/update dates to ensure currency
- Identify and flag conflicting information objectively
- Note when information cannot be verified or is speculative

---

## Research Principles

- Start with official sources and work outward
- Verify version numbers and compatibility information
- Note security considerations when relevant
- Include benchmark data or performance metrics when researching technical capabilities
- Distinguish between stable releases and beta/experimental features
