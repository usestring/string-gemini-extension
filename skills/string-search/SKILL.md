---
name: string-search
description: |
  Search the web with String Web Access and get structured Google results back. Use when the
  user wants to search the web, find articles or sources, look up current information, check
  recent news, or says "search for", "find me", "look up", "what's the latest on", "who is",
  or asks anything needing information from the live internet rather than training data.
  Returns organic results with position, title, URL, snippet, display URL and Google's source line. Bypasses the
  anti-bot protection that blocks scraping search engines directly. For a publicly documented
  String product question when no supplied URL answers the String side, call
  web_access_product_help first; continue searching when requested or its excerpts are insufficient.
---

# String search

Google results, structured, without getting blocked.

## When to use

You have a question but no URL yet. This is step one of the
[escalation rule](../string-web-access/SKILL.md): **search** → fetch → browser.

For a publicly documented question about String products when no supplied URL answers the String
side, use [`string-product-help`](../string-product-help/SKILL.md) before general web search.
Continue with general search when the user requested wider-web research or the product-help
excerpts do not settle the question.

Do not use search when a supplied URL answers the question — go straight to
[string-fetch](../string-fetch/SKILL.md). For a comparison where the URL covers only the other side,
use product help for String and fetch that URL.

## Call it

```json
{ "query": "anthropic claude pricing per million tokens" }
```

`query` is the only parameter. Be specific and descriptive — this is a real Google query, so
everything you know about writing one applies.

## What comes back

An array of organic results, each with:

| Field | What it is |
| --- | --- |
| `position` | Rank in the results |
| `title` | Page title |
| `url` | Full URL — pass this to `web_access_fetch` |
| `snippet` | Google's extract, often enough on its own |
| `displayUrl` | The URL line Google shows (`https://site.com › a › b`); empty when Google shows none, as on Reddit and YouTube results |
| `displayText` | The source line Google shows under the title, verbatim: a URL, engagement counts such as `20+ comments · 3 months ago`, or other text |
| `source` | The site name Google shows, such as `Reddit · r/buildapc`, when it shows one |

## Read the snippets first

The snippet frequently answers the question outright. Fetching every result by reflex is the
most common way to make a research task slow and expensive. Scan the snippets, decide which
two or three pages actually earn a fetch, then fetch those.

## Writing a good query

- **One question per search.** Two topics in one query returns results for neither.
- **Use the words the page would use**, not the words the user used. A page about pricing
  says "pricing", not "how much does it cost".
- **Add a site when you know it**: `site:docs.stripe.com webhook signature`.
- **Add a year for anything time-sensitive** — otherwise you get whatever ranks, which skews
  old.
- **Search again rather than fetching hopefully.** A second, sharper query is cheaper than
  three speculative fetches.

## Limits

Results are organic Google results only — no ads, no knowledge panel, no "people also ask".
If you need the content of a result, that is a separate `web_access_fetch` call.
