---
name: string-search
description: |
  Search the web with String Web Access and get structured Google results back. Use when the
  user wants to search the web, find articles or sources, look up current information, check
  recent news, or says "search for", "find me", "look up", "what's the latest on", "who is",
  or asks anything needing information from the live internet rather than training data.
  Returns organic results with position, title, URL, snippet, display URL and Google's source line;
  `searchType` searches Google's images, videos, shopping, books, places and forums tabs. Bypasses the
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

`query` is the only required parameter. Be specific and descriptive — this is a real Google
query, so everything you know about writing one applies.

## Search a Google tab

`searchType` picks the Google tab: `"web"` (the default, the ordinary results page), `"images"`,
`"videos"`, `"shopping"`, `"books"`, `"places"` or `"forums"`. There is no news tab. Each page
fetched is billed as one search, as on the web.

| `searchType` | Answers in | What you get |
| --- | --- | --- |
| `videos`, `books`, `forums` | `results` | Paged like the web. Each video carries `video` (`channel`, `platform`, `duration`) and each book `book` (`authors`, `published`). |
| `images` | `images` | About 100 a page, each with `title`, `url` (the page it is on), `source`, `imageUrl` (the file), `imageWidth`, `imageHeight` and `thumbnail`. It has 3 pages; `searchCount` up to 300 collects across them, usually about 270 images. |
| `shopping` | `products` | The one page of about 55 product cards: `title`, `productId`, `price` and `originalPrice` as shown, `merchant` and `moreMerchants`, `delivery`, `returns`, `rating`, `reviews`. A card has no `url`. Google does not page this tab, so `page` must be 1. |
| `places` | `places` | 20 businesses a page, paged like the web up to page 15. |

`results` is empty for images, shopping and places. `dateRange`, `sortBy` and `verbatim` apply to
web, videos and forums only; `format: "raw"` and `aiOverview` to web only. `searchType` does not
combine with `aiMode`.

```json
{ "query": "eames lounge chair", "searchType": "images", "searchCount": 150 }
```

## Filter a Google search

Five optional filters; none of them combines with `aiMode`:

| Field | What it does |
| --- | --- |
| `safeSearch` | `true` removes explicit results. |
| `includeOmittedResults` | `true` includes the results Google hides as very similar to ones shown. |
| `autocorrect` | `false` searches the query exactly as typed instead of Google's correction. On by default. |
| `restrictCountry` | A two-letter code that keeps only pages from that country. Unlike `country`, which sets where the search runs from, it filters the results. |
| `verbatim` | `true` matches the words exactly, without synonyms. Web, videos and forums only. |

```json
{ "query": "climate policy", "restrictCountry": "FR", "safeSearch": true }
```

## Narrow a Google search

Four optional fields apply to Google results only; none of them combines with `aiMode`:

| Field | What it does |
| --- | --- |
| `page` | The results page to start from, an integer from 1 to 30 (default 1), where page N is the page Google shows as N. Without `searchCount` the response is that one page, and with `format: "raw"` it always is; with `searchCount`, results are collected starting from that page. Together they stay within the first 300 results: (page - 1) × 10 + (`searchCount`, or 10 without it) must be at most 300. A page past Google's last result returns no results with `zeroResults: true`. A page holds about 8 to 10 results, so separate `page` calls can repeat or skip a result; for one list without repeats, make one call with `searchCount`. |
| `dateRange` | Limit results to a publication window: `"hour"`, `"day"`, `"week"`, `"month"` or `"year"` for the past hour through the past year, or `{ "from": "2024-01-01", "to": "2024-06-30" }` for a custom range of ISO dates (YYYY-MM-DD), inclusive. Either end is optional but at least one is required, and `from` must not be after `to`. |
| `sortBy` | `"relevance"` (the default) or `"date"` for the newest results first. |
| `format` | `"structured"` (JSON, the default) or `"raw"` (the Google results page as HTML, one page per call; `page` only, no `searchCount`; web only). We recommend structured. See [Raw HTML](#raw-html). |

`dateRange` and `sortBy` apply to web, videos and forums only. On the other tabs `page` follows
the tab: images stop at page 3, places at page 15, and shopping has page 1 only.

```json
{ "query": "heat pump grants", "page": 2, "dateRange": "month", "sortBy": "date" }
```

Reach for `dateRange` and `sortBy: "date"` when the question is about recent events, rather than
only adding a year to the query.

### Raw HTML

`format` is `"structured"` (JSON, the default) or `"raw"` (the Google results page as HTML). We
recommend structured; pass `format: "raw"` only when you need HTML:

```json
{ "query": "heat pump grants", "format": "raw", "page": 2 }
```

Raw returns one Google results page per call. It supports `page` only, not `searchCount`: a raw
call with `searchCount` is rejected (the API answers 400), so ask for page N with `page` and send
one call per page, each billed as one search. `dateRange` and `sortBy` work with raw.

The response is `html`, `htmlBytes` (the whole page's size) and `htmlTruncated`, in place of
`results` and the surfaces. The tool sends at most 60,000 bytes of markup per call, so a large
page can be cut; a cut page has `htmlTruncated: true`. For the whole page, call the HTTP API's
`POST /v1/search` with `"format": "raw"`.

## What comes back

With `searchType` `"images"`, `"shopping"` or `"places"`, the answer is `images`, `products` or
`places` as described in [Search a Google tab](#search-a-google-tab). Otherwise it is an array of
organic results, each with:

| Field | What it is |
| --- | --- |
| `position` | Place in this response, from 1 |
| `rank` | Google's own rank for the result, so the first result of `page` 3 is 21; Google only |
| `title` | Page title |
| `url` | Full URL — pass this to `web_access_fetch`. Absent when Google hid the destination; the result is still returned with its title and snippet |
| `snippet` | Google's extract, often enough on its own |
| `displayUrl` | The URL line Google shows (`https://site.com › a › b`); empty when Google shows none, as on Reddit and YouTube results |
| `displayText` | The source line Google shows under the title, verbatim: a URL, engagement counts such as `20+ comments · 3 months ago`, or other text |
| `source` | The site name Google shows, such as `Reddit · r/buildapc`, when it shows one |
| `video` | Videos tab only: `channel`, `platform`, `duration` |
| `book` | Books tab only: `authors`, `published` |

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

Search reads Google's results, not the pages behind them. If you need the content of a result,
that is a separate `web_access_fetch` call. A shopping card has no `url` to fetch.
