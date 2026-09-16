---
name: string-search
description: |
  Search the web with String Web Access and get structured Google results back. Use when the
  user wants to search the web, find articles or sources, look up current information, check
  recent news, or says "search for", "find me", "look up", "what's the latest on", "who is",
  or asks anything needing information from the live internet rather than training data.
  Returns organic results with position, title, URL, snippet and display URL, plus whatever
  Google rendered around them: knowledge panel, AI overview with cited sources, local pack,
  People also ask, related searches, videos, discussions. Bypasses the anti-bot protection
  that blocks scraping search engines directly.
---

# String search

Google results, structured, without getting blocked.

## When to use

You have a question but no URL yet. This is step one of the
[escalation rule](../string-web-access/SKILL.md): **search** → fetch → browser.

Do not use it when you already have the URL — go straight to
[string-fetch](../string-fetch/SKILL.md) and save a round trip.

## Call it

```json
{ "query": "anthropic claude pricing per million tokens" }
```

`query` is required. Be specific and descriptive — this is a real Google query, so everything
you know about writing one applies.

`searchCount` is optional: how many organic results you want, an integer from 1 to 50 (above 50
is rejected). Google is paged, up to 10 pages, until that many are in hand, and each page is
billed as one search. Omit it for one page, about 10 results.

```json
{ "query": "construction consulting firms Ohio", "searchCount": 30 }
```

## What comes back

`results`, the ranked organic documents, each with:

| Field | What it is |
| --- | --- |
| `position` | Rank in the results |
| `title` | Page title |
| `url` | Full URL — pass this to `web_access_fetch` |
| `snippet` | Google's extract, often enough on its own |
| `displayUrl` | The breadcrumb Google shows |

`zeroResults` is `true` when Google itself reported that nothing matched — broaden the query
rather than retry.

Every other field is a surface Google rendered around the documents. Each is present only when
the page carried it, is never merged into `results`, and only Google returns it:

| Field | What it carries |
| --- | --- |
| `entity` | The knowledge panel for the one business or person the query named: `title`, `subtitle`, `description` and its `descriptionSource`, `rating`, `reviews`, `website`, labelled `attributes` (address, phone, hours), social `profiles`. |
| `places` | Local-pack business listings: `name`, `category`, `rating`, `reviews`, `address`, `phone`, `hours`, `url`, `mapsUrl`. |
| `overviews` | Google's AI overviews. The first entry with no `topic` is the query's own summary; entries with a `topic` and `question` are the "Things to know" tabs; `declined: true` marks a frame Google did not fill. Each has `text` and cited `sources` as `{ title, url }`. |
| `peopleAlsoAsk` | `{ question }` entries; answers are not on the page. |
| `relatedSearches` | Query strings Google suggests. |
| `answers` | One widget under `localTime`, `currency`, `unitConversion`, `weather`, `translation`, `sports` or `flights`. |
| `spelling` | `{ kind, query, asked }` — `substituted` means the results are for the corrected query, `suggested` means they are for the query as typed. |
| `ads`, `videos`, `shortVideos`, `discussions`, `images`, `sitelinks` | Ranked blocks, each entry with `position`, `title` and `url`. |
| `paging` | `{ pages, complete }`, only when `searchCount` was sent. `pages` is how many results pages answered, each billed as one search. `complete: false` means the search was cut short (a later page could not be fetched, or the time budget ran out before `searchCount`) and `results` holds what was collected; fewer results with `complete: true` means Google had no more, the ten-page cap was reached, or the first page carried no organic results (a local pack or knowledge panel alone is not paged). Surfaces describe the first page only; positions run on across pages. |

A query naming a single business often comes back with `results` empty and the answer in
`entity` or `places` — read those before treating an empty `results` as no answer.

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

An overview is Google's summary, not a source: cite and read its `sources`, not the summary.
If you need the content of a result, that is a separate `web_access_fetch` call.
