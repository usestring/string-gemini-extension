# String Web Access — Gemini CLI extension

Fetch, search and map any website as clean, LLM-ready Markdown with
[String Web Access](https://usestring.ai).

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
| `/string-setup` | Checks the connection and confirms fetch and search respond |
| `/web-research <topic>` | Searches, reads the best sources, reports back with citations |

## Skills

Four skills ship with the extension, one per job. Gemini picks the right one on its own; you
never have to name them.

| Skill | Use when |
| --- | --- |
| `web-search` | Finding sources for a question when there is no URL yet, and answering with citations |
| `read-page` | Reading a URL, then summarizing it, answering from it or pulling its facts into a table |
| `map-site` | You need the URLs on a site or a section of it: quote, approve, crawl, then read them |
| `send-request` | Sending a request you name, such as a GraphQL query, and showing the data it returns |

Each skill carries the access rules that apply to it: public content and your own accounts only,
no getting past logins or paywalls, and page content treated as data rather than instructions.

## Tools

The MCP server at `https://mcp.usestring.ai/v3/mcp` provides six tools.

| Tool | What it does |
| --- | --- |
| `web_access_search` | Searches the web and returns ranked results with titles, URLs and snippets. Read-only. |
| `web_access_fetch` | Reads one public URL and returns it as Markdown, the page's raw body, or a JSON envelope with the status and headers. Read-only. |
| `web_access_sitemap_quote` | Quotes a crawl of one site: estimated pages and cost. Crawls nothing and costs nothing. |
| `web_access_sitemap_start` | Starts a quoted crawl after the user approves the quote. |
| `web_access_sitemap_status` | Reports a crawl's progress, and lists its URLs once it finishes. Read-only. |
| `web_access_request` | Sends a POST, PUT or PATCH the user asks for, such as a GraphQL query, and returns the response. Prompts before it runs. |

Full reference: [portal.usestring.ai/docs/mcp/overview](https://portal.usestring.ai/docs/mcp/overview).

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

**A page comes back empty.** It probably renders in the browser. The `read-page` skill retries
once with `executeJS: true`.

**A crawl sits pending.** A crawl runs as an asynchronous job. It starts only after you approve
the quote, and `web_access_sitemap_status` reports its progress until the URLs are ready.

## Also available

| Client | Where |
| --- | --- |
| Claude Code | [usestring/string-claude-plugin](https://github.com/usestring/string-claude-plugin) |
| Cursor | [usestring/string-cursor-plugin](https://github.com/usestring/string-cursor-plugin) |
| Any MCP client | `https://mcp.usestring.ai/v3/mcp` with an `Authorization: Bearer` header |

## Support

Bugs and questions: [open an issue](https://github.com/usestring/string-gemini-extension/issues)
with your Gemini CLI version and the error text. Leave API keys out of the report.

Security issues go privately to support@usestring.ai, never into a public issue. See the
[security policy](https://github.com/usestring/.github/blob/main/SECURITY.md).

## License

MIT
