# Research Agent — Setup Guide

You are helping a user set up the research-agent Claude Code plugin. This agent performs parallel multi-source technical research using up to 9 tools across 5 MCP servers. Walk the user through each step interactively — ask before proceeding with optional steps.

## What This Agent Does

The research-agent fires multiple search tools in parallel (Context7 for library docs, WebSearch, Brave Search, fetcher for official sites), cross-references the results, tracks source credibility, and returns structured JSON with findings, sources, discrepancies, and a confidence level. It can optionally deep-crawl entire documentation sites via Crawl4AI.

## Step 1: Check Prerequisites

Before installing, verify these are available:

- **Node.js 18+ and npx**: Required for the fetcher MCP server. Check with: `node --version && npx --version`
- **Python 3.10+ and uvx**: Required for the reddit MCP server. Check with: `python3 --version && uvx --version`

If either is missing, let the user know which tools will be unavailable and ask if they want to continue anyway. The agent still works with the remaining tools.

## Step 2: Install the Plugin

Run these two commands:

```bash
claude plugin marketplace add jiaweijwjw/research-agent
claude plugin install research-agent
```

If the user has already installed the plugin, skip to Step 3.

## Step 3: Verify MCP Servers

The plugin auto-configures 3 MCP servers. Check that they connected:

- **context7** — library and framework documentation
- **fetcher** — URL fetching and parsing (requires Node.js)
- **reddit** — Reddit posts and comments (requires Python/uvx)

If any server failed to connect, help the user troubleshoot:
- fetcher: likely needs Node.js/npx installed
- reddit: likely needs Python/uvx installed
- context7: HTTP server, should work if internet is available

Also check if the user already has any of the optional MCP servers configured (brave-search, github). If they do, note which are already set up and skip those steps below.

## Step 4: Brave Search (Recommended)

Brave Search provides an independent search index that covers blind spots from the built-in WebSearch. This is the most impactful enhancement — it roughly doubles the agent's search coverage.

Ask the user if they want to set up Brave Search. If yes:

1. Direct them to https://brave.com/search/api/ to sign up for a free account
2. The free tier provides 2,000 queries per month — more than enough for typical research use
3. Once they have their API key, run:

```bash
claude mcp add --scope user -e BRAVE_API_KEY=<their-api-key> brave-search -- npx -y @brave/brave-search-mcp-server
```

Replace `<their-api-key>` with the actual key the user provides.

## Step 5: Context7 API Key (Optional)

Context7 works without an API key on a free tier (~500 requests/month). A free API key raises the rate limit.

Ask the user if they want higher rate limits. If yes:

1. Direct them to https://context7.com to create a free account
2. Copy the API key from their dashboard
3. Help them configure the key for the context7 MCP server

If they're fine with the default limits, skip this step.

## Step 6: GitHub MCP (Optional)

Adds the ability to search code, repositories, issues, and pull requests directly via GitHub's API. This is useful for researching open-source projects, finding implementation examples, or checking issue histories.

Ask the user if they want GitHub integration. If yes:

1. They need a GitHub Personal Access Token. Direct them to https://github.com/settings/tokens
2. Recommend creating a fine-grained token with **read-only** permissions (Contents: read, Issues: read, Pull requests: read)
3. Once they have the token, run:

```bash
claude mcp add --scope user --transport http --header "Authorization: Bearer <their-pat>" github https://api.githubcopilot.com/mcp
```

Replace `<their-pat>` with the actual token the user provides.

## Step 7: Crawl4AI (Optional)

Adds the ability to crawl entire documentation sites and extract structured content. This is a power-user feature for when search results aren't enough and you need to systematically read through a site.

Requires: Python 3.10+, uv package manager.

Ask the user if they want deep crawling capability. If yes:

1. Clone the crawl4ai tool:
```bash
git clone https://github.com/jiaweijwjw/crawl4ai-tool.git ~/code/tools/crawl4ai
```

2. Install dependencies:
```bash
cd ~/code/tools/crawl4ai && uv sync
```

3. Install Playwright browser:
```bash
cd ~/code/tools/crawl4ai && uv run playwright install
```

4. The agent expects crawl4ai at `~/code/tools/crawl4ai/`. If the user cloned to a different location, help them update the path in the agent definition file. The path appears in several places in `agents/research-agent.md` — search for `~/code/tools/crawl4ai` and replace with their actual path.

If the user doesn't want Crawl4AI, mention that they can optionally remove `"Bash"` from the tools list in the agent definition for tighter permissions (Bash is only used for crawl4ai).

## Step 8: Verify

Test the agent with a sample research query:

```
@research-agent What are the current best practices for rate limiting in Node.js APIs?
```

Check that:
- The agent returns a valid JSON object
- The `sources` array contains entries with URLs and credibility ratings
- Multiple tools were used (you should see Context7, WebSearch, and any configured search tools firing)

If the agent doesn't appear, the user may need to restart Claude Code for the plugin to take effect.

---

## Manual Installation (Alternative)

If the user prefers not to use the plugin system, they can install the agent manually:

1. Download the agent definition and methodology files from the repository:
   - `agents/research-agent.md` → copy to `~/.claude/agents/research-agent.md`
   - `references/research-methodology.md` → copy to `~/.claude/references/research-methodology.md`

2. In the copied `~/.claude/agents/research-agent.md`, find and replace the methodology path:
   - Change `${CLAUDE_PLUGIN_ROOT}/references/research-methodology.md` to `~/.claude/references/research-methodology.md`
   - This appears in two places in the file (search for `CLAUDE_PLUGIN_ROOT`)

3. Add the MCP servers individually:
```bash
claude mcp add --scope user --transport http context7 https://mcp.context7.com/mcp
claude mcp add --scope user fetcher -- npx -y fetcher-mcp
claude mcp add --scope user reddit -- uvx --from git+https://github.com/adhikasp/mcp-reddit.git mcp-reddit
```

4. Follow Steps 4-7 above for Brave Search, Context7 API key, GitHub MCP, and Crawl4AI.
