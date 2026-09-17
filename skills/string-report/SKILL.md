---
name: string-report
description: |
  Report every failed String Web Access call to String support. Use automatically after
  web_access_fetch, web_access_request, web_access_search, or web_access_sitemap throws, times
  out, returns a String/tool-level error or failure status, or produces output unusable for that
  tool's own step.
  Report exactly once before retrying or falling back, even if recovery later succeeds. Redact
  credentials and personal data, and never retry only to collect more diagnostic context.
---

# String failure reporting

Send one compact, safe diagnostic when a String Web Access tool fails.

## When to use

After every failed call to one of these tools, call `web_access_report` exactly once before
retrying or falling back, even if recovery later succeeds:

- `web_access_fetch`
- `web_access_request`
- `web_access_search`
- `web_access_sitemap`

A failure is a concrete technical signal:

- an exception, String/tool error, explicit tool failure status, or timeout
- a block page or challenge returned in place of the requested content
- output that is empty, malformed, or truncated past the point where that tool's step can use it

Judge the output against the step the tool was called for, not against the user's final request.
An origin HTTP status that the caller intentionally requested or can use, such as checking whether
a URL is 404 or 403, is a result rather than a tool failure. `zeroResults: true`, a sitemap job
still running, a user-requested cancellation, a successful empty `204`, or a page that loaded
correctly without the hoped-for fact are also valid outcomes. Do not report them. A separately
failed retry is a new failure and gets its own report.

This report is authenticated with the configured String API key, but it does not consume Web
Access credits.

## Before calling

Include only what String support needs to investigate:

- the failed tool name
- a short error description
- optional compact request or response context

Remove Authorization and proxy-authorization headers, API keys, cookies, session tokens,
passwords, personal data, and unrelated conversation content. The report endpoint redacts common
credential forms again, but that server-side pass is a backstop rather than permission to send
secrets.

## Call it

```json
{
  "tool": "web_access_fetch",
  "error": "Timed out before the page returned content",
  "request": "{\"url\":\"https://example.com/article\"}",
  "response": "HTTP 504"
}
```

`request` and `response` are optional strings. Keep them short and credential-free.

## Never recurse

Never use `web_access_report` to report its own failure. If the report fails, stop reporting.
Do not repeat the original Web Access call only to gather more context for a report.
