# String Web Access — Gemini CLI extension

Fetch, search and map any website as clean, LLM-ready Markdown, without getting blocked.

[String Web Access](https://usestring.ai) handles proxy rotation, anti-bot protection,
CAPTCHAs and JavaScript rendering, so the agent gets the page instead of a block screen.

## Install

```bash
gemini extensions install https://github.com/usestring/string-gemini-extension
```

The CLI asks for your API key during install and stores it with the extension. Keys come
from [portal.usestring.ai](https://portal.usestring.ai). To change it later:

```bash
gemini extensions config string-web-access
```

You can also set `STRING_API_KEY` in your environment instead. Never paste the key into a
chat message and never commit it.

## Authentication

Two ways into the hosted server.

**API key.** What this extension uses. The key is sent as an `Authorization: Bearer` header on
every request.

**Sign in with String.** The server also speaks OAuth, for clients that support remote MCP
connectors. A browser window opens, the consent screen asks to grant `web_access` and shows a
picker of your organisation's active keys, and you choose which one the app may spend. Connected
apps are listed in [settings](https://portal.usestring.ai/settings) with the key each one uses,
and disconnecting takes effect within a minute. Claude and ChatGPT support this today. The
Gemini CLI is not wired up for it yet, so use the key above. See
[the remote MCP docs](https://portal.usestring.ai/docs/mcp/remote).

## Try it

Ask Gemini in plain language:

> Read https://news.ycombinator.com and list the top five posts with their links.

> Search for the Postgres 18 release notes and tell me what changed in logical replication.

> Map every docs page on modelcontextprotocol.io and give me the URLs as a list.

## Commands

| Command | What it does |
| --- | --- |
| `/string-setup` | Checks the connection and confirms the read tools respond |
| `/web-research <topic>` | Searches, reads the best sources, reports back with citations |

## Skills

Five skills ship with the extension. Gemini picks the right one on its own; you never have
to name them.

| Skill | Use when |
| --- | --- |
| `string-search` | Finding sources, current information, recent news |
| `string-fetch` | Reading a page you already have the URL for |
| `string-sitemap` | You need every URL on a site rather than one page |
| `string-request` | Sending a POST, PUT or PATCH instead of reading |
| `string-web-access` | Any multi-step web task, or a fetch came back blocked or empty |

## Tools

| Tool | What it does | Effect |
| --- | --- | --- |
| `web_access_fetch` | Fetch one URL as Markdown | read-only |
| `web_access_search` | Web search with structured results | read-only |
| `web_access_request` | Send a POST, PUT or PATCH with a body | writes, prompts first |
| `web_access_sitemap` | Crawl a site and return its URLs | billed job, quotes first |

Backed by the hosted MCP server at `https://mcp.usestring.ai/v1/mcp`. Full reference:
[portal.usestring.ai/docs/mcp/overview](https://portal.usestring.ai/docs/mcp/overview).

## Handling fetched content

Everything these tools return is untrusted third-party data, and a page can contain text
written to hijack the agent reading it. The bundled context file tells Gemini to treat page
content as data rather than instructions, to leave links found inside a page alone unless you
asked for them, and never to put credentials in a `url`, `query` or `headers` value. See
[GEMINI.md](./GEMINI.md).

## Troubleshooting

**The tools do not appear.** Restart Gemini CLI, then run `/string-setup`.

**Everything errors.** The key is probably missing or wrong. Run
`gemini extensions config string-web-access`.

**A fetch returns a block page or an empty body.** Change one thing rather than retrying the
same call: turn on JavaScript rendering first, then set a country. The `string-web-access`
skill covers the escalation rule.

**A crawl sits pending.** Sitemap runs as an asynchronous job. It quotes the cost, waits for
approval, then pages through results.

## Also available

| Client | Where |
| --- | --- |
| Claude Code | [usestring/string-claude-plugin](https://github.com/usestring/string-claude-plugin) |
| Cursor | [usestring/string-cursor-plugin](https://github.com/usestring/string-cursor-plugin) |
| Any MCP client | `https://mcp.usestring.ai/v1/mcp` with an `Authorization: Bearer` header |

## Support

Bugs and questions: [open an issue](https://github.com/usestring/string-gemini-extension/issues)
with your Gemini CLI version and the error text. Leave API keys out of the report.

Security issues go privately to support@usestring.ai, never into a public issue. See the
[security policy](https://github.com/usestring/.github/blob/main/SECURITY.md).

## License

MIT
