# Live Market Data Pipeline — ETL from yfinance into SQLite, with Scheduling

## What this project is

A small, real data engineering pipeline — not a one-off analysis notebook. It fetches real daily price data for TCS, Infosys, and Reliance, checks it for data quality problems, stores it in a local SQLite database without creating duplicates on re-run, and can be scheduled to run automatically every day.

## What's in this repo

| File | Description |
|---|---|
| `live_market_data_pipeline.ipynb` | The full notebook — all 7 pipeline steps, run against real data |
| `market_data.db` | The SQLite database this notebook creates (generated after you run it) |
| `README.md` | This file |

## The 7 steps

1. **Extract** — real daily OHLCV data via `yfinance` for TCS, Infosys, Reliance
2. **Validate** — checks for missing values, duplicate dates, and zero/negative prices, reporting anything found rather than silently dropping it
3. **Transform** — computes a daily return column
4. **Load** — inserts into SQLite using an upsert (`ON CONFLICT ... DO UPDATE`), so re-running the pipeline never creates duplicate rows — tested directly before this notebook was written
5. **Incremental fetch logic** — checks the latest date already stored per ticker and only fetches newer data on subsequent runs, instead of re-downloading full history every time
6. **Scheduling** — shown two ways: a `schedule`-library version (for a live demo process) and a real `cron` entry (for actual unattended scheduling on a server)
7. **Logging/monitoring** — every run writes a row to a `pipeline_log` table (timestamp, rows fetched/inserted, success or failure), so pipeline health is checkable after the fact

## Data — real only, no synthetic fallback

Real daily OHLCV data via `yfinance` for TCS.NS, INFY.NS, and RELIANCE.NS. **There is no synthetic data anywhere in the fetch step.** If the fetch fails (no internet), it raises a clear error that also gets logged to `pipeline_log` as a `FAILED` run — visible failure is itself part of what a real pipeline should do, not something to hide.

## Honest limitations, stated up front

1. **Colab is not production hosting.** The SQLite file only persists while the Colab session is alive, unless explicitly saved to Drive or downloaded. Real unattended scheduling (the `cron` example in Section 6) needs an actual always-on machine — a small server, a Raspberry Pi, a scheduled cloud function — not a notebook session.
2. **Single data source, single point of failure.** This relies entirely on Yahoo Finance via `yfinance`. A real production pipeline beyond a personal project would want a paid, SLA-backed vendor as a fallback, not just error logging.
3. **Three tickers, not a full market data warehouse.** Sized to demonstrate the pattern clearly — the same functions scale to more tickers with no structural changes needed.

## How to run

Fully Colab-compatible for the ETL logic itself. No local setup, no paid data required.

```bash
pip install yfinance schedule --quiet
```

Then run all cells top to bottom. Since there is no synthetic fallback, an internet connection is required to actually pull data — Colab has one by default.

**To actually schedule this for real, unattended, daily runs:** copy the pipeline functions into a standalone `.py` script and use the `cron` entry shown in Section 6 on an always-on machine — Colab itself is not suitable for this.

## Requirements

```
pandas
numpy
yfinance
schedule
sqlite3   # built into Python's standard library, no install needed
```
