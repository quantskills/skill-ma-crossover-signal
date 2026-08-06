# skill-ma-crossover-signal

Single-symbol moving-average crossover skill: given a ticker and fast/slow MA periods, it returns the fast/slow MAs (SMA/EMA), the current trend state, the latest golden/death cross (date + bars-ago), and the price bias (乖离率), auto-routing A-share / HK / US by suffix.

> This first paragraph doubles as the platform summary (suggested Topics: `quant` `moving-average` `golden-cross` `trend` `technical-analysis` `a-share` `hk` `us`).

> 中文说明见 [`README.md`](./README.md)（中文优先）。

## What it does

Pulls daily closes from `panda_data` (A-share) / `tqx_data` (HK / US), computes both
MAs on the **same cleaned close series**, and detects a cross by the **sign change of
(fast − slow)** — so it reports the *actual last cross with its date*, not merely
"fast is currently above slow" (which hides that the cross may be 40 bars stale).
Returns state, MA gap, price bias (乖离率), the last cross, and recent signals as JSON.

Full spec — parameters, output schema, definitions, when-NOT-to-use — is in [`SKILL.md`](./SKILL.md).

## Layout

```
skill-ma-crossover-signal/
├── SKILL.md                    # required: skill spec
├── README.md                   # Chinese-first readme (first paragraph = summary)
├── README.en.md                # this file (English)
├── LICENSE                     # GPLv3
├── quantskills.yaml            # upstream manifest (license: GPL-3.0-only)
├── agents/                     # runtime adapters (cursor / hermes / openclaw)
│   ├── cursor-rule.mdc
│   ├── openai.yaml
│   └── portable-loader.md
├── scripts/
│   └── ma_crossover_signal.py  # executable: async def run(...) -> str + CLI
└── references/
    └── example_output.md       # example output
```

## Run it

**As a platform skill** — entry point `async def run(...) -> str` in
`scripts/ma_crossover_signal.py` (Panda QuantFlow skill contract).

**Standalone CLI:**

```bash
python scripts/ma_crossover_signal.py 600519.SH --fast-period 5 --slow-period 20
python scripts/ma_crossover_signal.py AAPL.NB --ma-type ema --fast-period 10 --slow-period 50
```

Dependencies: `pandas`, `numpy` + `panda_data` / `tqx_data`. Missing data libs → a
structured `Error: …` string, never a raise.

## Status

Listed → Runnable → Verified. Local checks passing: `py_compile`, frontmatter lint,
`run()` signature ↔ parameters match, and a synthetic-data smoke test (crossover
detection, EMA path, `fast >= slow` rejection, short-history guard).

## Disclaimer

- **Research & educational use only.** Output is informational research, **not investment advice**, and makes **no promise of returns** or profitability.
- **Data sources:** A-share via `panda_data`; HK / US via `tqx_data` (platform-provided). Output quality depends entirely on upstream data availability and correctness.
- **Assumptions & limitations:** MAs are computed on daily closes; a cross is the sign change of `(fast − slow)`. This is a **signal descriptor, not a backtest** — no transaction costs, slippage, or position sizing are modeled, and a crossover is not a buy/sell recommendation. `last_cross` is `null` when history `< slow_period`.
- **Risk boundary:** do not use as the sole basis for real-money entries/exits; validate independently and understand market risk.

## License

GPL-3.0-only. See [`LICENSE`](./LICENSE).
