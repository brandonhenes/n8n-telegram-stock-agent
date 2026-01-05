# n8n-telegram-stock-agent
Telegram → market data + scorecard → investable brief (n8n, FMP, LLM)

Telegram Stock Brief Agent (n8n + Market Data + Scorecard)

What it does
Turn a stock ticker into a Telegram-friendly, 60–90 second investable brief with:

Fundamentals

Price context

Catalysts/news

Scenarios (bear/base/bull)

An action plan

A numeric score so you can rank opportunities consistently

Why it matters
Most “AI stock bots” are unstructured and inconsistent. This workflow produces repeatable, phone-readable output with a stable schema, while handling messy real-world data (missing fields, thin news, rate limits).

Input (Telegram)
Dead simple: one line, comma-separated.

Examples

DAL

DAL, horizon=90d, risk=medium, style=value, max_articles=10 (optional parsing logic)

Output schema (Telegram brief)

Snapshot (price / trend / next earnings date)

Scorecard (overall + sub-scores)

Fundamentals (valuation + quality + leverage/liquidity)

Price context (trend, 52w position, MA50/MA200)

Catalysts (30–90d)

Scenarios (bear/base/bull) with ranges + rough probabilities

Plan (buy zone, don’t-buy-above, profit-take, invalidation)

Key risks

Data gaps / next fetches

Example output (DAL)
See: examples/DAL.md

Scorecard (0–100)
The score exists so you can compare tickers on the same rubric.

Weights

Valuation: 25% (P/E, EV/EBITDA, FCF yield)

Quality: 20% (margins, ROIC/ROE)

Balance sheet risk: 20% (net debt/EBITDA, interest coverage, Altman Z, liquidity)

Catalyst strength: 15% (earnings timing + catalyst list completeness)

Price context: 15% (trend, MA alignment, 52w position)

Data completeness: 5% (penalty for missing key fields)

Interpretation

85–100: A-tier (rare; strong setup + clean data)

70–84: B-tier (good candidate; needs one more confirm)

55–69: C-tier (watchlist only)

<55: avoid / too many red flags / too much missing data

Architecture (high level)
Telegram Trigger → Parse Input → Fan-out fetches → Normalize/Pack → Scorecard → LLM Brief → Telegram response

Key workflow stages

Price: pull 1d/1w/1m candles → merge → compute MAs and trend context

Fundamentals: quote, profile, key metrics, ratios, growth, scores, balance sheet, earnings calendar → merge → summary object

News: fetch articles → trim → LLM summary

Pack: combine price + fundamentals + news into a consistent object

Scorecard: deterministic scoring with sub-scores + data completeness penalty

Agent: generates final brief in a consistent Telegram-readable format

Data sources
Financial Modeling Prep (FMP) stable endpoints used:

/stable/quote

/stable/profile

/stable/key-metrics-ttm

/stable/ratios-ttm

/stable/financial-growth

/stable/financial-scores

/stable/balance-sheet-statement?period=quarter&limit=1

/stable/earnings

News note
News providers vary. Some FMP legacy news endpoints are gated on newer plans. A dedicated news API (e.g., NewsAPI) is recommended.

Reliability behaviors

Continue-on-fail for non-critical fetches

Null handling: missing fields are treated as missing and surfaced in “Data gaps”

Structured packing: consistent schema regardless of partial failures
Recommended next hardening

Retry/backoff on HTTP nodes (rate limits)

Caching for repeated requests (10–30 min)

Run logging (inputs, outputs, error flags)

How to run

n8n (cloud or self-hosted)

Telegram bot token

OpenAI API key

Market data API key(s) (FMP, plus any price/news providers you use)

Security / privacy

Do not commit API keys or Telegram identifiers.

Sanitize workflow exports (credentials removed).

Output is educational only, not financial advice.
