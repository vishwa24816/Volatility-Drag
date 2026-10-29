# Volatility Drag: Real Data and Monte Carlo

A Jupyter notebook that demonstrates volatility drag, the gap between an asset's arithmetic average return and its realized compound growth rate. It uses real market data from Yahoo Finance (via `yfinance`) and a vectorized Monte Carlo simulation.

## Purpose

When returns fluctuate, compounding them produces a lower growth rate than their arithmetic average. The gap grows with volatility. This notebook shows the effect in four ways: a simple two-step arithmetic example, a generalized round-trip calculation, an empirical comparison using SPY and TQQQ, and a simulation of 10,000 paths under geometric Brownian motion. It closes with a check on leveraged-fund behavior.

The target audience has a quantitative background. The explanations are short and the emphasis is on the math and what the numbers mean.

## Contents

| Section | Topic | What it does |
|---|---|---|
| 1 | Setup and data download | Downloads about 12 years of adjusted closes for SPY and TQQQ, computes simple and log returns, and prints an annualized summary table with the realized drag. |
| 2 | The two-step example | Reproduces $100 → +50% → −50%, then generalizes the round-trip loss to swings of 1% to 90%, with a break-even curve for comparison. |
| 3 | Empirical drag | Plots realized wealth against wealth compounded at the arithmetic mean, and compares the realized drag with the ½σ² approximation. |
| 4 | Monte Carlo simulation | Simulates 10,000 ten-year paths for two synthetic assets with 10% and 50% volatility, both with a 10% arithmetic mean. Reports growth statistics and plots terminal-wealth histograms on a log scale. |
| 5 | Leveraged-fund check | Builds a synthetic 2× daily-rebalanced return series, compares it with 2× the underlying's cumulative return, and summarizes findings and limitations. |

## Requirements

- Python 3.9 or later
- `yfinance`
- `numpy`
- `pandas`
- `matplotlib`
- `scipy`
- An internet connection for Section 1 (Yahoo Finance is queried at runtime)
- Roughly 300 MB of free memory for Section 4, since the simulation arrays are float32 with shape 10,000 × 2,520

Install the packages with:

```bash
pip install yfinance numpy pandas matplotlib scipy
```

Or uncomment the install line at the top of Cell 1.

## How to Run

1. Open the notebook in Jupyter Lab, Jupyter Notebook, VS Code, or Google Colab.
2. Run the cells in order from top to bottom. Each section depends on objects created earlier, especially Section 1, which defines `simple`, `logr`, and `summary`.
3. Use a fresh kernel if you want to confirm reproducibility. The notebook does not depend on prior state beyond the cell order.

The simulation uses `np.random.default_rng(42)`, so Section 4 produces identical results on every run. Section 1 does not: market data changes as Yahoo Finance updates, so the historical numbers will differ depending on when you run the notebook.

## Methodology

**Returns.** Simple daily return is $r_t = P_t / P_{t-1} - 1$. Log return is $x_t = \ln(P_t / P_{t-1})$. Prices are adjusted for dividends and splits (`auto_adjust=True`).

**Annualization.** Arithmetic mean is $\bar{r} \times 252$. Volatility is $s_r \times \sqrt{252}$. Geometric growth is $\bar{x} \times 252$. Realized drag is the arithmetic mean minus the geometric growth rate.

**Approximation.** For lognormal returns with volatility $\sigma$, the drag is approximately $\tfrac{1}{2}\sigma^2$. Section 3 compares this prediction with the realized drag and reports the percentage difference.

**Round trip.** An up move of $s$ followed by a down move of $s$ gives $(1+s)(1-s) = 1 - s^2$, a net loss of $s^2$. The break-even down move after an up move of $s$ is $d^* = s/(1+s)$, which is smaller than $s$. For $s = 50\%$, $d^* = 1/3$.

**Monte Carlo.** Daily log returns are $X \sim \mathcal{N}(\mu_{\log}, \sigma_d^2)$, with $\sigma_d = \sigma / \sqrt{252}$ and $\mu_{\log} = \ln(1 + \mu_{\text{arith}}/252) - \tfrac{1}{2}\sigma_d^2$. This choice makes the expected simple daily return equal $\mu_{\text{arith}}/252$. Terminal log wealth is the sum of daily log returns, and the realized growth rate is that sum divided by 10 years.

**Leverage.** A synthetic 2× daily-rebalanced return is $r^{(2)}_t = 2 r_t$. Its cumulative return is compared with $2 \times$ the underlying's cumulative return. The gap is the compounding penalty.

## Results

The notebook prints tables and displays plots. Exact figures depend on the data downloaded and therefore on the run date. Section 4 results are deterministic because of the fixed seed.

To interpret the output:

- In Section 1, the realized drag for TQQQ should be much larger than for SPY, reflecting its higher volatility.
- In Section 3, the ½σ² approximation should be close to the realized drag for the lower-volatility asset. The gap can be larger for the high-volatility asset, since the approximation is a leading-order term.
- In Section 4, the median growth rate for the 50% volatility asset should be well below its 10% arithmetic mean, and a large fraction of paths should end below $1 despite the positive expected return. The 10% volatility asset should show a much smaller gap.
- In Section 5, the gap between linear and compounded 2× returns should increase with volatility.

## Known Issues

**Section 5 stacks leverage on TQQQ.** TQQQ is already a 3× leveraged product. Applying an additional 2× to its returns produces roughly 6× exposure to the Nasdaq-100, not a 2× check on a plain index. The section's purpose is to test a 2× leveraged position against its underlying, so the cleanest fix is to use SPY alone for that comparison, or to substitute QQQ as the underlying. If you keep TQQQ in the comparison, label the result as "6× the Nasdaq-100" and explain why.

**Summary tables use Styler objects.** The `.style.format` calls render as formatted tables only in Jupyter. Outside a notebook environment, they display as raw objects. Use `.round(4)` or export to CSV if you run the code in a plain Python script.

**Untested code.** The cells were written without being executed in this environment. Run them once and check the printed output before relying on the numbers.

## Limitations

- **Sample dependence.** Section 1 and Section 3 reflect one historical path. Results depend on the window, here roughly 2014 to the present, and on the particular market regime it contains.
- **GBM assumptions.** Section 4 assumes lognormal, independent, identically distributed returns with constant volatility. Real returns have fat tails, volatility clustering, and drawdown dependence that GBM does not capture.
- **Ignored frictions.** Fees, expense ratios, financing costs on leveraged products, bid-ask spreads, taxes, and rebalancing costs are all excluded. Leveraged ETFs also charge management fees, which add to the drag.
- **Survivorship and selection.** The two tickers were chosen to illustrate the effect and are not a representative sample.
- **Not investment advice.** The notebook is an educational demonstration of a mathematical effect.

## Suggested Extensions

- Add a rolling-window drag estimate to see how it varies across regimes.
- Replace the GBM with a Student-t or bootstrapped-return simulation to test sensitivity to fat tails.
- Compare the leveraged-fund gap with actual TQQQ or SSO prices net of expense ratios.
- Vary the target arithmetic return in Section 4 and plot the median growth rate as a function of volatility.
