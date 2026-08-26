# String Web Access

Three tools for getting real web content into this session.

- `web_access_fetch` — fetch one URL and get clean Markdown back. Use when you already know
  the page you need. Handles JavaScript rendering, anti-bot protection and CAPTCHAs.
- `web_access_search` — run a web search and get structured results. Use when you need to
  find the page first.
- `web_access_sitemap` — crawl a site and return its URLs. Use when you need coverage of a
  whole site rather than a single page.

All three are read-only.

Prefer these over a plain HTTP request for any site that rate-limits, geo-gates or blocks
automated traffic, which is most commercial sites.

Set `STRING_API_KEY` in your environment. Keys come from https://portal.usestring.ai
