# Roadmap

`strategy-evaluation` is an umbrella for evaluating trading-strategy backtests honestly. Today it ships one capability — **`convergence`** — and grows by adding sibling evaluator skills, each a self-contained folder under `plugins/strategy-evaluation/skills/`.

Tracking: Tier 1 is filed as issues under the [`v0.2` milestone](https://github.com/martianmobile/strategy-evaluation/milestone/1). Tiers 2–3 are directional until promoted.

## Tier 1 — sharpen the core (`v0.2`, next)

Make the existing convergence verdict defensible on real exports.

- **Full metric schema + cardinality-based inference** — [#1](https://github.com/martianmobile/strategy-evaluation/issues/1). Classify param-vs-metric by cardinality (a swept param takes few distinct grid values; a metric is near-unique per variant) instead of a keyword list, so real exports with 12+ metric columns classify correctly. Ships a realistic example.
- **Multiple-testing correction** — [#2](https://github.com/martianmobile/strategy-evaluation/issues/2). Deflated Sharpe (Bailey–López de Prado) + Probability of Backtest Overfitting folded into the verdict — the rigorous answer to "is my winner real, or did I overfit by trying N variants?"
- **Tests + CI** — [#3](https://github.com/martianmobile/strategy-evaluation/issues/3). pytest fixtures for all three verdicts + edge cases, GitHub Actions on a Python matrix proving the analyzer stays stdlib-only.

## Tier 2 — new evaluators (later)

The umbrella payoff — each a sibling skill alongside `convergence`.

- **`robustness`** — outlier-trade drop test (does edge survive removing top-k PnL trades?), bootstrap CIs on Sharpe, parameter-perturbation sensitivity.
- **`walk-forward`** — multi-fold rolling/anchored stability of a chosen variant, beyond a single IS/OOS split.
- **`regime`** — performance broken out by vol/trend regime; flags strategies that only work in one regime.

## Tier 3 — workflow & polish (later)

- **Folder / multi-file input** — point at a directory of per-variant exports and auto-assemble the variant table.
- **`--json` output + `config.yaml`** — machine-readable verdict for CI gating (fail a research build if not CONVERGED); project-pinned defaults.
- **Visual report** — HTML/SVG heatmap of the parameter grid colored by metric, with the plateau/edge highlighted.

## Stretch

- **Joint multi-metric convergence** — require a secondary metric (e.g. max-drawdown, hit-rate) to also converge before declaring CONVERGED. Enabled by the Tier-1 schema work.

---

Contributions welcome — issues tagged [`good first issue`](https://github.com/martianmobile/strategy-evaluation/labels/good%20first%20issue) are a good entry point. Built by [Martian Mobile](https://martianmobile.com).
