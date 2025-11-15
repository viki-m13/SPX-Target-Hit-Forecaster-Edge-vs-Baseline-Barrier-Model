## SPX Multi-Horizon Target-Hit Forecaster — Lift vs Baseline Barrier Model

This model is a barrier-hitting forecaster for SPX (^GSPC) that searches over horizon, minimum move, and regime filters to maximize edge vs a naive baseline.

On each day it asks:

> “Will SPX’s high reach at least X% above today’s close within the next N trading days?”

It then:

* Builds labels from the future max high over [t+1, …, t+N].
* Only fires in favorable regimes based on:

  * Convex momentum (quadratic trend + curvature in log price),
  * ATR-based target sizing,
  * 30-day realized volatility,
  * RSI band (40–70),
  * Position in a 60-day rolling high/low range.
* Uses walk-forward cross-validation (WFCV) with no data leakage to tune:

  * horizon_days
  * min_move
  * lambda_convex
  * ATR_MULT
  * VOL_MIN
    with the objective of maximizing lift vs baseline subject to a minimum coverage constraint.

---

### Edge vs Baseline: How It’s Measured

We compare a naive baseline to the gated model.

* **Baseline hit rate**
  Probability that the move happens, over all valid days (no filtering):

  `Baseline = P(hit +min_move within N days | all days)`

* **Model hit rate**
  Probability that the move happens, *conditional on the model actually firing*:

  `Model = P(hit target (>= min_move) | model fires)`

* **Lift vs baseline (edge)**
  Absolute improvement in hit rate:

  `Lift (edge) = Model - Baseline`

This is the key metric: it shows how much the regime filter concentrates probability mass into a small subset of high-conviction days, instead of predicting every day.

---

### Configuration A — +3% Target Within 10 Trading Days

* **Target:** SPX high reaches at least **+3.0%** within **10 trading days**.
* **Baseline hit rate:**
  `Baseline = 25.3257%`
  (Random day: ~1 in 4 times SPX hits +3% within 10 days.)
* **Model hit rate (conditional on firing):**
  `Model = 46.2451%`
* **Coverage:**
  `Coverage = 6.3377%` of days have a live signal.
* **Lift vs baseline (absolute):**
  `Lift = 20.9194 percentage points`
  (46.25% − 25.33%)

**Interpretation:**
The model roughly doubles the probability of hitting a +3% target (from ~25% to ~46%) on a small, selected subset of days (~6% of the sample) where the regime filter is on.

---

### Configuration B — +2% Target Within 5 Trading Days

* **Target:** SPX high reaches at least **+2.0%** within **5 trading days**.
* **Baseline hit rate:**
  `Baseline = 25.5762%`
* **Model hit rate (conditional on firing):**
  `Model = 49.0119%`
* **Coverage:**
  `Coverage = 6.3377%` of days have a live signal.
* **Lift vs baseline (absolute):**
  `Lift = 23.4357 percentage points`
  (49.01% − 25.58%)

**Interpretation:**
For a shorter horizon and smaller move (+2% in 5 days), the model again fires on about 6% of days, but these days have roughly double the baseline hit probability (from ~26% to ~49%), producing an edge of about 23 percentage points in target-hit probability.
