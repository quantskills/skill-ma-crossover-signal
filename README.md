# skill-ma-crossover-signal

单标的均线交叉信号 skill：给一个股票代码和快/慢均线周期，返回快慢均线（SMA/EMA）、当前趋势状态、最近一次金叉/死叉（日期 + 距今 bar 数）、均线乖离率，自动按后缀路由 A股 / 港股 / 美股。Fast/slow moving-average crossover (golden/death cross with date + bars-ago) for one symbol across A-share / HK / US.

> 首段即平台简介（Topics 建议：`quant` `moving-average` `golden-cross` `trend` `technical-analysis` `a-share` `hk` `us`）。

## What it does

Pulls daily closes from `panda_data` (A 股) / `tqx_data` (港股 / 美股), computes both
MAs on the **same cleaned close series**, and detects a cross by the **sign change of
(fast − slow)** — so it reports the *actual last cross with its date*, not merely
"fast is currently above slow" (which hides that the cross may be 40 bars stale).
Returns state, MA gap, price bias (乖离率), the last cross, and recent signals as JSON.

Full spec — parameters, output schema, definitions, when-NOT-to-use — is in [`SKILL.md`](./SKILL.md).

## Layout

```
skill-ma-crossover-signal/
├── SKILL.md                    # 必须：技能说明
├── README.md                   # 本文件（首段=简介）
├── scripts/
│   └── ma_crossover_signal.py  # 可执行代码：async def run(...) -> str + CLI
└── references/
    └── example_output.md       # 示例输出
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
