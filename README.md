# Research Agent

A Claude Code subagent that performs parallel multi-source technical research with source credibility tracking, structured output, and optional deep web crawling.

## Getting Started

Copy the contents of [SETUP.md](SETUP.md) and paste it into your Claude Code session. Claude will install the plugin and walk you through setup.

## Why Use This

When Claude Code needs to research something, it typically uses a single search tool and returns whatever it finds. This agent does it differently:

- **Parallel multi-source search** — fires Context7, WebSearch, Brave Search, and fetcher simultaneously, then synthesizes across all results
- **Source credibility tracking** — every source is tagged as official, primary, reputable, or community, with freshness dates
- **Structured JSON output** — returns a consistent schema with summary, key points, sources, discrepancies, and confidence level
- **Cross-referencing** — detects and reports conflicting information across sources instead of silently picking one
- **Optional deep web crawling** — crawl entire documentation sites via Crawl4AI when you need more than search results can provide

## Example Prompts

```
@research-agent What are the current best practices for rate limiting in Node.js APIs?
```
Returns: comparison of popular libraries (express-rate-limit, bottleneck, rate-limiter-flexible), with benchmarks and official docs.

```
@research-agent Is Bun 2.0 stable? What changed from 1.x?
```
Returns: release status from official sources, changelog highlights, community reception, any known issues.

```
@research-agent Crawl the Supabase docs on Row Level Security
```
Returns: crawled RLS documentation pages, synthesized with web search results for community patterns and gotchas.

## Install

Add this repo as a plugin marketplace, then install:

```
claude plugin marketplace add jiaweijwjw/research-agent
claude plugin install research-agent
```

### Post-Install Setup

After installing, paste the contents of [SETUP.md](SETUP.md) into your Claude Code session. Your Claude will walk you through setting up API keys and optional features interactively.

## Tools

| Tool | Included | Setup |
|------|----------|-------|
| WebSearch | Built-in | None |
| WebFetch | Built-in | None |
| Context7 | Auto-configured | None (free tier) |
| Fetcher | Auto-configured | Requires Node.js/npx |
| Reddit | Auto-configured | Requires Python/uvx |
| Brave Search | Manual (via SETUP.md) | Free API key from brave.com |
| GitHub MCP | Manual (via SETUP.md) | GitHub PAT or Copilot access |
| Crawl4AI | Manual (via SETUP.md) | Python + uv + Playwright |

3 MCP servers are auto-configured by the plugin. Brave Search, GitHub, and Crawl4AI are guided enhancements set up via SETUP.md.

## Output Format

The agent returns structured JSON:

```json
{
  "summary": "2-3 sentence overview of findings",
  "freshness_warning": null,
  "key_points": ["Finding 1", "Finding 2", "Finding 3"],
  "sources": [
    {
      "title": "Source title",
      "url": "https://...",
      "date": "2026-01-15",
      "credibility": "official",
      "freshness": "current"
    }
  ],
  "discrepancies": ["Any conflicting info across sources"],
  "confidence": "high"
}
```

## How It Works

The agent follows a parallel multi-source strategy: for any research query, it fires multiple tools simultaneously (e.g., Context7 for library docs + WebSearch + Brave Search + fetcher for official sites), then cross-references and synthesizes the combined results. Sources are prioritized by credibility (live-fetched official docs > indexed docs > search results > community content) and freshness. The full methodology is in `references/research-methodology.md`.

## Security Notes

- **Bash access**: The agent has Bash tool access, used exclusively for running the Crawl4AI script. This is instruction-enforced (the agent is told to only use Bash for crawl4ai), not permission-enforced. If you don't use Crawl4AI, you can remove `"Bash"` from the tools list in the agent definition for tighter permissions.
- **Web content**: The agent fetches and reads content from the web. Fetched content could theoretically contain prompt injection attempts. The agent's structured JSON output format and research-only scope limit the impact, but be aware when researching untrusted sites.
- **GitHub MCP**: If you set up GitHub MCP, use a read-only Personal Access Token with minimal scopes.

## License

MIT
