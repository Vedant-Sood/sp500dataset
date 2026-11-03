# Using the S&P 500 membership dataset

[Repository overview and upstream notes](../README.md)

This is a fork of [fja05680/sp500](https://github.com/fja05680/sp500).
The existing [MIT license](../LICENSE) and upstream attribution are retained.
Index membership is distinct from price history and total returns.

## Environment

Use a separate Python environment. A starting dependency set for the committed
notebooks is:

~~~bash
python -m venv .venv
~~~

Activate that environment for your platform, then:

~~~bash
python -m pip install jupyterlab pandas numpy wikipedia lxml matplotlib tqdm
jupyter lab
~~~

Dependencies are inferred from notebook imports; there is no pinned environment
or current cross-version test matrix. Run notebooks from the repository root.
Legacy display-only JavaScript cells can be skipped if your notebook frontend
does not support them.

## Files and workflow

| Step | Notebook | Inputs and output |
| --- | --- | --- |
| 1 | [sp500.ipynb](../sp500.ipynb) | Reads Wikipedia's current table and writes sp500.csv |
| 2 | [sp500_historical.ipynb](../sp500_historical.ipynb) | Combines the original history and changes file; writes the updated historical CSV |
| 3 | [sp500_by_date.ipynb](../sp500_by_date.ipynb) | Selects membership at a requested historical date |
| 4 | [PIT to Ticker Delta.ipynb](../PIT%20to%20Ticker%20Delta.ipynb) | Writes ticker entry/exit intervals to sp500_ticker_start_end.csv |

The changes CSV is curated input, not an automatically complete corporate-action
feed. Before a refresh, replace the notebook's inherited User-Agent contact
with your own appropriate identity. Review changes and compare the rebuilt
latest membership against the retrieved current table before committing CSVs.

## Read a historical snapshot

With pandas installed, from the repository root:

~~~python
import pandas as pd

history = pd.read_csv(
    "S&P 500 Historical Components & Changes (Updated).csv",
    parse_dates=["date"],
).sort_values("date")

requested = pd.Timestamp("2023-10-23")
if not history["date"].min() <= requested <= history["date"].max():
    raise ValueError("Requested date is outside the recorded snapshot range")
eligible = history.loc[history["date"] <= requested]
row = eligible.iloc[-1]
tickers = sorted(set(row["tickers"].split(",")))
print("Membership recorded at:", row["date"])
print(tickers)
~~~

The lookup takes the most recent recorded membership on or before the date.
Inspect the final date in the file before interpreting it as current.
Rows represent membership states and changes, not necessarily every trading day.

## Schema and interpretation

| File | Main fields |
| --- | --- |
| sp500.csv | Wikipedia table; Symbol is used by the notebooks |
| Historical membership CSVs | date and a comma-separated tickers field |
| sp500_changes_since_2019.csv | date, add and remove lists; inspect the actual header before extending |
| sp500_ticker_start_end.csv | ticker, start_date and end_date; an open interval may have no end_date |

A ticker can enter the index more than once. Symbol changes, missing historical
members and the initial source coverage affect research use. Membership files
alone do not supply delisted price history or prove an unbiased backtest.
See the original README for the upstream author's historical data-quality notes.

Notebook refreshes overwrite their named output CSVs. Keep a clean Git checkout
or retain the previous snapshot before regenerating data. No notebook refresh
or market-data accuracy audit was performed for this documentation change.
