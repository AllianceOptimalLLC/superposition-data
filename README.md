# superposition-data

Daily public data for the SuperPosition Mac app.

This repo holds no code and no application secrets. It is a plain, public
data feed: a small JSON file describing the macro regime board (a sizing
posture, not a buy or sell signal) that SuperPosition draws on its regime
strip, refreshed each trading morning after the desk's daily scan.

## What is in here

- `regime/latest.json`, schema 1, the current board. Shape:

  ```json
  {
    "schema": 1,
    "generated": "2026-09-08T12:00:00.000Z",
    "asOf": "2026-09-08",
    "posture": "half",
    "counts": { "red": 1, "amber": 1, "green": 1 },
    "signals": [
      { "id": "trend", "label": "QQQ trend", "color": "#0ca30c", "value": "+9.1% vs 200-day", "note": "Green above, red below." }
    ],
    "moodComposite": { "score": 48, "rating": "neutral", "components": { "volatility": 58.7, "breadth": 45.7, "credit": 38.1 }, "asOf": "2026-09-04" }
  }
  ```

- `regime/YYYY-MM-DD.json`, a dated copy of the same document written on
  that trading day, kept for 90 days and pruned automatically after that.

## Update cadence

Updated each trading morning, after the desk's daily scan finishes and the
regime board is recomputed. Not updated intraday, and not updated on days
the scan does not run (market holidays, an offline desk).

## Not investment advice

The posture, the signal colors, and the mood composite score are a sizing
heuristic for one person's own desk, not a recommendation to buy, sell, or
hold anything. Nothing here is investment advice.

## No personal data

Every file in this repo is checked against a denylist (no positions, no
account numbers, no tickers, no file paths) before it is written. Nothing
here identifies any account, holding, or trade.

## License

Data files under `regime/` are licensed **CC BY 4.0** (Creative Commons
Attribution 4.0 International). See `LICENSE`. Attribution: "SuperPosition
regime data (AllianceOptimalLLC/superposition-data)".
