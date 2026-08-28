# String Web Access — Gemini CLI extension

Fetch, search and map any website as clean, LLM-ready Markdown, without getting blocked.

[String Web Access](https://usestring.ai) handles proxy rotation, anti-bot protection,
CAPTCHAs and JavaScript rendering, so the agent gets the page instead of a block screen.

## Install

```bash
gemini extensions install https://github.com/usestring/string-gemini-extension
```

Then set your API key:

```bash
export STRING_API_KEY=...
```

Keys come from [portal.usestring.ai](https://portal.usestring.ai).

## Tools

| Tool | What it does |
| --- | --- |
| `web_access_fetch` | Fetch one URL as Markdown |
| `web_access_request` | Send a POST, PUT or PATCH with a body to a URL |
| `web_access_search` | Web search with structured results |
| `web_access_sitemap` | Crawl a site and return its URLs |

`web_access_fetch` and `web_access_search` are read-only. `web_access_request` writes, and
`web_access_sitemap` creates billed crawl jobs, so both prompt before they run. Backed by the
hosted MCP server at `https://mcp.usestring.ai/v1/mcp`.

Documentation: https://portal.usestring.ai/docs/mcp/overview

## License

MIT
