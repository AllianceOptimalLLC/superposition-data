# superposition-data

Daily public data for the SuperPosition Mac app.

This repo holds no code and no application secrets. It is a plain, public
data feed: small JSON files describing the macro regime board (a sizing
posture, not a buy or sell signal) and a macro economic calendar that
SuperPosition draws on its regime strip and calendar screen, refreshed each
trading morning after the desk's daily scan, both in the same commit.

## What is in here

- `regime/latest.json`, schema 1, the current board. Shape:

  ```json
  {
    "schema": 1,
    "generated": "2026-09-08T12:00:00.000Z",
    "asOf": "2026-09-08",
    "posture": "half",
    "counts": { "red": 1, "amber": 1, "green": 1 },
    "omitted": ["fearGreed", "aaii"],
    "signals": [
      { "id": "trend", "label": "QQQ trend", "color": "#0ca30c", "value": "+9.1% vs 200-day", "note": "Green above, red below." }
    ],
    "moodComposite": { "score": 48, "rating": "neutral", "components": { "volatility": 58.7, "breadth": 45.7, "credit": 38.1 }, "asOf": "2026-09-04" }
  }
  ```

- `regime/YYYY-MM-DD.json`, a dated copy of the same document written on
  that trading day, kept for 90 days and pruned automatically after that.

- `macro/latest.json`, schema 1, a rolling macro economic calendar (CPI, PPI,
  PCE, jobs, GDP, FOMC, initial claims). No dated copies: the file is already
  a rolling window (see below), so "latest" is the whole useful history.
  Shape:

  ```json
  {
    "schema": 1,
    "generatedAt": "2026-09-11T12:00:00.000Z",
    "events": [
      {
        "id": "cpi-2026-09-15",
        "series": "cpi",
        "name": "Consumer Price Index (August 2026)",
        "releaseAt": "2026-09-15T08:30:00-04:00",
        "importance": "high",
        "prior": { "period": "2026-07", "mom": 0.1, "yoy": 3.1 },
        "actual": null,
        "source": "bls"
      }
    ],
    "sources": "BLS (CPI, PPI, employment) API v2 and schedule pages, the BEA release schedule (GDP, PCE), and the Federal Reserve's FOMC calendar: all free, official U.S. government sources..."
  }
  ```

  `events` holds every event in the next 60 days rated `high` or `medium`
  importance, plus every event in the last 30 days that already has a
  non-null `actual`, sorted chronologically (history first, then upcoming).
  Each event carries exactly eight fields, `id`, `series`, `name`,
  `releaseAt`, `importance`, `prior`, `actual`, `source`, and nothing else:
  a whitelist copy, re-checked before every write. `prior`/`actual` are
  shaped per series (percent changes for CPI/PPI, level changes for jobs);
  they are null for PCE, GDP and FOMC, which have no free keyless actuals
  feed. `source` is one of `bls`, `bea`, `fed`, `computed`, or `static`
  (the bundled fallback table). The file is capped at 200 KB; a payload that
  would exceed that is refused and the previous day's file is left in place.

## Eight signals, not ten

The desk's own board computes ten signals, but two of them are derived from
licensed third-party data this feed is not allowed to redistribute: CNN's
Fear and Greed index (`fearGreed`) and the AAII bull/bear survey spread
(`aaii`). Those two are dropped before this file is ever written. `signals`
holds the other eight, `counts` is retallied from those eight (so it always
sums to `signals.length`), and the top-level `omitted` field names the two
that are missing so a consumer knows why the count is eight, not ten.
`posture` is unaffected: it is a desk-wide sizing call computed upstream off
all ten signals, licensed ones included, and is carried over unchanged.

## Update cadence

Updated each trading morning, after the desk's daily scan finishes and the
regime board and macro calendar are recomputed. `regime/latest.json` and
`macro/latest.json` land in the SAME commit, so the two files never drift out
of sync with each other. Not updated intraday, and not updated on days the
scan does not run (market holidays, an offline desk). A macro-only refusal
(the 200 KB size guard, a malformed source file on the desk) skips only
`macro/latest.json` for that day; the regime files still publish.

## Not investment advice

The posture, the signal colors, the mood composite score, and the macro
calendar's `prior`/`actual` figures are a sizing heuristic and a reference
calendar for one person's own desk, not a recommendation to buy, sell, or
hold anything, and not a forecast. Nothing here is investment advice.

## No personal data

Every file in this repo is checked against a denylist (no positions, no
account numbers, no tickers, no file paths) before it is written. Nothing
here identifies any account, holding, or trade. The same check also refuses
to publish if either licensed signal id (`fearGreed`, `aaii`) ever turns up
in the `signals` array, as a second line of defense behind the drop
described above. `macro/latest.json`'s events additionally go through a
WHITELIST (only `id`, `series`, `name`, `releaseAt`, `importance`, `prior`,
`actual`, `source` are ever copied off the desk's own calendar) and a 200 KB
size cap before they are written.

## Fetching this data

Both files are served two ways, free and requiring no auth:

- `https://raw.githubusercontent.com/AllianceOptimalLLC/superposition-data/main/regime/latest.json`
- `https://raw.githubusercontent.com/AllianceOptimalLLC/superposition-data/main/macro/latest.json`
- `https://cdn.jsdelivr.net/gh/AllianceOptimalLLC/superposition-data@main/regime/latest.json`
- `https://cdn.jsdelivr.net/gh/AllianceOptimalLLC/superposition-data@main/macro/latest.json`

jsDelivr is a CDN in front of this repo and can lag a fresh push by several
minutes to a few hours; prefer raw GitHub when freshness matters and jsDelivr
for lighter-weight, cached fetches (e.g. every install of SuperPosition,
including store copies with no access to the desk this data comes from).

## License

Data files under `regime/` and `macro/` are licensed **CC BY 4.0** (Creative
Commons Attribution 4.0 International). See `LICENSE`. Attribution:
"SuperPosition regime and macro calendar data
(AllianceOptimalLLC/superposition-data)".
