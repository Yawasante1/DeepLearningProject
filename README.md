# Is the Ghana Stock Exchange Weak-Form Efficient?

### Classical econometric tests and LSTM-based deep learning, applied to 23 years of monthly GSE Composite Index data

If a market is weak-form efficient, past prices should disclose nothing useful about future prices. This project tests that idea on the **Ghana Stock Exchange Composite Index (GSE-CI)** from two angles at once:

1. **Statistical tests**: do returns behave like a random walk?
2. **Forecasting models**: can ARIMA, Random Forest, XGBoost, neural networks and LSTMs actually predict returns from past data?

**Short answer:** the evidence is broadly consistent with weak-form efficiency. The best model, a hybrid ARIMA-LSTM, beat the others, but only by a small margin, and most models did worse than simply predicting the average return.

## Why I built this

I come from both financial economics and data science, and this question sits where they meet. Most studies of the GSE use either classical tests or forecasting models, but not both. Combining them gives two independent kinds of evidence, which is more convincing when they agree. It also guards against over-reading either one: a model can be statistically "better" without being economically useful.

## Key findings

- **Returns are stationary.** ADF (-15.29) and Phillips-Perron (-15.65) reject a unit root at the 1% level, and KPSS does not reject stationarity.
- **Little linear dependence.** The Ljung-Box test finds no significant autocorrelation at lags 2, 5, 10 or 20 (p-values 0.31 to 0.75).
- **Mostly a random walk.** The variance ratio test does not reject the random walk at any horizon from 2 to 20 months (p-values around 0.25 to 0.27).
- **One test disagrees.** The runs test rejects randomness (statistic -7.42, p < 0.001), so the evidence is not unanimous.
- **Volatility clusters.** A GARCH(1,1) model gives persistence (alpha + beta) of 0.9647: calm and turbulent periods tend to last. This is about changing risk, not predictable returns.
- **Forecasting gains are small.** The hybrid ARIMA-LSTM had the lowest error (RMSE 0.046) and was the only model with a positive R-squared (0.063). Every other model had a negative R-squared.

## Data

- **Source:** monthly Ghana Stock Exchange Composite Index (GSE-CI) prices from the Ghana Stock Exchange
- **Period:** December 2002 to April 2026 (280 monthly returns)
- **Returns:** log returns, `r_t = ln(P_t / P_(t-1))`

| Statistic | Value |
|---|---|
| Observations | 280 |
| Mean | 0.0085 |
| Std. deviation | 0.1322 |
| Skewness | -11.52 |
| Kurtosis | 170.92 |
| Min / Max | -1.9417 / 0.3569 |

The return distribution is heavily skewed with very fat tails, which is common in financial data but extreme here (see Limitations).

## Method

**Step 1: Econometric tests**

| Question | Test |
|---|---|
| Are returns stationary? | ADF, Phillips-Perron, KPSS |
| Are returns random? | Runs test, variance ratio (k = 2, 5, 10, 15, 20) |
| Is there serial correlation? | Ljung-Box (lags 2, 5, 10, 20) |
| Does volatility cluster? | GARCH(1,1) with constant mean, normal errors |

**Step 2: Forecasting models**

- **Linear baseline:** ARIMA
- **Machine learning:** Random Forest, XGBoost, ANN
- **Deep learning:** LSTM, BiLSTM, LSTM with attention
- **Hybrid:** ARIMA-LSTM, where ARIMA captures the linear part and the LSTM models what is left over

Models were tuned on a validation set. Performance is measured by RMSE, MAE, R-squared and directional accuracy (how often the model gets the up/down direction right).

**Step 3: Joint reading.** If the market is efficient, the tests should find little structure *and* the models should struggle to forecast. Both pointing the same way is stronger evidence than either alone.

## Forecasting results

| Model | RMSE | MAE | R-squared | Directional accuracy |
|---|---|---|---|---|
| ARIMA | 0.072 | 0.043 | n/a | n/a |
| Random Forest | 0.092 | 0.057 | -0.834 | n/a |
| XGBoost | 0.099 | 0.049 | -1.150 | n/a |
| ANN | 0.078 | 0.048 | -0.332 | 0.491 |
| LSTM | 0.069 | 0.044 | -0.027 | 0.722 |
| BiLSTM | 0.069 | 0.044 | -0.043 | 0.704 |
| LSTM + Attention | 0.071 | 0.044 | -0.073 | 0.704 |
| **Hybrid ARIMA-LSTM** | **0.046** | **0.031** | **0.063** | 0.657 |

**How to read this**

- A negative R-squared means a model does worse than just predicting the average return every month. Random Forest, XGBoost, ANN and all three LSTM variants fall in this group. Only the hybrid is above zero, and only barely (0.063).
- The hybrid's lower error shows that combining linear and nonlinear modelling helps a little. It does not show that the market is easy to beat.
- Statistical predictability is not the same as a profitable trading strategy. This project does not simulate trading or include transaction costs.

## Limitations

- **A very small test window.** The forecast plot covers roughly three years of monthly data, so all forecasting metrics, directional accuracy especially, rest on few observations and can move a lot with a different split.
- **Directional accuracy has no baseline.** The LSTM's 72.2% is the highest number in the table, but I did not compare it against a naive rule such as "always predict up", or test it formally against 50%.
- **Extreme observations.** The minimum monthly log return of -1.94 is far outside the rest of the series and drives the skewness (-11.5) and kurtosis (171). Results of the tests and the GARCH fit may be sensitive to it, and I have not yet run robustness checks without it.
- **Mixed test evidence.** The runs test rejects randomness while the variance ratio and Ljung-Box tests do not. "Consistent with weak-form efficiency" is the fair reading, not proof of efficiency.
- **Normal errors in GARCH.** The returns are heavy-tailed, so a Student-t or skewed error distribution would likely fit better.
- **One market, monthly data only.** Daily data might show shorter-term patterns that monthly returns hide.
- **Weak-form only.** The project says nothing about semi-strong or strong-form efficiency.

## Ideas for next steps

- Robustness checks excluding or winsorising extreme months
- Walk-forward (rolling) evaluation instead of a single split, with significance tests for forecast comparisons such as Diebold-Mariano
- Directional accuracy tested against a naive baseline
- GARCH variants with heavier-tailed errors (and EGARCH/GJR for asymmetry)
- Rolling-window efficiency tests to see whether efficiency changes over time
- A simple trading backtest with transaction costs to test economic exploitability
- Daily data, and a comparison with other African frontier markets


## References

- Fama, E. F. (1970). Efficient capital markets: A review of theory and empirical work. *The Journal of Finance*, 25(2), 383-417.
- Dickey, D. A. and Fuller, W. A. (1979). Distribution of the estimators for autoregressive time series with a unit root. *Journal of the American Statistical Association*, 74(366), 427-431.
- Phillips, P. C. B. and Perron, P. (1988). Testing for a unit root in time series regression. *Biometrika*, 75(2), 335-346.
- Kwiatkowski, D., Phillips, P. C. B., Schmidt, P. and Shin, Y. (1992). Testing the null hypothesis of stationarity against the alternative of a unit root. *Journal of Econometrics*, 54(1-3), 159-178.
- Lo, A. W. and MacKinlay, A. C. (1989). The size and power of the variance ratio test in finite samples. *Journal of Econometrics*, 40(2), 203-238.
- Lo, A. W. (2004). *The Adaptive Markets Hypothesis*. Princeton University Press.

## Author

**Opoku Asante Emmanuel**, data scientist and financial economist
Email: vkpoasante@gmail.com

