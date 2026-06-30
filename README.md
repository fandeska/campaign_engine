# CRM Campaign Target Engine

A reusable Python engine that turns raw CRM transaction data into a prioritized,
ready-to-send client list for WhatsApp/email campaigns in luxury retail.

Built for a multi-brand watch and jewelry retailer to replace manual Excel-based
campaign targeting with a configurable, repeatable pipeline.

## What it does

Given a target brand, series, and price range, the engine automatically:

- Identifies candidate clients from two sources: existing buyers of the target
  brand (who haven't bought the target series yet) and clients with purchase
  history in cross/affinity brands
- Scores each client's spending power using per-visit transaction data
  (not just per line-item), so multi-item purchases aren't undercounted
- Builds an RFM (Recency, Frequency, Monetary) segmentation to classify
  clients into Champions / Loyal / Potential / At Risk / Lost
- Excludes clients blasted within a configurable cooldown window, to avoid
  over-contacting the same people across campaigns
- Auto-detects realistic price thresholds from the series' own purchase
  history (P25–P75) instead of requiring manual price input
- Calculates a propensity score (0–100) per client and ranks them into
  priority tiers: Top Priority, High, Medium, Low
- Exports a formatted, color-coded Excel file with a target list, summary
  breakdown, and the full configuration used — ready to hand off to the
  marketing team
- Updates a running blast history log automatically after every run

## Why

CRM campaign targeting in luxury retail typically means manually filtering
spreadsheets by brand, price tier, and recency — a process that's slow,
inconsistent between analysts, and easy to get wrong (e.g. accidentally
re-targeting clients blasted last week). This engine encodes that logic once,
so each new campaign is just a matter of changing a handful of config values.

## How it works

```
Edit CAMPAIGN CONFIGURATION (brand, series, price, filters)
                ↓
   Load transaction data + blast history
                ↓
   Auto-detect price range from purchase history
                ↓
   Build RFM segments + client spending profiles
                ↓
   Identify candidates (brand buyers + affinity buyers)
                ↓
   Apply filters (spending power, recency, frequency,
   gender, client type, RFM segment, blast cooldown)
                ↓
   Score & rank by propensity to purchase
                ↓
   Export Excel target list + update blast history
```

## Usage

1. Place your transaction CSV and `blast_history.csv` in the working directory
2. Edit the **CAMPAIGN CONFIGURATION** block at the top of the script —
   this is the only section that changes between campaigns:

```python
CAMPAIGN_NAME   = "Tudor Monarch — June 2026"
TARGET_BRAND    = "Tudor"
TARGET_SERIES   = "Monarch"
CROSS_BRANDS    = ["TAG Heuer", "Breitling", "Bell & Ross"]
SEGMENTS        = ["Champions", "Loyal", "Potential"]
MAX_LIST_SIZE   = 500
```

3. Run the script. Output:
   - `campaign_target_<name>_<date>.xlsx` — the target list, ready to send
   - `blast_history.csv` — automatically updated with this run's contacts

## Key design decisions

- **Per-visit spending, not per-item**: a single visit with 3 items at
  Rp 30M each reads as Rp 90M spending power, not Rp 30M — this matters
  for accurately gauging whether a client can afford a given product.
- **Auto-detected price thresholds**: rather than manually guessing a
  product's price bracket, the engine derives it from the P25–P75 range
  of actual historical transactions for that series, which is more robust
  to outliers than min/max.
- **Configurable spending power logic**: a client qualifies if they meet
  any one of three signals (max single-visit spend, average visit spend,
  or total lifetime spend) — accounting for different client behaviors
  (big one-time spenders vs. frequent moderate spenders).

## Tech stack

Python · pandas · numpy · XlsxWriter

## Status

Actively used and iterated on for live campaign targeting. Configuration
parameters and scoring weights are tuned per campaign based on result
distribution (e.g. adjusting thresholds if a filter removes too many/few
candidates).
