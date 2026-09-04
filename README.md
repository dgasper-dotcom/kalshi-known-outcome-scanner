# Kalshi Known-Outcome Scanner

This scanner looks for Kalshi markets where an external verifier indicates the outcome is already known, then records APY-adjusted paper entries using real executable orderbook liquidity.

It is separate from the MLB fair-value model. It does not model future baseball outcomes; it only attempts to trade markets where the result has already happened or can already be verified.

## What It Scans

- MLB player props: already-hit YES, final known YES, final known NO
- MLB, NFL, NCAAF, NBA, NHL, WNBA game winner markets
- Final-score spread, total, and team-total markets where the threshold can be parsed
- Hourly and daily high-temperature markets using Weather.com verifier endpoints
- Index close/range markets using Yahoo Finance chart closes

## Economics

The default paper settings are:

- `contracts=100`
- `min_contracts=1`
- `max_ask=0.99`
- `known_outcome_apy=0.0325`
- `known_outcome_min_net_profit_per_contract=0.001`

Capital used is execution cost plus fees. APY-adjusted PnL subtracts a 3.25% annualized cash hurdle for the expected settlement wait.

## Setup

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Optional Kalshi authentication:

```bash
export KALSHI_ACCESS_KEY="your-key-id"
export KALSHI_PRIVATE_KEY_FILE="/path/to/private_key.pem"
```

The scanner can read public orderbooks without auth when Kalshi allows it, but authenticated requests are more reliable.

## Run One Scan

```bash
python3 run_known_outcome.py scan \
  --contracts 100 \
  --min-contracts 1 \
  --max-ask 0.99 \
  --known-outcome-apy 0.0325
```

## Run Continuously

```bash
./known_outcome_loop.sh
```

Environment overrides:

```bash
CONTRACTS=100 POLL_SECONDS=90 ./known_outcome_loop.sh
```

## Outputs

Runtime files are intentionally ignored by git.

- `known_outcome_candidates.csv`: verified known outcomes and liquidity status
- `known_outcome_opportunities.csv`: currently fillable entries passing the APY filter
- `known_outcome_trades.csv`: cumulative deduped paper ledger
- `known_outcome_pnl.csv`: APY-adjusted PnL summary by market family and side

## Notes

The ledger is deduped by `kalshi_market_ticker|known_side`. If a market was previously entered at a smaller cap, raising `--contracts` does not automatically top up that existing paper entry.
