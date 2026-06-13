# strategy-evaluation

A Claude Code plugin for quant researchers: **evaluate trading-strategy backtests** and get an honest verdict on what to do next.

The convergence math is computed deterministically by a bundled, dependency-free Python analyzer. Claude interprets the verdict and — on an ITERATE — proposes concrete next variants.

This is the public, MIT-licensed, sanitized version of patterns used in live crypto trading research at [Martian Mobile](https://martianmobile.com).

---

## Capabilities

| Skill | Status | What it does |
|-------|--------|--------------|
| **`convergence`** | shipped (v0.1) | Read a variant parameter sweep → measure dispersion, IS/OOS rank stability, and parameter-plateau structure → verdict: **CONVERGED / ITERATE / KILL**. |
| `robustness` | planned | Single-strategy OOS degradation / outlier-sensitivity scoring. |
| `walk-forward` | planned | Rolling-window stability of a chosen variant. |

Each capability is a self-contained skill folder under `plugins/strategy-evaluation/skills/`, so new evaluators drop in without touching the existing ones.

---

## Install

This repo is also a Claude Code plugin marketplace. From inside Claude Code:

```
/plugin marketplace add martianmobile/strategy-evaluation
/plugin install strategy-evaluation@martianmobile
```

Then invoke the convergence evaluator:

```
/convergence results.csv --metric sharpe_oos
```

…or just ask in natural language: *"are these variants converged?"*, *"should I keep iterating?"*

---

## What `convergence` checks

Given one row per variant (parameters + metrics), it measures:

- **Top-K dispersion** — are the best variants clustered, or is one a lucky outlier?
- **IS↔OOS rank stability** — does the in-sample ordering survive out-of-sample? (Spearman)
- **Parameter-space structure** — do winners sit on a plateau, or are they isolated points (overfit)? Is the best variant pinned to the edge of the swept range?
- **Sample sufficiency** — are any top variants below a trade-count floor?

…then returns a verdict:

| Verdict | Meaning |
|---------|---------|
| **CONVERGED** | Tight, OOS-stable, plateau region. Ship the representative variant. |
| **ITERATE** | Some criteria unmet (edge peak, weak stability, high dispersion). Suggested next sweep included. |
| **KILL** | No stable edge — IS doesn't survive OOS, or best ≈ median. Abandon or rethink. |

---

## Usage

Direct (no Claude required — it's a plain CLI):

```bash
python3 plugins/strategy-evaluation/skills/convergence/scripts/analyze.py \
  plugins/strategy-evaluation/skills/convergence/examples/results_converged.csv \
  --metric sharpe_oos
```

Exit code encodes the verdict: `0` CONVERGED · `1` ITERATE · `2` KILL.

### Input format

A CSV with one row per variant:

```
variant_id, lookback, threshold, sharpe_is, sharpe_oos, n_trades
v001, 20, 0.5, 1.82, 1.61, 412
v002, 25, 0.5, 1.85, 1.58, 401
...
```

Column roles are inferred from names/types. IS/OOS pairs are detected by suffix (`_is`/`_oos`, `_in`/`_out`, `_train`/`_test`). For Parquet/SQLite, export to CSV first.

### Flags

| Flag | Default | Purpose |
|------|---------|---------|
| `--metric` | auto | Ranking metric column |
| `--top-k` | 5 | Top variants to assess |
| `--dispersion-threshold` | 0.05 | Max relative dispersion for CONVERGED |
| `--min-samples` | 200 | Sample floor per variant |
| `--rank-threshold` | 0.6 | Min IS/OOS Spearman for CONVERGED |
| `--lower-is-better` / `--higher-is-better` | auto | Metric direction override |
| `--id-column` / `--sample-column` | auto | Column-role overrides |
| `--save` | off | Also write `iteration_check_<timestamp>.md` |

---

## Example output

```
# Iteration Check | results_converged.csv | 2026-06-06

## Verdict: CONVERGED

**Top-5 cluster tightly (1.7% dispersion) on a parameter plateau, OOS-stable (ρ=0.99) — ship `v10`.**

## Convergence Analysis
| Metric             | Top-K Mean | Top-K Std | Dispersion | Threshold |
|--------------------|-----------|-----------|------------|-----------|
| sharpe_oos (rank)  | 1.58      | 0.03      | 1.7%       | < 5% ✓    |
| sharpe_is          | 1.82      | 0.02      | 1.3%       | < 5% ✓    |

IS→OOS rank stability (Spearman): 0.99 (threshold > 0.60 ✓)

## Parameter-Space Check
- lookback: top-K at [20, 25, 30] — contiguous
- threshold: top-K at [0.5, 0.6] — contiguous
[x] Plateau region — robust, safe to deploy
```

---

## Limitations

- **Not a backtester.** It analyzes results you already have; it does not run strategies.
- **Not a live monitor.** Research-phase tooling — live trading needs different rails.
- **Metric-agnostic, not metric-smart.** It checks dispersion of whatever metric you point at; it doesn't know whether that metric is the right one.
- **Convergence ≠ profit.** Convergent variants can converge on losing strategies. The verdict reports convergence, not edge.

---

## Why this exists

In real quant research, the failure mode isn't running too few backtests — it's calling one lucky variant a "winner" when its neighbors perform completely differently. Convergence-across-variants is the only honest signal that the parameter region has structure. This plugin enforces the discipline — and gives the same treatment to the other ways a strategy can look better than it is.

---

Built by [Martian Mobile](https://martianmobile.com). MIT licensed — see [LICENSE](LICENSE).
