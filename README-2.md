# A Dynamic AI-Assisted Algorithmic Framework for Portfolio Construction and Tail-Risk Hedging

**An Application to the Olive Oil Sector**

Source code accompanying the MSc dissertation (Master in Finance, ISEG – Universidade de Lisboa) by **Luis Sa Morais** (2026).

This repository contains the full, reproducible pipeline that builds hedging portfolios for **any** agent carrying a fixed, hard-to-diversify exposure to a single sector. The algorithm is the contribution; the olive-oil sector is used as the empirical case study. It is the implementation referenced in **Appendix A** of the dissertation.

---

## Overview

The framework runs in a single notebook and proceeds in five stages:

1. **Multi-criteria screening** of 36 candidate ETFs / commodity proxies on four criteria — Sharpe ratio, correlation to the olive-oil proxy, conditional (GARCH) volatility, and excess kurtosis — with robust (winsorized) min-max normalization.
2. **Objective criterion weighting** with the **CRITIC** method (no arbitrary ex-ante weights).
3. **AI-assisted regime tilt**: a deterministic rule-based stress index is computed from macro-financial signals (VIX, six-month S&P 500 momentum, credit-spread and inflation trends); a language model returns a *structured regime view* (macro regime, macro-stress, olive-oil sector tail-risk, confidence); the two are fused **in proportion to the model's confidence**, and a confidence-gated sector tail-risk overlay lifts the tail-protective criteria. Multipliers are bounded to `[0.60, 1.60]` with an exact unit geometric mean. The mechanism falls back to the deterministic rule if the API is unavailable, so the pipeline is reproducible.
4. **Portfolio optimization** under a fixed **50%** olive-oil allocation:
   - Markowitz mean–variance (Minimum-Variance and Maximum-Sharpe frontiers);
   - **Merton Jump-Diffusion (MJD)** simulation of monthly returns;
   - **Differential Evolution** minimizing **95% CVaR** on the simulated distribution.
5. **AI-assisted recommendation** layer that maps the three portfolios to investor profiles.

## Repository structure

```
.
├── quant_portfolio_framework.ipynb    # the full pipeline (run top to bottom)
├── dadostrabalho_ate_marco2026.xlsx   # input data (place here — see "Data")
├── requirements.txt
├── .gitignore                         # Python template (ignores .env, caches)
├── LICENSE                            # MIT
└── README.md
```

## Requirements

Python 3.11+. Install dependencies into a clean virtual environment:

```bash
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

> **Reproducibility note.** For an exact reproduction of the figures in the dissertation, pin the packages to the versions used on the original machine (`pip freeze > requirements.txt`). The `arch` (GARCH) version in particular can shift the volatility screening at the margin and, very occasionally, the marginal asset selected.

## Data

The notebook reads a single Excel workbook, **`dadostrabalho_ate_marco2026.xlsx`**, expected in the repository root. It contains monthly series (January 2010 – March 2026, 195 observations): the 36 candidate assets, the constructed olive-oil proxy `RevOliveOil`, and its components (`POLVOILUSDM`, `prod_EU27`).

If your data files cannot be redistributed, keep them out of version control (see `.gitignore`) and document their source instead.

## API key (required for the AI tilt and recommendation)

The notebook never stores a key in the `.ipynb`. Provide it via an environment variable:

```bash
export OPENAI_API_KEY="sk-..."   # Windows (PowerShell): $env:OPENAI_API_KEY="sk-..."
```

If the variable is unset, the notebook prompts for the key at runtime (`getpass`). If no key is available or the call fails, the AI tilt **falls back to the deterministic rule**, so every result remains reproducible without an API call.

If you prefer to keep the key in the project, create a `.env` file containing `OPENAI_API_KEY=sk-...` — it is git-ignored and will not be committed.

## How to run

1. Place `dadostrabalho_ate_marco2026.xlsx` in the repository root.
2. Set `OPENAI_API_KEY` (or let the notebook prompt you).
3. Open the notebook and **Run All**.

## Key parameters (for reproducibility)

| Parameter | Value |
|---|---|
| Sample | 2010-01 → 2026-03 (195 monthly obs) |
| Risk-free rate | 3-month U.S. Treasury (FRED `DGS3MO`), 3.67% p.a. at 2026-02-27 |
| Olive-oil allocation | fixed 50% |
| MJD scenarios | `N_SIM = 50,000` |
| Simulation seed | `SEED = 777` (common random numbers) |
| Differential Evolution | `SEED_DE = 42`, `popsize = 15`, `maxiter = 150` |
| CVaR level | `ALPHA = 0.05` (95%) |
| Tilt band | `[0.60, 1.60]`, geometric mean = 1 |

## Citation

> Sa Morais, L. (2026). *A Dynamic AI-Assisted Algorithmic Framework for Portfolio Construction and Tail-Risk Hedging: An Application to the Olive Oil Sector.* MSc dissertation, Master in Finance, ISEG – Universidade de Lisboa.

## License

Released under the MIT License — see [`LICENSE`](LICENSE).
