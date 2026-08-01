---
name: Search and retrieve regulatory documents
description: >-
  Find regulatory or clinical documents in Rhizome AI by keyword, then pull the
  full page-level text for the most relevant hit.
api: openapi/rhizome-ai-openapi.yml
operations:
- searchDocuments
- getDocumentContents
---

# Search and retrieve regulatory documents

Use the Rhizome AI API to answer a regulatory question with cited, primary-source text.

## Prerequisites
- An API key (create one at https://rhizomeai.com/api-keys; enterprise plan required).
- Send the key in the `x-api-key` header on every request.
- Base URL: `https://rhizomeai.com/api`. All requests are JSON `POST`.

## Steps

1. **Search** — call `searchDocuments` (`POST /search`) with `query_text`. Optionally
   narrow with `dataset_to_search`, `year_from`, `year_to`, and page with `top_k`
   (1-100, default 20) and `offset`. Results come back ranked by BM25 `score`; each
   result carries `doc_id`, `page_num`, `name`, `company`, `year_decision`, `text`,
   and `highlight`.

2. **Pick a document** — choose the result with the best `score`/`highlight` and keep
   its `doc_id`.

3. **Retrieve contents** — call `getDocumentContents` (`POST /contents`) with that
   `doc_id`. Page through with `top_k` (default 10, max 100) and `offset`; read
   `pages[].text` and cite `page_num`. Use `total_pages` to know when to stop.

## Rules
- Respect the rate limit: 50 requests/minute per key. On `429`, back off exponentially
  (1, 2, 4, 8, 16s).
- Handle `401` (bad/missing key), `400` (bad body), `404` (unknown `doc_id`) explicitly.
- There is no idempotency key; both operations are read-only, so safe to retry.
- Always cite `doc_id` + `page_num` + source `url` when reporting an answer.
